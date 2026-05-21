---
title: "Service Cloud V2 给 case 重新分工"
date: 2026-05-21T23:10:00+08:00
draft: false
tags: ["SAP CX", "Service", "技术深度"]
description: "Self-Service、Case Management、Service Management 加上一个外部 Voice Agent，分工已经写在 case 生命周期上。"
source_url: "https://www.sap.com/topics/events/sapphire/innovation-news-guide-2026"
---

很多人看 Sapphire 2026 上 Service Cloud 那段更新，第一反应是"又加了几个 AI 助手"。但要把 Self-Service Assistant、Case Management Assistant、Service Management Assistant 三个名字摆在一起读，再叠上 Parloa 走 Agent Gateway 接进来这件事，会发现 Service Cloud V2 这次不是塞 AI 进现成模块，是把整个 case 生命周期重新切了刀。

切给谁看？切给做出海客服中心的架构师看——尤其是那种坐席已经分布在欧洲、北美、东南亚，对话量大、SLA 严，没法再往里堆人头的项目。

## 先看分工，不是看产品名

原文给的是产品蓝图，但产品名背后是 case 生命周期的四个阶段：客户提问 → 自助先过滤 → 转坐席处理 → 事后看绩效。三个 Assistant 各自落在一个阶段：

- **Self-Service Assistant**（Q2 2026 GA）：客户进来先不进 case 队列。它读订单历史、服务权益、财务记录，能答的当场答，答不了再升级为 case 进入坐席队列。
- **Case Management Assistant**（Q2 2026 GA）：坐席端的助手。case 进来后帮着分析上下文、协调跨服务组的工作流，让坐席不用自己翻五个系统拼上下文。
- **Service Management Assistant**（Q3 2026 GA）：管理者视角，跟踪客户情绪走向、给服务绩效洞察。这是事后视角，不进 case 处理路径。

还有一个 Agentic Case Resolution 已经 GA，作用是分析新 case、检测重复、推荐路由、起草回复。这个东西看起来像是 Case Management Assistant 的子集，但定位完全不同——它是引擎层的能力，三个 Assistant 都可以调它。

所以 Service Cloud V2 的真实分层不是"客服模块 + 几个 AI"，是这样：

![Service Cloud V2 在 Agentic 时代的能力分层](https://architecture.learning.sap.com/docs/ref-arch/ca1d2a3e)

## Parloa 为什么不放进 Service Cloud 内部

Sapphire 上 SAP 把 Parloa 单拎出来讲，措辞很谨慎："embed Parloa's AI agents within the SAP Service Cloud solution"——是嵌入，不是替换。Parloa 干的是语音和数字渠道前置 AI，对话从头到尾它接，但接到一半发现要开工单，工单这件事不是它做。

为什么这么切？因为对话识别和工单系统是两套截然不同的工程问题。Parloa 解决的是 ASR + LLM + TTS 端到端体验，要求 200ms 内的低延迟、自然轮换、口音容错。而 case 创建要走业务流程：服务权益校验、SLA 计算、工单路由、跨产品 entitlement 联动——这些放在对话引擎里跑，既慢又乱，更没法审计。

SAP 的解法是让 Joule 当中央调度。Parloa 通过 A2A（Agent-to-Agent）协议接进 Joule 的 Agent Gateway，对话过程里只要识别出"客户在报障"这个意图，意图就交回 Joule，Joule 再触发 Case Management Assistant 写工单、调度 Agentic Case Resolution 分配优先级。

```
Joule: Central AI copilot ... Routes requests to agents and skills,
manages conversations and enables bidirectional A2A communication
through the Agent Gateway (inbound) and Joule Capabilities (outbound).
```

注意 Disclaimer 里写得很清楚：Agent Gateway 还没 GA，目前只支持单向（outbound），双向通信能力等正式版本。这意味着今天落地 Parloa 集成，得接受过渡期方案——大概率是 webhook + 自定义中转，而不是原生 A2A 双向。

## 和 S/4 的资产数据怎么联

Self-Service Assistant 能"基于订单历史、服务权益、财务记录"答疑这件事，不是简单接 OData 接口。考虑这种典型场景：客户问"我上次买的那台设备保修还剩多久"。要回答这个问题，Assistant 需要拉到：

- S/4HANA 里的 equipment master + 服务合同 entitlement
- Sales Cloud 里的历史订单
- BDC 里聚合好的客户 360 数据产品

SAP 的标准答案是 BDC 提供 Data Products 作为 grounding 数据源，Assistant 调用 Generative AI Hub 的 orchestration service 做 RAG。架构上是干净的，但实施时要考虑两件事：

- **Data Product 的覆盖度**。如果 S/4 是非标的、equipment master 字段做了大量客制，先做 Data Product 建模再谈 Assistant，否则 Assistant 拉到的数据是残的。
- **实时性边界**。BDC 的 Data Products 不是实时同步，对"保修还剩多久"这种半静态数据没问题，但对"我刚下的单到哪了"这种实时查询，得让 Assistant 直接调 OMF V3 的 async API，不能走 BDC。

## 和老一代 C4C 的设计差异

老一代 SAP Cloud for Customer 时代，case 路由靠 routing rules + workflow，本质是 if-else 的可视化。坐席端的 AI 是 Service Ticket Intelligence 这种独立的预测模型，跟核心流程是松耦合的。

V2 这次的差异在于：AI 不再是"旁挂"，是嵌进 case 生命周期的每一个状态转换。Agentic Case Resolution 已经 GA 这件事很关键——它不是 Q3 2026 才有的产品计划，是现在就在线上跑的能力。这意味着 SAP 选择把 case 路由的"脑子"从规则引擎换成 AI Agent，规则引擎降级为兜底。

竞品对比放一句：Salesforce Service Cloud 的 Einstein Service Agent 走的也是 Agent Gateway 类似的路子，但 Salesforce 把对话引擎和工单引擎合在 Agentforce 一个产品里。SAP 选择把对话引擎让给 Parloa（外部）+ Joule（编排），自己只守住 case 处理引擎和数据层——这是一条更"让出前端、守住数据"的路线。

## 什么样的项目现在该考虑 V2

出海制造企业、跨国服务运营、跨境电商售后中心——这三类是适合的。共同特征是：客户分布在多时区、对话量大但同质化高（80% 是发货状态、保修、退换货之类问题）、坐席成本是大头、且业务数据已经在 SAP 体系里。

不适合的：纯内贸品牌不用考虑——Service Cloud V2 没有国内数据中心。已经深度定制 C4C 的客户，迁 V2 等于重做，要算清楚迁移成本。case 量很小（一年几千张）的项目，Self-Service Assistant 那点 ROI 撑不起 BDC 的建模工作量。

## 踩坑警示

- **不要等 Q3 一起上**。Self-Service Assistant 和 Case Management Assistant 是 Q2 GA，先把这两个跑通。Service Management Assistant 偏报表向，晚一个季度上不影响业务。
- **Parloa 集成今天还是过渡期方案**。Agent Gateway 双向通信未 GA，意味着对话状态回写工单、工单事件触发 Parloa 主动外呼，这两个方向要写自定义中间层。等 GA 之前，把 Parloa 当成纯入站渠道用更稳。
- **Data Product 不能临时做**。如果项目还没启动 BDC，Self-Service Assistant 的体验会很卡——要么数据拉不全，要么调 S/4 直接接口拉得慢。建议把 BDC 数据产品建模放在 Service Cloud V2 上线前一个里程碑。
- **知识库要先重整**。三个 Assistant 都依赖 grounding 数据，但很多客户的知识库是 Word 文档堆在 SharePoint 里。这种素材进了 RAG 等于拉低天花板。先把知识库结构化、加元数据，再谈 Assistant 上线。

Service Cloud V2 这一波的判断很清楚：客服中心未来不再是"一群坐席加一个工单系统"，而是"三个 Assistant 加一个 Voice Agent 加 Joule 编排"。Service Cloud 这条线把自己定位成 case 引擎和数据层，把对话前端让给 Parloa 这种专精厂商。这种解法对 SAP 客户来说不一定最酷，但最稳——因为 case 处理和数据治理是 SAP 真正擅长的事，对话体验本来就不是。

参考来源：
- https://www.sap.com/topics/events/sapphire/innovation-news-guide-2026
- https://architecture.learning.sap.com/docs/ref-arch/ca1d2a3e
