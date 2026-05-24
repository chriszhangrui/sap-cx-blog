---
title: "Joule Studio 想解决的不是写代码"
date: 2026-05-24T23:05:00+08:00
draft: false
tags: ["SAP CX", "AI", "Joule", "技术深度"]
description: "Sapphire 2026 公布的 Joule Studio，把意图驱动开发、SAP 业务上下文和托管运行时绑成一件事——它要解的不是写代码慢，而是从业务问题到能跑的 Agent 之间那段管线时间。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/joule-studio-from-business-intent-to-enterprise-ready-ai-agents/ba-p/14395010"
---

不少 AI 项目走过这样一条曲线：第一周写出能跑的原型，团队兴奋；第二个月被各种连接、上下文、合规拖住；半年后，预算烧完，交付了一套脆弱的、谁也不敢动的东西。

这不是 AI 的问题，是企业平台对 AI 开发这件事的姿势没调整过来。Sapphire 2026 上 SAP 把 Joule Studio 这个新开发环境推出来，公开的目标就一句话——开发者不要在 plumbing（管线接线）上花时间。能不能做到另说，但这一次架构画得很清楚，值得拆开看。

## 意图驱动：把 PRD 这一步做厚

大部分代码生成工具的卖点是"自然语言进、代码出"。Joule Studio 反过来，把中间最容易被跳过的那一步——产品需求文档（PRD）——做厚。

流程是这样：你用自然语言描述一个业务问题，比如下面这句来自原文：

> 我们的卡车有 25% 是空载，没法把货塞满，等于在运空气。一年损失大概 100 万欧元。帮我找个方案。

系统不直接出代码，先生成一份完整的 PRD，并主动从两个地方拉上下文：SAP Signavio 里的真实流程模型（瓶颈在哪、错单率多高），和 SAP LeanIX 里的架构画像（哪些系统已经接入、什么属于 clean core 不能动）。业务团队在 PRD 上改一轮，确认无误，系统再生成技术 SPEC——数据模型、API 契约、工作流逻辑、安全策略。最后才到代码。

原文里 The Value Chain 那位 SAP Mentor 的一句话很到位：The PRD makes the implicit explicit。把隐含假设显化，是这条流水线真正的价值——不是省了写代码的时间，而是把"做错的事重做一遍"的代价提前摁住了。SAP 自己给的数字是简单 Agent 15 分钟、复杂场景 90 分钟，关键耗时在 PRD review。

## Grounding 这一层：差异不在模型，在数据怎么进来

泛 LLM 平台和企业 AI 平台的真实差异，从来不是底层模型谁强，而是业务上下文怎么进入推理过程。Joule Studio 把 grounding 这件事拆成四块明确说清楚：

- **SAP Knowledge Graph**：客户、订单、发票、物料以及它们之间的关系，用语义图的方式喂给 Agent。Agent 不是在 query 一张表，而是在一个有业务语义的网络上推理。
- **SAP Domain Models**：基于 SAP 内部代码库训练，沉淀了"全球 77% 商业实施"的实现模式。代码生成不是猜，而是从真实模板里组装。
- **Signavio + LeanIX**：流程实况 + 架构边界。前者告诉 Agent 哪些痛点是真痛点，后者告诉它哪些系统不能碰、哪些扩展模式才合规。
- **Business Data Cloud（BDC）数据产品**：真实运营数据，不是 demo 数据集。这一条让 Joule Studio 和已有的 BDC 投资接上了。

这套设计要传递的一个判断是：grounding 必须从开发期就进来，不是 Agent 写完之后再"接一下数据"。后接的代价是连接质量永远不达标，前置的代价是平台必须把 Knowledge Graph、流程图、架构图这些资产都标准化好——这恰好是 SAP 在 ERP 里积累了几十年的东西。

## 双轨路径：Web 工作室 + IDE，同一个项目互通

关于"低代码 vs 专业代码"的二元争论，Joule Studio 给的答案是：同一个项目在 Web 和 IDE 之间无缝切换。

Web 端是低代码/无代码体验，业务团队和顾问可以直接画 Agent、配工作流。专业开发者拉一份代码到 VS Code、Cursor、Claude Code、Cline 任选一个 IDE 里继续——生成的代码是标准 CAP 后端、CDS 实体、Fiori Elements UI、n8n 工作流，不带 vendor lock-in，可以进 Git、可以走 CI/CD。

更值得注意的是协议这一层。Joule Studio 同时支持 MCP（Model Context Protocol，把 SAP 上下文以工具形式开放）和 A2A（Agent-to-Agent，让 Agent 之间互调）：

```
# MCP：把 SAP 业务上下文以工具形式暴露给外部 Agent
# A2A：让 Joule Agent 与第三方 Agent 直接对话

Joule Agent  ⇄  MCP Server  ⇄  Customer/Order/Material 语义图
     ↕
  A2A 协议
     ↕
外部 Agent（LangGraph / AutoGen / LlamaIndex / 自研）
```

这意味着如果团队已经在 LangGraph 上养了一套自定义 Agent，不必重写。它们可以通过 A2A 接进来，把 SAP 的业务上下文当作工具调用。这一步如果走通，会是 Joule Studio 与 Microsoft Copilot Studio、Salesforce Agentforce 拉开差距的地方——前者把 Agent 框架开放，后者把 Agent 框架自己锁死。

## 运行时与 Agent Hub：治理这次没"贴膏药"

很多 AI 项目卡在"代码能跑了，怎么部署"。Joule Studio 配套的 Joule Studio Runtime 是 SAP 托管的——不用自己维护 Kubernetes，一键部署，扩缩容、负载均衡 SAP 接管。Tracing、Metrics、Logging 默认开启，每个 Agent 的执行轨迹（请求、结果、单步决策）全部可追溯。

另一边，所有部署的 Agent 自动注册到 SAP AI Agent Hub。Hub 是 IT 与业务负责人共用的中枢——发现、监控、策略下发、审计，SAP 自带 Agent 和客户自建 Agent 一视同仁。这个设计回应了一个企业里很现实的焦虑：当组织里跑着几十上百个 Agent，没有一个集中视图，连"谁授权了什么"都说不清。

架构层面有一个判断要留意：治理面（Agent Hub）和运行面（Runtime）是分层的。Agent 在哪个 Runtime 跑、用什么底层 Infra，未来理论上可以解耦——SAP 只要把"注册 + 策略 + 审计"这三件事抓在 Hub 里，就保住了治理主权。这条路如果走得稳，Hub 会变成企业 AI 治理的"控制平面"，类似于 Service Mesh 之于微服务。

## 商业模型：设计期免费，运行期按消耗

这是原文里少有的、能直接拿去做内部预算评估的硬信息：

- **设计期工具**（Agent Builder、应用开发、自动化设计、MCP Builder、AI 辅助编码）：包含在 AI Units 或 Joule Base（$0 SKU）权益里，不额外计费，附带"公平使用"的 build-and-test 运行配额。
- **托管运行时**（生产环境跑 Agent、应用、工作流）：按资源消耗计费。
- **合作伙伴**：通过 TDD（Test, Demo, Development）许可拿到完整设计期工具和有限运行时，用于内部测试和 Demo，不能跑客户生产负载。

这个分法有一个隐含信号：SAP 想把开发门槛降到零，让客户和合作伙伴愿意"试一下"，把生产负载的钱留到运行时这一层收。短期对生态友好，长期会把成本结构推到运行时一侧——做项目时要做的功课，是把 Token 消耗、Agent 调用频次、上下文规模都纳入 TCO 测算，不能只算 license。

## 什么样的项目适合先碰、什么时候不要碰

**适合先上的场景**：

- 已经在用 Signavio 做过流程建模、用 LeanIX 做过应用画像的客户。Joule Studio 真正能调动的资产正是这两份。没有这两份，意图驱动那一段会退化成普通代码生成。
- 出海企业的全球总部 IT 团队，要给散落在多个区域的子公司搭一套统一 Agent 治理，Agent Hub 这条是现成的中枢。
- 跨国制造、全球供应链的 SI 合作伙伴在做的标准化扩展项目——CAP + Fiori + n8n 这一套生成出来的代码可以直接进 Git，不会沦为"低代码黑盒"。

**暂时不建议碰的场景**：

- 数据主体在中国境内、且没有跨境业务的纯内贸项目。Joule Studio Runtime 当前没有中国境内可用区，托管运行时这条路基本走不通。
- 业务流程没有沉淀过模型、IT 资产没有画像的客户。先把 Signavio / LeanIX 该补的补齐，再谈 Agent。
- 强监管行业（金融、医疗）的核心交易 Agent。Agent Hub 的审计能力足够做事后复盘，但具体监管要求需要一案一议，目前还看不到行业级合规模板。

总结一句：Joule Studio 真正想解决的不是"写代码慢"，是"从业务问题走到能跑的 Agent"那段管线时间。这段时间值不值得 SAP 这么大手笔重做一套，要看 SAP Knowledge Graph、Domain Models 这些资产能否真正喂出有业务感的代码——架构画清楚了，剩下的要工程团队来跑。Sapphire 之后两个季度，是观察这条路落地真实度的窗口期。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/joule-studio-from-business-intent-to-enterprise-ready-ai-agents/ba-p/14395010
