---
title: "SAP 把门关上的那一刻，ServiceNow 把门拆了"
date: 2026-05-27T16:05:00+08:00
draft: false
tags: ["SAP CX", "ServiceNow", "AI Agent", "MCP", "Joule", "AI 治理"]
description: "Knowledge 2026 上 ServiceNow 把 MCP 通道大门拆开。同一个协议，SAP 用来收门票，ServiceNow 用来当过道。出海企业的 CX 选型要重新算账。"
source_url: "https://erp.today/servicenow-ai-security-governance-knowledge-2026/"
---

开放还是封闭，是企业软件吵了二十年的老话题。从 SOA 到 SaaS，从 API 经济到中台，每一轮新技术起来，都会重新审一遍这道题。AI Agent 这一轮本来也只是历史的又一次重演——直到这个月底，ServiceNow 在 Knowledge 2026 上把答案换了一种讲法。

**这次不一样的地方在于：同一个协议，SAP 用来收门票，ServiceNow 用来当过道。** 两家用的都是 MCP（Model Context Protocol），但路线完全分叉。

5 月 27 日，ERP Today 报道了 ServiceNow 在 Knowledge 2026 上的几个大动作。三件事并在一起看，画面就出来了——Action Fabric 把 MCP Server 直接 GA 了，对外开放给 Claude、Copilot、客户自研的 Agent；AI Control Tower 从一个监控仪表盘升级成执行级的治理产品；再加上把 Armis 和 Veza 整合进来的 Autonomous Security & Risk。一句话：**ServiceNow 不再把自己当工作流平台，它要做企业 AI 的"控制层"。**

![ServiceNow AI Control Tower](https://erp.today/wp-content/uploads/2026/05/k26-post-ai-control-tower-xl.png)

这是一记很有针对性的出招。两个月前 SAP 在 Sapphire 推出 Joule Studio、Agent Hub、Business AI Platform，路数大致是：把 Agent 的"户口本"做在 SAP 这边，所有外部 Agent 想进来调 SAP 数据和流程，得先到这个治理层登记。Forrester 当时已经写过一篇分析，说这是 SAP 在"试图把自己变成企业 AI 的看门人"。

**ServiceNow 这次的回应非常直接：我也做治理，但门是开的。**

Action Fabric 这个产品名取得很有意思。它没有叫 Agent Hub、Agent Studio、Agent Manager 之类的，而是用了"Fabric"——结构、织物、底层走线。Anthropic 的 Boris Cherny 在发布会上说了一句话很到位："知道该做什么和把它做出来之间的鸿沟，是生产力消失的地方。"Action Fabric 干的就是把 ServiceNow 已经积累十几年的工作流、审批、SLA、Catalog Request——这些带着权限、上下文和审计痕迹的"已治理动作"——变成 MCP 协议下任何 Agent 可以调用的能力。

这一招的精准之处在于：它没去抢"Agent 大脑"那一层。Claude、Copilot 谁来当大脑都行，ServiceNow 只负责让大脑下达的指令在企业里"算数"。**它认输了一步，赢回了更值钱的位置。**

**对比一下 SAP 这边的姿势。** SAP 5 月在 Sapphire 上说得很清楚：自家数据和 Agent 必须经过 Joule Studio、Agent Hub 治理；外部 Agent 想调 SAP 业务流程，要按调用次数算钱，叫 "AI Units"。techzine 当时一篇报道的标题就是《SAP blocks external AI agents, Salesforce and ServiceNow don't》，把对比点出来了。

SAP 选这条路有它自己的逻辑。Sapphire keynote 上 Klein 反复强调 "operational context"——AI 必须在企业流程里跑才有价值，而 SAP 是这个流程的所有者。从产品哲学上讲没毛病。问题是，**当 ServiceNow 把同一个 MCP 协议的另一头打开，故事的张力就出现了：客户的 Agent 大脑可能不在 SAP 里，但 ServiceNow 让它到达了执行层。**

Rolls-Royce 的数据是这场对比里很关键的一组事实。从 2025 年 8 月开始用 Now Assist 跑 helpdesk，到 Knowledge 2026 时已经做到了 54% 的偏转率，节省了 5,000 小时人工时间，全年减少了 38,000 个工单，工单解决时间下降 34%。这是真实生产数据，不是 demo。

但 Rolls-Royce 全球业务服务负责人 Phil Priest 说了一句更值得 SAP CX 项目方留意的话："我们意识到，把 AI 助手从 IT 扩展到其他业务部门时，几乎要把所有的知识库文章重写一遍，让它们 AI-ready。" **Agent 跑得动跑不动，瓶颈不在 Agent，在底下的数据和流程治理质量。** 这条结论对 Service Cloud V2、Sales Cloud V2 同样成立——SAP 把 Joule 装进 V2 之后，最难的不是 Joule 配置，是让 V2 的知识库、case 路由规则、客户主数据真正干净。

**这局面对中国出海企业的 CX 决策者意味着什么？**

第一件事，**"全栈 SAP" 这个选型默认值要重新算账了。** 过去做跨境零售、出海制造、海外分销网络的 IT 负责人，往往倾向于一套 SAP——Commerce Cloud + Sales Cloud + Service Cloud + Engagement Cloud 走全家桶。理由是数据不出 SAP、Agent 治理一站式。但 ServiceNow 已经把对外开放的 MCP 通道铺好，意味着如果客户的工单管理和资产管理本来就跑在 ServiceNow 上，那 Service Cloud V2 + Joule 的方案要面对一个新对手——客户可以选择让 Claude 直接经 Action Fabric 调 ServiceNow，绕过 SAP 这边的 Agent 治理层。

第二件事，**Agent 治理这门生意，未来一定不止一家。** Sapphire 之后我看到很多客户在问"是不是上 Joule Agent Hub 就够了"，答案现在更明确了：不够。一个出海企业的 IT 栈里同时跑着 SAP、ServiceNow、Workday、Snowflake，每家都在做自己的治理层。SAP CX 项目的架构师在选型时，需要把"Agent 跨平台编排"这件事放进决策清单——不是 SAP 做不做得到，而是客户会不会接受 SAP 一家说了算。

第三件事是给做 SAP 生态的合作伙伴的。**ServiceNow 已经把 SAP 列进 AI Control Tower 的 30 个集成对象之一**——这意味着客户用 ServiceNow AI Control Tower 同样可以治理跑在 SAP 里的 Agent。SI 在做提案时，再坚持"治理层必须是 SAP 这一家"已经站不住脚了。更现实的角度是：让 Joule Agent Hub 和 ServiceNow AI Control Tower 各自做自己擅长的事——Joule 治理 SAP 内的 Sales、Service、Commerce、Engagement Cloud Agent，ServiceNow 治理跨平台的 Agent 调度。

**这个方向我看好吗？说实话，半看好半担心。** 看好的是，企业 AI Agent 这盘棋如果真要规模化，"开放治理"一定比"封闭治理"赢面大。MCP 协议被两家都接受，就已经说明赛道方向不会变。担心的是，SAP CX 这边过去一年讲 Joule、讲 Agent Hub、讲 Business AI Platform 时，故事还是绕着"SAP 自己的栈"在讲——这套故事在客户实际的 IT 现实里，越来越接近"理想模型"。

Klein 那篇 5 月 19 日发在 Fortune 上的专栏标题叫 "The AI Race Is Being Fought in the Wrong Place"，他说战场不在模型，在企业上下文。说得对。但 ServiceNow 已经替他把下一句话补上了：企业上下文也不在 SAP 一家。

Knowledge 2026 这个时间点很微妙。ServiceNow Q1 2026 收入 37.7 亿美元、增长 22%，把 2026 全年订阅收入指引上调到 157.4 亿到 157.8 亿美元区间。这不是一个等着 SAP 来救场的玩家。SAP CX 在出海客户那里如果还想保住"AI 时代第一选择"的位置，**下一步要回答的就不是"Joule 能干什么"，而是"在客户已经选了 ServiceNow 当 IT 调度层的情况下，SAP CX 还能给我什么"。**

这是个不那么愉快的问题，但躲不开。

参考来源：https://erp.today/servicenow-ai-security-governance-knowledge-2026/
