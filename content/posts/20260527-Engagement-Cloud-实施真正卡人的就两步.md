---
title: "Engagement Cloud 实施真正卡人的就两步"
date: 2026-05-27T11:15:00+08:00
draft: false
tags: ["SAP CX", "Engagement Cloud", "Emarsys", "技术深度"]
description: "改名后底盘还是 Emarsys。8 步实施里，关键路径只有两条：域名/IP 预热和交易数据接入。其他 6 步弄错都还能补救，这两步弄错要么炸主域名，要么实时触发场景全废。"
source_url: "https://www.spadoom.com/en/blog/sap-emarsys-setup-a-spadoom-guided-journey/"
---

做过 Emarsys（现在叫 SAP Engagement Cloud）项目的人都见过这一幕：业务方信心满满地说"我们要在 6 周内上线邮件、短信、Push 三个渠道，加上购物车放弃和会员日促销"。然后第三周还在等 DNS 团队改 SPF。第六周邮件总算发出去了，结果 60% 进了 Gmail 的垃圾箱。

上周读到 Spadoom 的 Dario Pedol 写的一篇 8 步实施指南，把这个过程拆得很清楚。我顺着他的逻辑捋了一遍，发现有一件事值得单独拎出来说：8 个步骤里其实只有两步是真正的关键路径。其他 6 步弄错了还有补救空间，这两步弄错——要么烧主域名声誉，要么实时触发场景全部失效。

![SAP Engagement Cloud 8 步实施关键路径](https://www.spadoom.com/en/blog/sap-emarsys-setup-a-spadoom-guided-journey/)

## 为什么改名之后底盘没动

SAP 在 2026 年 2 月 19 日把 Emarsys 重新打包成 SAP Engagement Cloud，并推出 Enterprise Edition。但对实施团队来说，租户结构、API、Web Extend 脚本、Sales Data 通道——一个没换。这不是品牌包装那么简单：SAP 没动底盘，是因为 Emarsys 这套东西本来就是为高频营销场景设计的，多租户隔离、IP 预热、deliverability 工程都是十几年磨出来的，重写代价太大。

真正变化的是定位。改名之后这个产品不再是"营销云的一个模块"，而是被放在 BDC、Commerce、Sales、Service 这些产品中间的实时触达层。但站在实施视角，这层语义变化不影响动手——你要做的事和三年前一样。

## 关键路径一：域名与 IP 预热

Step 2 是大多数项目的第一道坎。它需要四件事按顺序到位：

- 一个专用发送子域名，比如 `mail.yourcompany.com`，**不要用主域名**
- SPF 记录指向 Emarsys 的发送基础设施
- Emarsys 生成 DKIM 公钥，发布到你的 DNS
- DMARC 策略起步设 `p=none`，预热完成后再收紧到 `p=quarantine`
- MX 记录处理退信

为什么必须用子域名？这个问题真的有人问过我。答案很直白：IP 预热是个有概率出问题的过程。新 IP 在 Gmail/Outlook 眼里是"陌生人"，前两周如果你的内容触发了垃圾邮件信号、列表里有大量无效邮箱，整个发送域的声誉就会被打上烙印。如果你用的是主域名，那条烙印会跟着所有从这个域名发出去的邮件——包括 CTO 发给客户的合同邮件、HR 发给员工的入职通知。

> 这件事我见过真实案例：某零售客户图省事用主域，预热第二周硬是把 main domain 的 sender reputation 打废了，整个公司外发邮件被 Gmail 全量进 spam。回滚花了三周，期间所有官方邮件靠人工分发。

IP 预热本身没什么花哨的，就是按 Emarsys 给的曲线慢慢加量，2–4 周拉满。这一步没有捷径——你不能用钱买，不能用关系绕，Gmail 的反垃圾团队也不会因为你是 SAP 客户给特殊待遇。

一组典型的 DNS 配置长这样：

```
; 子域名 SPF（发送方授权）
mail.yourcompany.com.   IN TXT  "v=spf1 include:emsmtp.com ~all"

; DKIM（Emarsys 生成 selector，你发布公钥）
emarsys1._domainkey.mail.yourcompany.com.  IN TXT  "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQE..."

; DMARC（预热期 none，后续 quarantine）
_dmarc.mail.yourcompany.com.  IN TXT  "v=DMARC1; p=none; rua=mailto:dmarc@yourcompany.com"

; MX（退信处理）
mail.yourcompany.com.   IN MX 10 bounces.emsmtp.com.
```

Google 从 2024 年起对发送量超过 5000/天的发件方强制要求 DMARC 与 List-Unsubscribe header，这不是 Emarsys 的要求，是收件方的硬门槛。

## 关键路径二：交易数据怎么进来

Step 7 决定了你后面所有"实时触发"场景能不能跑起来。Emarsys 给了两条路：Sales Data API 和 SFTP 文件上传。文章里说推荐用 API，但很多项目还是会因为各种理由选 SFTP——我见过最常见的理由是"我们 ERP 那边的同事不愿意写 API 对接"。

这两条路的差别不在技术难度，而在**触发场景的语义**：

- **Sales Data API**：订单创建那一刻，事件就到 Emarsys。Welcome、Cart Abandon、Post-Purchase 这类时间敏感的 journey 都依赖它
- **SFTP 批量**：典型每小时或每天一次。Cart Abandon 的"30 分钟没结账就发提醒"在这个模式下根本无法实现，因为数据本身就有 1 小时延迟

如果你已经在用 SAP Commerce Cloud，恭喜——原生连接器把 Sales Data API 的事情都替你做了，订单事件实时流到 Engagement Cloud 不需要自己写代码。但如果你的下单系统是其他电商平台，或者是 S/4HANA Sales Order，就需要走 CPI（Cloud Integration）做中间层，把订单事件转成 Emarsys 能吃的 JSON 推过去。

Sales Data API 的事件大致长这样：

```json
POST /api/v2/sales/transactions
{
  "customer": {
    "id": "CUST-88291",
    "email_hash": "5e884898da280..."
  },
  "items": [
    { "product_id": "SKU-RED-M", "quantity": 1, "price": 89.00 },
    { "product_id": "SKU-BLU-L", "quantity": 2, "price": 65.00 }
  ],
  "currency": "USD",
  "occurred_at": "2026-05-27T10:42:11Z",
  "channel": "web"
}
```

这里有一个细节值得注意：customer 段落里的 email_hash 不是加密 token，是 SHA-256 散列。Emarsys 用它做匿名访客和已识别用户之间的串联——Web Extend 脚本在浏览器端先把邮箱 hash 一遍上报，订单事件也用同样的 hash，两边匹配上才能把购物车放弃事件归到具体用户身上。

## Web Extend 怎么挑身份策略

Step 4 的 Web Extend 是 Emarsys 自家的前端追踪脚本，跟 Google Analytics 类似但作用不同——GA 给你看流量趋势，Web Extend 给营销云做**分群和触发**。装一段 JS 在所有页面，它就开始记录浏览路径、加车、搜索关键词。

真正需要决策的是"用什么标识访客"。文章给了两个选项，但实际项目里需要按登录率分类：

- 登录率高的站点（B2B、会员制零售）：用你自己 CRM 的 customer ID。最干净，跨设备识别也准
- 登录率低的站点（DTC 快消、首次访问的旅游）：用 hashed email + cookie fallback。匿名访客用 cookie 跟踪，邮件订阅时把 cookie ID 与邮箱做关联

选错的代价是分群失效。曾经有个客户登录率只有 8%，但坚持用 CRM ID 当唯一标识，结果 Web Extend 抓到的浏览数据里 92% 都挂在 anonymous 桶里，purchasing intent 模型完全没数据吃。

## 渠道顺序：为什么必须先邮件

Step 6 是激活渠道。Email 是默认开的，SMS 需要本地合规（每个国家的 opt-in 规则不同）、Push 需要你的 App 集成 Emarsys Mobile SDK 并配 APNs 证书和 FCM key。文章里有句话我很认同：

> Get email solid first. Add SMS and push once email is stable.

原因不是渠道难度递增，而是**问题诊断成本**。邮件如果有问题，你能从 Emarsys 后台看到 bounce、open、click，从 inbox 端看到送达情况。SMS 如果出问题，你只知道运营商返回的状态码，看不到用户是不是真收到。Push 更糟——iOS 用户关闭通知你完全不知道。

先把邮件这条路跑顺，意味着你已经验证了：用户数据流通、个性化 token 替换正确、tracking 链路完整、unsubscribe 处理对了。这些验证完，再加 SMS 和 Push，问题就只可能出在新渠道本身，定位成本指数级下降。

## 什么样的项目适合上这套

给三类对象的判断基准。

**出海品牌（DTC、跨境电商、消费品出海）**：值得上。Engagement Cloud 在 EU 和 US 都有 data center，GDPR 合规、CCPA 合规都打包好了；Web Extend 全球加速、Email 送达率有专门的 IP 池；Commerce Cloud 原生连接器让你省掉 60% 的集成工作量。**判断点是发送量**：如果月活邮件账号低于 50 万、年发送量低于 600 万封，HubSpot 或 Klaviyo 的 ROI 更好；超过这个量级，再考虑 Emarsys 的 Tactics 库和多语言模板优势。

**跨国制造企业的 B2C 业务线**：值得，但要做边界设计。比如某汽车主机厂的官方商城用 Engagement Cloud 做会员触达，但保留经销商系统的客户独立管理。这种场景下 Web Extend 只装在直营商城，不要试图覆盖经销商门店。

**在华外资企业的中国本地化业务**：要谨慎。SAP Engagement Cloud 在中国大陆没有 data center，数据出境合规需要单独评估。如果你的中国业务客户数据必须留在境内，这条路走不通——这种情况下中国区域往往会另选一套本地的 marketing automation 与之并行。

**纯内贸品牌**：直接说不要看了。除了 Commerce Cloud，SAP CX 的其他产品都没国内数据中心，Engagement Cloud 没法在国内卖。市面上替代方案不少，没必要绕远路。

## 几个真实踩过的坑

- **用主域名发送**：预热翻车一次，整个公司外发邮件全部进 spam。永远用专用子域名
- **跳过 IP 预热**：Day 1 直接发 10 万封，新 IP 直接被 Gmail 限流 30 天
- **导脏数据**：未清洗的联系人导入直接拉低 list health 分。bounce 率超过 5% 之后 Emarsys 会限制你的发送速率
- **四个渠道一起上**：问题混在一起，根本分不清是数据问题、模板问题还是 SDK 问题。先邮件，跑稳再扩
- **不装 Web Extend**：等半年后业务方说要做 browse abandon，发现没埋点，行为数据从零积累，模型还要冷启动 3 个月

## 一句话总结

SAP Engagement Cloud 的实施周期写的是 6–10 周。这 4 周差距，几乎全部体现在域名预热和交易数据集成两件事上——前者考验你能不能拿到 DNS 操作权，后者考验你的 ERP/Commerce 团队愿不愿意配合写 API。其余 6 个步骤再怎么翻车，也只是 +几天 而不是 +几周。

改名之后这个产品的工程难度没降，认知门槛反倒升了——之前还有"Emarsys"这块明确的招牌，现在叫 SAP Engagement Cloud，容易让 IT 误以为它就是 BDC、Commerce 那种 SAP 原生云产品。它不是。它是一个被 SAP 收购十几年、技术体系成型已久的独立栈，按 Emarsys 的逻辑实施它，比按"SAP 项目"的范式实施它，成功率高得多。

参考来源：https://www.spadoom.com/en/blog/sap-emarsys-setup-a-spadoom-guided-journey/
