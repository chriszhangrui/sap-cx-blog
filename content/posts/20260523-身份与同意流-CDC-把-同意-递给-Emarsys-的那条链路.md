---
title: "身份与同意流：CDC 把'同意'递给 Emarsys 的那条链路"
date: 2026-05-23
module: Engagement Cloud
source: https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/integrating-sap-customer-data-cloud-with-sap-emarsys-for-personalized/ba-p/14308465
tags: [SAP, Engagement Cloud, Emarsys, CDC, 同意流, External Events API, WSSE, Integration Suite]
---

# 身份与同意流：CDC 把"同意"递给 Emarsys 的那条链路

营销侧聊得最热闹的还是改名——SAP Emarsys 改叫 SAP Engagement Cloud。但如果你真打算把它放进出海业务的技术栈，更值得花时间看的是另一件事：**当 SAP Customer Data Cloud 拿到消费者的同意标记之后，这条同意信号是怎么准时、不丢、按字段映射地走到 Engagement Cloud 的活动触发器上的**。

这条链路看起来只有"采集 → 同步 → 营销使用"三步，工程上要拆成至少六层：身份解析、字段映射、同步节奏、连接器认证、活动触发时校验、回流监控。下面按工程实现路径展开。

![CDC → Engagement Cloud 双管道架构](http://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLVflRE5huRrS5QDYaiaMdpsPrjDNR6nuZib99aegOEsgKE3SaVrib1G3ZO0YJt1Wx810wLvsNLFRZc4vG1R4U0pugN0nM7Okvm6ib8/0?from=appmsg)

## 一、起点：CDC 里那一条 consent 记录到底长什么样

CDC 的 Profile 是一份 JSON 文档，identity 段里既保存账号 ID、邮箱、手机号哈希，也保存一个完整的 `preferences` 子树。每个偏好下面挂三件事：

- **状态**：`isConsentGranted` 布尔值
- **时间戳**：`lastConsentModified`，ISO 8601
- **法律基础**：`legalLanguageVersion`、`tag`，对应当时勾选的隐私政策版本

这是合规审计的真实落点。营销系统拿过去用，不是只取那个 true/false，而是要连同时间戳和法律基础一起带走——否则一旦数据主体提出删除或撤回，你回不到当时具体勾的哪一版条款。

外发到 Engagement Cloud 这一侧的对应字段，就要按 Emarsys 的预定义系统字段表来对：

```
Field 1     -> Email
Field 3     -> Email Opt-In        (1=opted-in, 2=opted-out)
Field 9     -> Mobile (E.164)
Field 47    -> Mobile SMS Opt-In   (1/2)
Field 31    -> External ID
```

CDC 的 `email.isConsentGranted=true` 不是直接塞 Field 3=1 就完事——还要判断这条 consent 对应的 purpose 是否和 Field 3 这条 channel/purpose 对齐。CDC 一个用户可以有多条 preferences（Newsletter、Transactional、Promotion），Emarsys 这边只有一个 Email Opt-In 字段。一对多压扁的策略要在连接器里写清楚：是按"任一 purpose 同意即视为 opt-in"，还是"全部 purpose 都同意才视为 opt-in"。这不是技术问题，是和合规口径一起拍板的事。

## 二、连接器：放在 SAP Integration Suite 还是直接调

CDC 提供的标准外发方式有两种：

第一种是 **Webhook**，事件触发即出账。一次 profile 写入，CDC 推一个 JSON payload 到目标 endpoint。优点是延迟在秒级；代价是吞吐有限，且失败重试逻辑要自己兜——CDC 默认重试三次，如果你的下游网关临时 503，第四次就丢了。

第二种是 **批量导出 + 计划任务**，每隔 N 分钟从 CDC 拉一个增量快照，落地后再推 Emarsys。优点是可对账、好回放；代价是 lag 通常 15 分钟起。

实战的拓扑大多落在中间——**SAP Integration Suite 作为连接器**。它解决三个问题：

- **认证转换**：CDC 用 OAuth2 / API Key，Emarsys Core API 用 WSSE（Username + PasswordDigest + Nonce + Timestamp 的 SHA-1 摘要）。两套机制差异大，放在 IS 里做转换比让两边各自妥协合理。
- **重试与死信**：webhook 失败之后落 dead-letter queue，单独建议告警；这是 CDC 原生不提供的。
- **字段映射的可视化**：连接器做映射，比写在任何一端的 user-exit 里都更容易交接。

WSSE 头部的生成逻辑（这个细节经常被低估）：

```javascript
// 摘要 = base64( hex( SHA1( nonce + timestamp + secret ) ) )
const ts = new Date().toISOString();
const nonce = crypto.randomBytes(16).toString('hex');
const digest = Buffer.from(
  crypto.createHash('sha1')
        .update(nonce + ts + secret)
        .digest('hex')
).toString('base64');

const wsse = `UsernameToken Username="${user}", PasswordDigest="${digest}", Created="${ts}", nonce="${nonce}"`;
```

注意 SHA-1 摘要要先转成 hex 字符串再做 base64——直接对二进制 base64 是错的，签名会被 Emarsys 拒。这一步很多团队第一次接都会卡住。

## 三、同步节奏：不是越实时越好

实时同步的诱惑很大——同意状态变更秒级到达营销系统，听起来很合规友好。但工程上要冷静：

**实时适用场景**：opt-out 动作。用户在 CDC 隐私中心点了"取消订阅"，这个信号必须秒级抹掉所有正在走流程的活动里的他。延迟一分钟，你就可能给一个刚撤回同意的用户发了第二封邮件，这是合规事故，不是用户体验事故。

**批量适用场景**：profile 属性变更。例如生日、城市、偏好品类。这些字段对营销分群有用，但晚 15 分钟没人会因此投诉。批量同步的对账成本远低于 webhook。

务实的做法是**两条管道并行**：opt-out 走 webhook（高优先级、低延迟、必须有死信告警），其他属性走 15 分钟一次的批量。两条管道在 IS 里写成两个 iflow，复用同一套字段映射，但走不同的可靠性等级。

## 四、活动触发：External Events API 才是真正的活儿

很多人以为 CDC 同步进 Emarsys 之后，营销活动会"自动按 segment 触发"。这只对了一半——周期性活动确实按 segment 跑，但**事件驱动的活动**（购物车放弃、订单完成、生日提醒、积分到期）必须由外部系统通过 External Events API 主动调起。

Endpoint：

```
POST https://api.emarsys.net/api/v2/event/{eventId}/trigger
Header: X-WSSE: {上一节生成的 wsse 字符串}
```

Payload：

```json
{
  "key_id": "3",
  "external_id": "",
  "data": {
    "cart_value": 1280.00,
    "cart_currency": "USD",
    "abandoned_at": "2026-05-21T14:32:11Z"
  },
  "contacts": [
    { "external_id": "user_8821@brand.com" }
  ]
}
```

`key_id: "3"` 表示用 Field 3（也就是 Email Opt-In）作为联系人识别键——这里其实容易踩坑。常见的更稳妥做法是用 Field 31（External ID）作为识别键，因为邮箱会变、外部系统的 user_id 通常不变。`data` 字段是任意 JSON，会被注入到邮件模板的 personalization token 里。

整个调用链是：电商或会员系统检测到事件 → 落到事件总线 → IS 拿到事件 → 调 Emarsys External Events API → Emarsys 在 trigger 那一瞬间再去查这个 contact 的 Field 3、Field 47 是不是 opted-in，**未 opted-in 直接丢弃**。这就是 SAP 在文档里反复强调的"consent enforced at campaign-time"——不是数据进来时校验，是发送那一刻校验。

数据进来时校验只能保证不会发给"现在没同意"的人，但发不出"刚刚撤回同意"的人。后者只有在 trigger 时再查一遍才能拦住。架构上是个不小的差别。

## 五、回流监控：哪些信号必须看

集成跑起来之后，至少这几个指标要进监控板：

- **CDC → IS 的 webhook 失败率**：阈值 > 0.1% 就要看死信队列
- **Field 3 / Field 47 的状态翻转延迟**：opt-out 动作发生到 Emarsys 这边状态变更的 P95 时延，目标 < 60 秒
- **External Events API 的 4xx 比例**：4xx 主要是字段映射错或 contact 不存在；5xx 多半是 Emarsys 侧抖动
- **trigger 之后被 consent gate 拦下的量**：这个数字代表"如果不在 trigger 时校验你会多发的邮件量"，是给合规团队看的可见性指标

最后一个指标特别重要。如果你的"被拦下量"长期是 0，要么是同步太及时（小概率），要么是 trigger 那一刻没真正去查 Field 3——后者意味着合规校验形同虚设，得回过头复核连接器配置。

## 六、出海企业的实操清单

如果你的业务已经在用 SAP CDC 做账号体系，要把 Engagement Cloud 接进来，工程上落地步骤大致是：

- 确认 Engagement Cloud 数据中心位置（欧洲 / 美洲 / 亚太），和你出海目标市场的数据驻留要求对齐
- 在 IS 上规划至少两条 iflow：opt-out webhook（实时）+ profile batch（15 分钟一次）
- 把 CDC preferences 一对多到 Emarsys Field 3 / Field 47 的压扁规则写进映射文档，和合规一起签
- API 凭证按最小权限：触发活动只需要 `externalevent.trigger`，不要给全权 token
- 为 trigger-time consent check 留可见性指标，写进交付验收

改名带来的产品定位讨论会有热度，但它不会改变上面这六层。事件总线、字段映射、同意传递、WSSE 摘要——这些才是这套系统真正吃工程量的地方。

---

*参考来源：SAP Community 技术博客《Integrating SAP Customer Data Cloud with SAP Emarsys for personalized, compliant marketing》（2026-01）、SAP Community《Trigger an Emarsys Email Campaign via API using Postman/Bruno and WSSE Authentication》。*
