---
title: "SAP 把 voice 和 chat 让给了别人，Service Cloud V2 的边界在收窄"
date: 2026-06-08T19:30:00+08:00
draft: false
tags: ["SAP CX", "Service", "技术深度"]
description: "Sapphire 2026 一边升级 Parloa，一边把 Amazon Connect 接进 Service Cloud。两件事放一起，是 Service Cloud V2 把通道层让出去。"
source_url: "https://aws.amazon.com/blogs/awsforsap/sapphire-2026-how-aws-is-helping-sap-customers-move-faster-and-build-more/"
---

Sapphire 2026 期间，SAP 关于 Service Cloud 的两条消息其实可以放在同一张图里看。

一条是和 Parloa 把战略投资升级成产品深度集成，AI 语音和数字 Agent 直接进入 Service Cloud；另一条是和 AWS 宣布把 Amazon Connect Customer 与 Service Cloud、ESM 联起来，路径是 MCP 配 SAP Integration Suite 配 Bedrock AgentCore。两件事如果只各自看，会读成"又一个集成"。但放在一起，传出来的信号其实更结构性：Service Cloud V2 把 voice 和 chat 这一层真的让出去了。

## 通道层为什么会被切出去

老一代 C4C/Cloud for Customer 时代，SAP 是想把通道层做完的。Live Activity Center、Email Channel、Chat、社交媒体接入，这些组件都嵌在产品里面，由 SAP 自己维护连接器、自己调对话引擎。逻辑很直接：客户在哪里说话，平台就要在哪里听。

这条路在 Agentic AI 出现之后变得很难走。一个真正能"对话+完成任务"的语音 Agent，需要的不只是 ASR 和 TTS，还要 turn-taking、barge-in、超低延迟语音通道、跨 140 多种语言和方言的语义模型、电话号码资源、合规的录音存储。这些每一条单拎出来都是一个完整的产品赛道。Parloa 做的事情就是这个层，AWS 这边对应的是 Amazon Connect 这套联系中心栈。SAP 自己做，等于和这两类公司在它们最强的赛道上打。

所以 Sapphire 2026 给出的答案是收手——Service Cloud V2 不再亲自做语音和聊天的会话引擎，把"通道+对话"这一段开放给生态。自己守住的是 Case、Entitlement、Knowledge 这三块离业务数据最近的核心层。

## 三段式架构的样子

![Service Cloud V2 通道层架构示意](http://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLUicVbHqbGcR2eIkncqhrPI1IjpoojTtjwLjhC5eQKEzaHGWUlNpNyVemwVddB5lC2AySjOKwEclkCshY1iaWxygbC4siakgOAh0Q/0?from=appmsg)

通道层（Channel Layer）放外部 Agent 平台。Parloa AMP 这边走的是预置集成加低代码配置，开发者不需要为每条业务事件写连接代码；Amazon Connect Customer 这边走的是 MCP 协议，通过 SAP Integration Suite 进入 SAP 的业务对象。原生通道（email、webform）保留下来，但 SAP 文档里写过出站邮件通道有 20 个的硬上限，多于 20 个要走 Custom SMTP——这个数字本身就说明原生通道层的设计目标已经不是"承担一切"。

中间是核心层（Core Layer），就是 Service Cloud V2 真正不动的部分：Case Management（Case Type Determination、路由规则、Agent Inbox 这套统一工作台），Entitlement & SLA（合同等级和服务承诺判定路由优先级），Knowledge Base Provider（KB 接口对外开放，让外部 Agent 可以直接查询知识库）。这三块共同决定的是：一通电话进来要不要建 case、建什么类型的 case、走什么 SLA、由谁处理、回复模板从哪来。

最底下是数据层（Data Layer），S/4HANA 提供 Registered Product 和资产数据；Business Data Cloud 通过 Knowledge Graph 给 Joule Agent 提供语义关系；FSM 接外勤工单，集成方式可以是 Event Mesh 实时推送，也可以是没买 Event Mesh 时退而走 CPI Polling 拉取（这条退路 SAP 社区前阵子有过完整文档）。

## Parloa 路径和 Amazon Connect 路径有什么不同

两条集成路径表面上目的一样——把语音和数字 Agent 接到 Service Cloud——但走的协议和数据契约不一样。

- **Parloa** 走的是预置 connector + webhook。一通来电，Parloa AMP 先拉 Service Cloud 里的客户上下文（订单、open case、entitlement），用语义模型理解对话，对话过程中需要执行业务动作（建工单、改预约、查发货）就调 Service Cloud 的 OData/REST API，全程信息沉到 case 历史和 timeline 上。这条路的好处是出箱即用，对客户来说接入快；代价是数据契约由 Parloa 主导，扩展点跟着 Parloa 节奏走。

- **Amazon Connect Customer** 这条线走 MCP（Model Context Protocol）。Bedrock AgentCore 上跑的 Agent 通过 MCP 描述 SAP 业务对象的 schema，调用方式由 Integration Suite 中转。这条路对自建 AI 平台的客户更友好——同一套 MCP 工具集可以同时被 Amazon 的 Agent 用，也可以被 Joule 用，也可以被 Anthropic Claude 这类外部 Agent 用（Sapphire 2026 同时宣布了 Anthropic Claude 进入 SAP AI Foundation 模型池）。代价是配置链路更长，schema 治理在客户和 SI 这边。

两条路本质都是把 Service Cloud 当后端、把外部平台当前端，区别只是协议契约和谁主导扩展节奏。这一点对正在评估 Service Cloud V2 的中国出海企业有直接影响——你选的是"把通道层外包给一家成熟的 CCaaS"还是"自己组装一个 Agent 联系中心"。

## 代码长这样

外部 Agent 想给 Service Cloud 写一条 case，最常用的还是 OData v4 over REST。一个简化的请求长这样：

```http
POST /sap/c4c/odata/v1/c4codataapi/CaseCollection
Authorization: Bearer <OAuth2 token from BTP>
Content-Type: application/json

{
  "Subject": "Refund inquiry from voice agent",
  "ServiceCategoryID": "RETURNS_L1",
  "CaseTypeCode": "ZSRV",
  "ReporterPartyID": "CUST_8821001",
  "RegisteredProductID": "SN-7791",
  "ChannelTypeCode": "VOICE",
  "ExternalReference": "parloa_call_8e2c41",
  "Description": "Customer requested refund on order 47001..."
}
```

这条 POST 进来之后，Case Type Determination 引擎先按规则识别 case type（这一步现在可以挂 Joule Capability 做语义分类，不再纯靠正则），然后路由引擎按 entitlement 和团队负载分到队列，Agent Inbox 把它呈现给坐席。整条链路的运行时还是 Service Cloud V2 自己的，外部 Agent 只是触发者。

反过来，Service Cloud 想把"客户来电时主动通知"这种事件推给 Parloa，走的是 Webhook——在 Solution Guide 里有专门一节 Configure Webhook，订阅 case status change、SLA breach 这类事件，目标 endpoint 就是 Parloa AMP 那边。整套链路其实把 SAP 推到了 event producer 的位置，处理逻辑放在通道层。

## 什么样的项目适合走这条路

- **跨国客服中心、出海 B2C、跨境电商售后**——这一类用户量大、语种多、要求 7×24 含语音的场景，最值得走 Parloa 路径。Parloa 一上来就支持 140+ 语言和方言，省去了你在 BTP 上自己拼语音 stack 的成本。

- **已经在 AWS 上有联系中心栈的全球化制造企业**——AWS Connect 路径更顺，MCP + Integration Suite 这套链路可以把 Service Cloud 当成"业务数据后端"放到现有 Amazon Connect 流量里。前提是你有 AWS 这边的运维能力。

- **只在中国做内贸的纯本土品牌不在这场讨论里**——SAP Service Cloud 没有国内数据中心，Parloa 和 Amazon Connect 这两条新通道也都不会落地国内合规节点。这个组合是给有海外业务、跨境运营、或在华外资总部对接的企业看的。

## 什么时候不要碰

如果项目里 case 来源 90% 都是 email、单语种、量不大，没有必要去搬这套通道层。Service Cloud V2 自带的 inbound email channel 已经足够，再叠 AI Agent 就是过度设计。还有一种情况——entitlement 和 SLA 还没建好，路由规则还没沉淀清楚——这时候即便接了 Parloa，AI Agent 拿不到清晰的判定逻辑，对话再聪明也只是个客服 Demo。先把核心层的数据契约打磨干净，再去开通道层，顺序不能颠倒。

## 最后一段

SAP 把通道层让出去这件事，其实和早期 Composable Storefront 让出前端、OMF 让出订单中枢同一类动作——把自己最不擅长但又必须有人做的那部分交给生态，自己守住业务数据和流程治理。Service Cloud V2 的边界正在收窄到 case、entitlement、knowledge 这三件事上。这是个清晰得多的产品，但也意味着 SI 和 IT 团队要重新学一种集成方式：你的项目交付物，从前是搭一个完整的客服系统，现在是把通道层、核心层、数据层这三段拼起来，每一段的供应商还可能不同。这种拼装能力，未来几年会比单一产品的实施能力更值钱。

参考来源：
- https://aws.amazon.com/blogs/awsforsap/sapphire-2026-how-aws-is-helping-sap-customers-move-faster-and-build-more/
- https://news.sap.com/2026/05/sap-sapphire-sap-unveils-autonomous-enterprise/
- https://www.parloa.com/blog/proactive-personalized-interactions-parloa-sap/
