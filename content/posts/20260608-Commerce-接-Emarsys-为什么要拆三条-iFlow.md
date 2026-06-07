---
title: "Commerce 接 Emarsys，为什么要拆三条 iFlow"
date: 2026-06-08T10:00:00+08:00
draft: false
tags: ["SAP CX", "Engagement Cloud", "技术深度", "CPI", "Emarsys"]
description: "SAP 给 Commerce 和 Engagement Cloud 的标准集成方案，是把鉴权、流控、字段契约拆成三条独立的 iFlow——这套姿势看起来啰嗦，但每一段都对应一个具体的工程问题。"
source_url: "https://community.sap.com/t5/crm-and-cx-blog-posts-by-members/sap-emarsys-integration-with-social-media-apps-for-marketing-and-cpi-flow/ba-p/13776551"
---

很多 SAP Commerce 项目里，"把客户和订单同步到 Emarsys"被当作一行连接器配置写进文档。真正落地时，几乎每个团队都要重新发明一套链路：直连 API 撞到限流，丢消息没人发现，证书过期一夜停摆。SAP 自己给出的这套 CPI 集成方案，不长，但每一段都解决了一个具体问题。

这篇就拆这条链路：从 Commerce 的一次客户创建事件，到 Engagement Cloud（前身 Emarsys）的 Contact API 之间，三条 iFlow 加一个 JMS 队列，到底各管什么，为什么必须分开。

## 为什么不直连

最直观的方案是 Commerce 起个事件监听，把客户对象转成 Emarsys 格式，直接 POST 到 Engagement Cloud 的 Contact API。能跑通，但只能跑通一次。

第一个问题是流量节奏。Commerce 的客户创建事件可能集中在大促开账时刻，Engagement Cloud 的 API 有租户级 QPS 限制——同步直连意味着 Commerce 这一侧要处理 429 重试、退避策略、失败补偿。这些不是 Commerce 该操心的事。

第二个问题是契约耦合。Emarsys 的 contact 字段、API 路径、认证头格式会演进。每一次演进，Commerce 都要发版，这条链路越长越脆。

第三个问题是认证。Engagement Cloud 用 WSSE Header 鉴权，Header 里要拼 nonce、timestamp、password digest 三段动态值。把这段逻辑塞进 Commerce 的扩展点里，相当于让业务系统持有第三方系统的密钥——证书一轮换，业务代码就得重新发布。

SAP 的解法是把这三件事推到 CPI（Cloud Platform Integration）里，并且拆成三条独立的 iFlow。

## 三条 iFlow，一条队列

完整的链路是这样：

![CPI iFlow 架构图](./diagram.png)

- **iFlow #1（Receive Contacts）**：接收 Commerce POST 过来的客户对象，做字段映射转成 Emarsys 期望的 contact 格式，写进 JMS 队列。
- **JMS Queue**：CPI 内置的消息队列。这里是异步边界——Commerce 写完队列就返回成功，下游处理节奏与上游解耦。
- **iFlow #2（Poll Queue）**：按预设节奏从 JMS 拉消息，每条触发一次 iFlow #3。
- **iFlow #3（Generate WSSE Header）**：每次调用前生成 WSSE Header，把 payload POST 到 Emarsys 租户。

这个拆法看起来啰嗦——一次客户同步要过四个组件。但它把三件不一样的事情完全分开了：业务字段映射、流量节奏控制、第三方鉴权。

## JMS 那一段为什么是关键

iFlow #1 写完队列就返回成功，对 Commerce 来说这次同步已经"完成"。真正的 API 调用发生在异步消费侧。

这有几个直接的好处。一是吞吐解耦：Commerce 短时间涌入一万个客户事件，全部进队列，Emarsys 这边按自己的速率慢慢消费，不会触发 429。二是失败隔离：iFlow #3 调 Emarsys 失败时，消息退回队列重试，对 Commerce 不可见。三是观测点集中：所有需要重发的消息都在 JMS 里堆着，监控告警挂在队列长度上就够了。

代价是顺序性变弱了——同一个客户的多次更新到 Emarsys 那侧未必严格按时间序到达。要不要保序，看业务能不能接受最终一致。零售场景下大多数能接受，所以 SAP 默认走这条。

## WSSE Header 为什么要单独做一段

Engagement Cloud 的 API 用 WSSE UsernameToken 鉴权——业内偏老的一套规范，Header 大致长这样：

```
X-WSSE: UsernameToken Username="api_user",
       PasswordDigest="Base64(SHA1(nonce + created + secret))",
       Nonce="Base64(随机字节)",
       Created="2026-06-08T03:14:00Z"
```

每次调用都要重算 nonce、timestamp、digest。这段逻辑放在哪很关键。

放在业务 iFlow 里，业务流和鉴权混在一起，证书或密钥一轮换就要改业务流。SAP 把它独立成 iFlow #3，只做一件事：拼 Header、转发请求。业务流（iFlow #1、iFlow #2）只调它，不知道密钥也不知道证书。这是个标准的 sidecar 思路，CPI 上很少有团队第一版就这么写，但维护到第二年就会感谢这个拆法。

## 起步前必须先做完的两件事

这条链路上线前有两个硬前提，原文写得很轻描淡写，但漏掉任何一项整条链路都跑不通。

- **HTTPS 证书互导**：Commerce ↔ CPI ↔ Emarsys 三方都要走 HTTPS，三方各自的根证书或 leaf 证书要导入对端的 keystore。这一步在公司有强网络管控的环境里能耗掉两三周——证书签发流程、运维审批、轮换计划都要先搭出来。
- **Security Material 配置**：按 Emarsys API 文档在 CPI 里建 Security Material，存 API 用户名和密钥，iFlow #3 通过引用 Material 名称取值。密钥不能硬编码进 iFlow——这条是合规底线。

## 这套架构适合谁

几个判断点：

- 已经在用 SAP Commerce Cloud + Engagement Cloud 组合，且接客户/订单/事件三类数据中至少两类。Spadoom 给出的 native connector 路径适合最简单的场景——但只要有任何字段映射或业务规则要插，就得回到 CPI 这套。
- 同步规模在每天数千到数十万条客户/订单事件之间。再小不值得拆这么细，再大要重新评估 JMS 的吞吐上限以及是否切换到 SAP Integration Suite 的事件网格（Event Mesh）。
- 接受最终一致——同一客户的两次更新到 Emarsys 那边可能乱序。如果业务必须严格保序（比如积分扣减），这条链路要加 partition key 或者直接走单线程消费。

不适合的场景也很清楚：实时性要求亚秒级的（比如下单后立即触发 abandon cart 营销）、事件量低到一天就几十条的、或者还在自建 CRM 的——前者不该走 JMS，中者拆这么多 iFlow 是过度设计，后者根本不该上 Engagement Cloud。

## 几个容易踩的坑

- **JMS 死信处理**：iFlow #3 调用失败到一定次数，消息要进死信队列而不是无限重试。CPI 默认配置不一定打开这个，第一版上线前必须确认。
- **WSSE 时间戳容差**：Created 时间戳与 Emarsys 服务器有几分钟的容差，CPI 节点时区不对或者 NTP 不同步会出诡异的 401。
- **字段映射的版本化**：iFlow #1 里的字段映射不要直接写死，用 Value Mapping 或外置配置——业务侧加自定义字段是常态，每次都改 iFlow 太重。
- **证书轮换的演练**：Emarsys 那侧证书更新通常会提前通知，但 CPI 这边的 keystore 替换流程必须演练过。生产环境第一次换证书出错的概率不低。

SAP 在文档里把这套方案叫作"使用 CPI Flow 的 Emarsys 集成"，听起来像是一个工具说明。实际上它是 SAP 给 Commerce + Engagement Cloud 的标准对接姿势——把鉴权、流控、字段契约三件事拆开放在中间层，而不是堆在某一端。这套姿势对出海零售、跨境品牌、全球化品牌团队是有意义的：当你的 Commerce 和 Engagement Cloud 部署在不同 region、不同时区、不同合规域时，这条链路是可控点。

参考来源：https://community.sap.com/t5/crm-and-cx-blog-posts-by-members/sap-emarsys-integration-with-social-media-apps-for-marketing-and-cpi-flow/ba-p/13776551
