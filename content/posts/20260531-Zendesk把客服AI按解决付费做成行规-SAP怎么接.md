---
title: "Zendesk把客服AI按解决付费做成行规，SAP怎么接"
date: 2026-05-31T19:15:00+08:00
draft: false
tags: ["SAP CX", "Service Cloud", "Zendesk", "Agentic AI", "Joule"]
description: "Zendesk Relate 2026把outcome pricing做成行业默认，AI ARR同比涨130%。专业化对水平化的挑战正面摆上桌，SAP在Sapphire喊出的200个Agent要面对一个新问题。"
source_url: "https://customerthink.com/zendesks-specialist-bet-is-the-right-one-and-heres-what-would-make-it-a-moat/"
---

Zendesk 在五月底的 Relate 2026 大会上把"按解决付费"喊成了行业新规，CRM 老兵 Thomas Wieberneit 写了篇分析，结论很扎心——客服 AI 的赛道出现了真正的分歧，SAP、Salesforce、ServiceNow 这些走水平化平台的玩家，要面对一个只做服务、却跑得比谁都快的对手。

## 同一个市场，两种打法

Zendesk 这次的产品发布，核心是一个叫 Resolution Platform 的编排层，加上一个持续学习的 Resolution Learning Loop。Agent Builder 让客户用拖拽方式造 Agent，Copilot 拆成 Agent、Admin、Knowledge、Analyst 四个角色，语音 AI 支持 60 多种语言并能在对话中切换，知识图谱直接接入 SharePoint、Google Drive、Notion、Guru、Contentful 和 Document360。

这一长串里真正有杀伤力的只有两条。

第一条是 outcome-based pricing——只有第二个 AI 模型验证"问题真的解决了"，Zendesk 才收钱。Forrester 喊了一年要厂商往这个方向走，Zendesk 是第一个把它做成默认商业模式的。

第二条是专业化压过通用化的赌注。Zendesk 的论点是：积累了 19 年的服务数据、几十亿条交互记录、加上一个有自己观点的服务技术栈，比那些拿通用 LLM 跑水平化平台的玩家强。

数据上他们也站得住。AI ARR 同比涨 130%，8 万家客户里有 2 万家在用 AI，2025 年从竞品手里抢了 1500 多笔单子。Salesforce 自己的 State of Service 报告也承认，服务领域用 Agentic AI 的比例一年内从 39% 跳到 66%——市场在真正换底，不只是换名字。

## Zendesk 的取舍，恰恰是 SAP 不能取的

Zendesk 首席产品官在被问到编排定位时说得很干脆：我们只编排服务交互，不假装编排销售、营销或者整个公司。这是个诚实的回答，也是个值得琢磨的取舍。

SAP 在 Sapphire 走的是另一条路。Joule 被定位成中央编排器，AI Agent Hub 不仅装自家 Agent，连 Salesforce、ServiceNow 的 Agent 也要进户口本，1 亿欧元砸进去做生态。逻辑是"你不可能治理你看不见的东西"——所以平台必须把所有 Agent 都纳入视野。

这个逻辑没错。但它的另一面是：当 Zendesk 这种垂直玩家在服务这一格深挖到 19 年数据、做出按解决付费的商业模式时，水平化平台的服务能力只能跟得上、很难超过。SAP CX 在五月放出来的 50 多个 Agent，分布在销售、营销、服务、商务、忠诚度、订单管理多个产品线里——每个领域都不弱，但要在某一格做到 Zendesk 那种深度，需要的不是再发 Agent，而是垂直数据的累积。

这才是 Service Cloud V2 真正的压力点。把 Joule 装进客服后台、把 Agent Inbox 接进路由、把分析层换成事件驱动——这些都做了。但 Zendesk 拿来比的不是这些功能，而是"我只做这一件事，做了 19 年"。

## 客户面板抖出来的现实

Wieberneit 这篇文章最值得读的是客户环节。Direct Supply 的 Stacy Niven 说她们花了好几年重建数据底座，订单一半要人工干预，流程跑在 Excel 里，自己开发的 Chatbot 因为产品数据太烂被回滚。Lyra Health 的 Sam Bellach 一句话定性：AI 的好坏，取决于喂它的数据。

Salesforce 的研究也佐证：59%–72% 的服务从业者把"数据准备度"列为 AI 落地的首要障碍。这不是 Zendesk 的问题，是整个客服 AI 赛道共同的天花板。

第二个有意思的发现是，AI 时代客户要的不是更少的人工，而是更多人工——Levi's 的 Jessica Hsieh 说 61% 的 CX 主管反馈人工话务量在涨。Bumble 的 Elymae Cedeño 在做信任与安全产品时坚持人是地基。Levi's 的做法是 AI 处理"我的货在哪"这种事务性问题，把人工留给判断和共情。

这一点和 SAP 在 Sapphire 上喊的 Autonomous CX 形成了一种微妙的对照。当 Klein 反复说"自治企业"时，台下客户的实际诉求是"先把 AI 做好的部分做扎实、把人工留给关键时刻"。两边都不算错，但站位不一样。

## 真正的关键变量，藏在那个 learning loop 里

Wieberneit 在分析师沟通里追问了一个尖锐的问题：Resolution Learning Loop 怎么保证学到正确方向？如果系统按"什么算已解决"的当前规则去优化，会不会逐渐漂向"容易被验证的解决"，而把难处理的边缘案例丢掉？

Zendesk 的回答覆盖了基础——多 LLM 评分、客户申诉机制、对"已解决"的判定偏保守。这是个不错的起点，但要变成护城河，需要的是公开的治理姿态：漂移检测方法、规则版本管理、人工复审节奏、对抗性测试用例、审计可见度。这套东西如果发出来，Zendesk 在客服赛道的领先就不只是一两步。

这套对治理透明度的要求，对 SAP 同样适用。Sapphire 上 SAP 把 Agent Hub、Agent Governance 拎成单独产品线——方向是对的，但公开文档的颗粒度还没到 Zendesk 这种"公开 governance 姿态"的程度。EU AI Act 的合规压力快到了，谁先把治理做成可见的产品，谁就能在 CIO 的选型表上多一格分。

## 写给做出海的客户

这个对比对中国出海企业的 CX 决策者意味着什么？

如果你的客服场景集中在某一个垂直领域、需要快速跑出 ROI，Zendesk 的"按解决付费 + 专业化"打法吸引力不容忽视，尤其是出海到欧美市场、有合规和定价透明度压力的卖家。但有一条要算清楚：outcome pricing 的"什么算已解决"是按 Zendesk 的规则定义的，灵活度还在演进，Sam Bellach 自己作为客户咨询委员会成员都在公开吐槽"中合同期间 agent-seat 和 resolution 之间的转换还做不到"。

如果你是跨国制造、跨境电商或者全球化品牌，需要的是把销售、营销、服务、订单、忠诚度连成一张网，那 SAP 的水平化路线在数据底座（BDC）和跨流程编排上仍然是更现实的选择。SAP CX 五朵云加上 CLM、OMS、ESM 这些延伸产品，覆盖面是 Zendesk 比不了的。

真正的问题不在二选一，而在 SAP 的客户能不能从这次 Zendesk 的发布里学到东西——按结果定价的合同条款、对治理透明度的要求、把数据准备当成 AI 落地前置项目。这些做法不专属于 Zendesk，是这一轮客服 AI 重构里所有玩家都得回答的题。

水平化平台的护城河在广度，垂直玩家的护城河在深度。SAP 这次 Sapphire 喊出 200 个 Agent 时，把广度的故事讲到了顶。接下来要补的，是每一个具体行业、具体场景里的深度叙事——不只是"我们也能做"，而是"我们做了多久、积累了多少数据、有什么是别人复制不了的"。

参考来源：https://customerthink.com/zendesks-specialist-bet-is-the-right-one-and-heres-what-would-make-it-a-moat/
