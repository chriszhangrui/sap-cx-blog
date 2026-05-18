---
title: "Celonis 也来抢 AI 上下文了，SAP 的护城河还稳吗"
date: 2026-05-18T09:20:00+08:00
draft: false
tags: ["SAP CX", "SAP Business Data Cloud", "Celonis", "Agentic AI", "竞争分析"]
description: "Celonis 收购 MIT 出身的 Ikigai Labs 做 Context Model，直接对标 SAP 的 Operational Context 战略。同一个故事，两条路线，售前要回答一个新问题。"
source_url: "https://siliconangle.com/2026/05/12/celonis-buys-decision-intelligence-startup-ikigai-labs-provide-operational-context-enterprise-ai/"
---

今天看到一条消息值得聊聊：5月12日，Celonis 把一家叫 **Ikigai Labs** 的 MIT 出身的小公司收了。表面上是又一笔不大不小的 AI 收购案，但你把它放到这两个月 SAP 那一连串大动作的旁边一对照，味道就出来了。

这家 Ikigai Labs 是 2019 年成立的，CEO 是 MIT 教 AI 的 Devavrat Shah，主打的东西叫"大型图模型"——专门让 AI 看懂企业里那些杂七杂八的结构化数据。Celonis 收完之后，Shah 直接挂了 Celonis 首席科学家的头衔。

**Celonis 想做的事，名字叫 Context Model。** 说白了就是：把企业每个系统、每个流程、每条数据，翻译成 AI 能听懂的语言，做成一个实时的"运营数字孪生"。

Celonis 总裁 Carsten Thoma 接受 Computer Weekly 采访时说了一句话挺关键："AI 只能跟它拥有的上下文一样聪明。每个组织都需要给自己的企业 AI 一个完整的、活的业务运转模型，这件事以前从来没真正实现过。"

**熟悉吗？** 这话我们前两个月在 Sapphire 听 Christian Klein 讲过差不多一遍，措辞甚至更激进。Klein 当时说"AI 军备竞赛方向错了"，正确的方向是 Operational Context（运营上下文）。然后 SAP 一口气掏出 Reltio、Dremio、Prior Labs 三笔收购，再加上自研的 RPT-1.5 模型和 Knowledge Graph，一整套数据底座的故事就讲完了。

所以今天 Celonis 的这一手，本质上是在抢同一个叙事——**谁来当企业 AI 的"上下文管家"**。

**两条路线分得很清楚。**

- **SAP 走的是数据优先**：从 ERP 出发，靠 SAP Business Data Cloud (BDC) 把结构化的业务数据沉到一个底座，再让 Joule 上面的 Agent 去调。底层 LLM 选了 Anthropic Claude。
- **Celonis 走的是流程优先**：从流程挖掘出发，把"事情是怎么发生的、为什么会卡住、下一步该怎么动"做成一张实时图，AI 接进来就能用。

Celonis 的杀手锏是一句话：**"我对应用厂商无关、对系统无关。"** 这是直接冲着 SAP 打的。Celonis 的零拷贝集成同时支持 AWS、Databricks、Microsoft Fabric、Oracle，Agent 平台同时打通 Bedrock、Claude Cowork、Watsonx Orchestrate、Microsoft Copilot——SAP 强调"我家的 Agent 在我家的护城河里跑"，Celonis 是"你家系统都是我的工作台"。

**对售前来说，这事的意义在哪？**

首先，"AI 上下文"这个叙事再也不是 SAP 独家了。客户接下来跟你聊 Joule、聊 BDC，大概率会问"那 Celonis 那个 Context Model 跟你们是什么关系"。这个问题前两个月还没人问，下个月你的客户经理一定会被问到。

更刺激的是 SAP 旗下还有一个 Signavio——SiliconAngle 这篇文章里直接点名 Celonis 的主要竞争对手就是 SAP Signavio、IBM Process Mining 和 UiPath。也就是说 Celonis 这一手不只是新增一个 AI 故事，而是顺手在 SAP 自己的流程挖掘地盘上又插了一刀。

**有个细节很值得玩味。** Celonis 的早期客户案例里，Cardinal Health 的 CTO 说了一句话："我们这个行业接受不了'只是大多数时候是对的'的 AI。精度是底线。"——翻译一下，这就是在帮 Celonis 立"我比你更适合做生产级 AI"的人设。SAP 这边讲了 Knowledge Graph 配 RPT-1.5 也是同一个意思，但 SAP 的故事到目前为止落地案例还少。

**我的判断是这样**：未来 12 个月，"运营上下文"会变成所有企业级 AI 厂商的必答题。SAP 的护城河在 ERP 数据本身，Celonis 的护城河在流程图谱，Salesforce 那边大概率要从 Data Cloud 再加码补一刀。客户最后不会只选一个——但他们会问一个让售前很头疼的问题：**"我的 Context 主权放在哪一家？"**

这个问题没有标准答案。但躲是躲不过去的。

参考来源：https://siliconangle.com/2026/05/12/celonis-buys-decision-intelligence-startup-ikigai-labs-provide-operational-context-enterprise-ai/
