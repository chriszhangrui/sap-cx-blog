---
title: "Joule 接 Claude 这事，真正的产品是 MCP"
date: 2026-05-25T01:15:00+08:00
draft: false
tags: ["SAP CX", "SAP Joule", "Anthropic", "MCP", "Business AI Platform"]
description: "SAP 选 Claude 不是模型层面的事，它把自己升格成了企业 AI 与业务系统对话的协议层。Joule 不是真正的产品，MCP 才是。"
source_url: "https://erp.today/sap-anthropic-claude-joule-mcp/"
---

很多人看到 SAP 和 Anthropic 的合作，第一反应是**"哦，SAP 又选了一个大模型"**。就跟当年选 Azure OpenAI、Google Gemini 一个意思——用谁的模型只是个偏好题，今天选 Claude，明天换 GPT 也无所谓。

但 5 月 20 日 ERP Today 的那篇分析稿把这件事的真相挖出来了。**SAP 这次不是在"接一个模型"，而是把整条业务流的入口交给了 Anthropic。**这两件事的差别，落到客户身上是几千万欧元和上亿欧元的差别。

CTO Philipp Herzig 在 SAP 官博里写得很直白。Claude 通过 Model Context Protocol（MCP）这条标准协议，被同时接进了 SAP Business AI Platform 和 Joule。这话听起来像两件事，其实是同一件——Claude 既是底层基础模型，又是 Joule 编排层的延伸。换个说法：Joule 是那个站在前台听用户说话的 AI 助理，Claude 是站在它身后做推理的脑子。

![SAP Sapphire 2026 Joule Sign](https://erp.today/wp-content/uploads/2026/05/SAP-Sapphire-2026-Joule-Sign-1024x768.jpeg)

**关键的事情不在 Claude，关键的事情在 MCP。**

以前的玩法你应该熟。要让一个 AI 应用能查 S/4HANA 的客户主数据、能从 SuccessFactors 读员工档案、能查 Ariba 里的供应商目录、还能去外部供应商门户拉一笔订单状态，得给每一个系统单独写连接器、单独维护 token、单独处理 schema 变更。一个销售对话场景背后可能挂着五六个集成，全是项目实施时焊死的硬代码。客户系统升一次级，半个集成层要重写。

MCP 把这件事拍平了。SAP 在 TechEd 2025 已经预告过，HANA Cloud 在 2026 年初正式加上了完整的 MCP 支持，意味着 Joule 的 agent 直接走 MCP 协议就能访问数据库引擎。SAP Integration Suite 里所有的 API 和 flow，包括连到第三方系统的，都能一键转换成 MCP 工具。**说人话，原来你给客户做集成项目的那一堆胶水代码，未来可能不用写了。**Claude 那边只要会说 MCP，它就能调 SAP 任何角落的能力。

这背后其实有 SAP 的私心。它正在悄悄把自己变成所有企业 AI agent 跟业务系统对话的"网关"。Anthropic 的 Claude 进来是因为 MCP 标准开放，那将来 OpenAI、Google、阿里、字节的模型也能进来——但它们要进来，必须走 SAP 的协议、用 SAP 的语义。这盘棋下得不止 Anthropic 一家。

具体到 Joule 怎么用 Claude，Herzig 在博客里讲了个三层处理机制，挺有意思。Joule 收到一个用户请求，先去查三件事：第一是 Scenario Catalog，里面存着所有 SAP 云应用的场景、功能、技能元数据；第二是 Knowledge Catalog，包含 SAP 自家知识和客户自己的知识库；第三是用户上下文和历史，包括基于角色的权限和聊天记录。这三个东西揉成一个增强过的 query，再丢给底下的大模型——其中就包括 Claude。

![SAP Sapphire 2026 Opening Keynote](https://erp.today/wp-content/uploads/2026/05/SAP-Sapphire-2026-Opening-Keynote-1024x768.jpeg)

这个设计很聪明。它把"AI 思考"和"业务上下文"明确分了层。Claude 负责推理和规划，但不直接读企业数据；Joule 负责把企业的真实信息打包好递给 Claude；MCP 负责让 Claude 调用工具时不用关心后端是哪家的系统。三层各司其职，谁掉链子谁背锅。

**对 SAP CX 这条线的客户来说，这件事的味道不一样。**

Sapphire 上 SAP 已经发了 50 多个 CX 相关的 AI agent，覆盖 Sales Cloud V2、Service Cloud V2、Engagement Cloud 三大产品线。这些 agent 后面跑的是什么模型？以前 SAP 用的是自家的 Generative AI Hub 接 Azure OpenAI，现在 Claude 进来后，那些 agent 的推理引擎大概率会切到 Claude——尤其在欧盟客户那边，Anthropic 的合规故事比 OpenAI 讲得更稳。一个出海到欧洲的中国零售品牌，将来用 Service Cloud V2 处理客诉，agent 后台跑的可能就是 Claude 模型加 MCP 协议拉通的 SAP 客户数据。

Anthropic 和 SAP 接下来要联合开发的行业 agent，名单也透露了一些信息——公共部门、医疗、教育、生命科学、公用事业。乍一看跟 CX 没关系，但生命科学和公用事业的客户互动场景里，Service Cloud 加 Engagement Cloud 是常用组合。这等于在告诉欧洲监管最严的几个行业：你们用 SAP+Claude 这套，是经过定向调优的，不是通用模型瞎凑。

说回风险。ERP Today 这篇文章里有三个非常实在的问题，做过中大型 SAP 项目的人一看就懂。

**第一个，MCP 的维护责任谁担。**当你的采购 agent 要同时跨 S/4HANA、Ariba、SuccessFactors 和外部供应商门户这四个系统跑工作流，任何一段 MCP 连接断了，不只是慢——是整个采购决策直接停摆。这件事在合同里、在实施 SOW 里必须写清楚：谁负责测试、谁负责监控、谁在出问题的时候顶上去。SI 合作伙伴接这种项目，要把这一条单独拎出来谈费用，不能跟普通集成混着算。

**第二个，更深的耦合就是更深的绑定。**SAP 把这次合作包装成"开放生态给客户更多选择"，但 Claude 越深入到 Joule 编排层，对 Anthropic 路线图、定价、可用性的依赖就越重。Anthropic 哪天涨价、哪天某个能力下线、哪天被监管调查暂停在某地区服务——你的企业 agent 就会跟着震一下。采购和架构团队评估的时候，要把这个风险画出来，不要等到部署完了才想起来。

**第三个最关键，"AI 推荐"和"AI 行动"之间隔着一道审计鸿沟。**基于角色的权限只决定 agent 能访问什么数据，不决定人在什么时候介入。在受监管的行业，审计链和人为问责权是不能松动的。评估任何带 agentic 能力的 AI 平台时，要把三个问题问到底：agent 的推理在哪里发生？它能访问什么数据？它执行某个动作前，你能不能在生产环境里检查、能不能覆盖它的决定？

这三个问题，我看到的项目里很少有人在合同阶段就问清楚。多数是上线后出了问题再回头补流程，那时候已经被绑死了。

回到开头那个反直觉的判断。SAP 选 Claude 这件事，表面上看是模型层面的事，实质上是 SAP 把自己升格成了"企业 AI 与业务系统对话的协议层"。模型今天是 Claude、明天可以是别人，但这个协议层一旦建起来，就是 SAP 这十年最重要的一道护城河。Joule 不是它真正的产品，MCP 才是。Anthropic 这次不过是第一个押进来的玩家。

至于 Claude 进 Joule 这件事对中国出海企业意味着什么——如果你做的是欧洲、北美、生命科学、公共事业相关的市场，且企业里已经有 SAP 在跑，那你接下来一两年要做的事就是把现有 CX 流程拆成"哪些适合让 agent 处理"、"哪些必须保留人工"两个清单，然后挑两到三个高频低风险的场景试点 Joule agent。先别急着把 50 个 agent 都打开，先把 3 个跑稳。

参考来源：https://erp.today/sap-anthropic-claude-joule-mcp/
