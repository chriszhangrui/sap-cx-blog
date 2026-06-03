---
title: "Agentic Joule：ABAP AI 从助手做成了生态"
date: 2026-06-03T22:30:00+08:00
draft: false
tags: ["SAP CX", "AI", "Joule", "ABAP", "MCP", "技术深度"]
description: "ABAP MCP Server Q2 GA、ADT 进 VS Code、CCM 三个 Agent 接管自定义代码迁移——这次 SAP 没再造一套封闭体系，把开放权交回给客户。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/agentic-joule-for-developer-it-s-all-about-your-choice/ba-p/14407719"
---

过去一年里，ABAP AI 多数时候还停留在"聪明的助手"——能解释一段老代码，能帮你写测试，能把 finding 列得更清楚一点。有用，但本质上是辅助。Sapphire 2026 上 SAP 给 Joule for Developer 加的这一笔，方向变了：从一组 skill 开始，往一个 agent-led 的开发模式上挪。

变化最关键的几个标记，落在三件事上——ABAP MCP Server 即将 GA、ABAP Development Tools 进入 VS Code、自定义代码迁移（CCM）拆出三个 Agent。把这三件事拼起来，能看清这次架构选择背后真正放弃了什么、保留了什么。

## 这次改动到底在解决什么

在过去的封闭模型里，ABAP 开发者要用 AI 能力，路径是绑定的：装哪个 IDE、用哪个助手、走哪个 orchestrator，往往跟着 SAP 自己的产品组合一起决定。这种打包的好处是省事，代价是一旦客户已经买了别的助手栈、已经在 Microsoft Copilot 或 Anthropic 的链路上跑业务，就只能再开一条平行通道。

这次 SAP 把架构拆开了。SimonaM 在 SAP Community 那篇博客里把这次改动概括成一句话——"It's all about YOUR CHOICE"。这个 choice 不是营销话术，是架构层面的选择：IDE 选 Eclipse 还是 VS Code、AI 助手选哪家、orchestrator 选哪家、ABAP 落在 Public Cloud / Private Cloud / BTP / On-Prem 哪一段、商业上怎么按用量付费。

![Agentic Joule for Developers 五层架构](https://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLWPuAafJhkhvIYnN35YNiaZ8DgjqQDkhvyUqNCrJy3NlbeIp3Vib8JrLJlZm1SP1ExX3Tv09K2l3BEKkvZgXHV5ibKOsyj9cy4OvQ/0?from=appmsg)

## ① ABAP MCP Server：开放的真正落点

整个故事里最值得圈起来的信号，是 ABAP MCP Server 这一季度（Q2 2026）就要 GA。MCP（Model Context Protocol，模型上下文协议）这层东西，本质是把 ABAP 的能力——读代码、改代码、查依赖、跑分析——以一种 AI 助手能直接调用的形式暴露出来。

这一步比"我们也支持 MCP"听起来要重要得多。它把 SAP 在 ABAP 里的领域知识（代码语义、Clean Core 规则、findings 模型）和 AI 助手做了职责切分：

- SAP 提供 ABAP 智能本身，包括语义理解、规则、迁移知识；
- MCP 层做桥，把这些能力转成 AI 助手能消费的工具调用；
- 客户决定上面接谁——Microsoft、Amazon、Google、IBM、Anthropic 都在 SAP 列出的 partner 名单里。

对架构而言，这个分层比把所有事情塞进一个封闭 chatbot 要稳得多。Chatbot 是产品形态，MCP Server 是接口契约——前者会被新一代交互形态淘汰，后者只要协议没换就能复用。

## ② ADT 进 VS Code：选择权要落到日常工具

给 ABAP 开发者把 IDE 选择权"嘴上承诺"是一回事，把它真做出来是另一回事。ABAP Development Tools for Visual Studio Code 这次的 GA，就是把口头承诺变成可用产品。第一个版本聚焦 ABAP Cloud，按 SAP 给出的 roadmap，年底前 Eclipse 上的 ABAP Classic 支持也会迁到 VS Code。

这件事的价值在分工上：VS Code 当创作工作台，MCP 层是动作引擎，partner 的 orchestrator 把"想法 → 完成的 Fiori 应用"这条路径串起来。ABAP 团队不必再把自己锁在 Eclipse 一种节奏里，也不用为了用上 AI 而被迫换工具栈。

> 对在华外企或出海企业的 IT 团队来说，这一步意义更具体——很多本地团队的标准开发栈早就是 VS Code，过去对 ABAP 这条线只能"另开一台 Eclipse"。现在两条线能合到一台机器、一个工作流里。

## ③ 跨 landscape 覆盖：从 BTP 一路下到 On-Prem 2021

原文里有一句话被低估了：从 6 月起 Joule for Developer 会按客户实际所在的位置覆盖——SAP S/4HANA Cloud Public Edition、BTP ABAP Environment、S/4HANA Cloud Private Edition 2021 起、以及满足条件的 RISE 客户的 On-Premise 系统（同样从 2021 起）。

"从 2021 起"这件事很关键。它意味着大量已经升到 2021/2022/2023 但暂时没有打算上 Public Cloud 的客户——尤其是 RISE with SAP 路径上的客户——不需要等下一次 landscape 决策完成，就能开始用 agentic AI。这是迁移项目里最常见的场景：客户在 Private Cloud 上，今年没预算做版本跳跃，但明年要开始做 Custom Code Migration。

## ④ CCM 三剑客：把代码迁移做成执行流

Custom Code Migration 这件事，过去最大的痛点不是不知道哪些代码要改，而是知道了之后改不动——findings 列了一万条，谁去做、按什么顺序做、怎么验证。Joule for Developer 这次给出的回答是把它拆成上下游：

- **上游**用 Cloud ALM 做项目分析、backlog 结构化、风险与工作量定位；
- **下游**三个执行 Agent 接管真正的代码动作。

三个 Agent 的角色是错开的：

- **Mass S/4 Custom Code Conversion Agent**（Q2 2026）——解决规模问题，把大量 finding 从"分析报告"推到"可执行的修复"；
- **Clean Core Adoption Agent**（Q3-Q4 2026）——从技术修复升到 Clean Core 适配，给推荐和自动修复；
- **Redesign & Modernization Agent**（2026 末–2027 初）——最复杂，做 Dynpro 到 RAP 的应用现代化重构。

这三个 Agent 的顺序不是随便排的：从"批量改"到"按 Clean Core 改"到"重新设计"，恰好对应 RISE 项目里大部分客户的真实分阶段路线。把它做成 Agent 而不是工具集，原因也是这个——前一个 Agent 的输出就是下一个 Agent 的输入。

## ⑤ 商业模型：免费推广 9 月 30 日截止

有一个不在架构图里但要单独提的事——商业层。SAP 这次配套上了消费式定价（consumption-based pricing），明确区分客户和合作伙伴的不同路径。免费促销将在 2026 年 9 月 30 日结束。

但 SAP 同时强调：通过 ADT（Eclipse 或 VS Code）连接 ABAP MCP Server 和本地 ABAP MCP Tools 是不额外收费的——只对真正调用 AI 能力的部分计费。这种结构对 PoC 阶段的项目反而友好，可以先把链路搭起来再决定哪些 AI 调用放出去。

```
// 客户视角的成本结构（基于原文整理）
ADT for Eclipse / VS Code .................. 免费
ABAP MCP Server (本地连接) ................. 免费
本地 ABAP MCP Tools ....................... 免费
──────────────────────────────────────
AI 能力调用（partner 助手 + orchestrator） ... 按消费计费
CCM Agent 执行（迁移、Clean Core、Redesign） . 按消费计费
──────────────────────────────────────
免费推广截止：2026/09/30
```

## 它放弃了什么、保留了什么

这次设计放弃的，是 SAP 一直试图占据的"AI 助手主入口"位置——不再非要让客户用 SAP 自家的 chatbot 形态。保留的，是 SAP 真正稀缺的资产：ABAP 语义理解、Clean Core 规则、迁移知识、与 S/4 各版本之间的对齐能力。

对手层面，这个选择是合理的。Microsoft、Anthropic 这些公司在通用 AI 助手这块投入和迭代速度，SAP 短时间内追不上；但 ABAP 这层别人短时间内也做不出来。把开放权让出去，反而能让 SAP 的核心资产被更多 AI 工作流复用。

## 什么样的项目适合上、什么时候先别碰

- **适合的**：已经在做 RISE 迁移、有较大规模自定义代码（>500 个对象）、IT 团队已有标准 AI 助手栈（Copilot/Claude/Bedrock）、希望开发链路统一在 VS Code 里的出海企业或在华外企。
- **暂时别碰的**：还停留在 ECC、没有清晰 RISE 路线图、也没决定主用哪家 AI 助手的项目。MCP Server 不是给"AI 还没想清楚"的客户兜底用的，前提是上层 partner 链路要先选定。
- **需要警惕的踩坑点**：免费促销 9 月 30 日截止，但 ABAP MCP Server Q2 GA 之后到 9 月这段窗口期，恰好是验证 partner orchestrator 与 ABAP 工具调用兼容性的最佳时间——错过这段，后面试错就要按消费计费了。

对架构师来说，这次改动最值得记下的判断是：当一个平台开始把开放层（MCP）做得比自家入口（chatbot）更厚时，往往意味着它对核心资产的边界想清楚了。这种姿态比堆 feature 更值得留意。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/agentic-joule-for-developer-it-s-all-about-your-choice/ba-p/14407719
