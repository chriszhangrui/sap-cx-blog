---
title: "Forrester把话挑明了：SAP这把封锁牌，是从弱势位置打的"
date: 2026-05-21T13:10:00+08:00
draft: false
tags: ["SAP", "SAP API Policy", "Forrester", "Joule", "Agentic AI", "Enterprise AI"]
description: "三周前像废纸的一份API政策，Sapphire之后长出了牙。Forrester给CIO的判断是：SAP从AI弱势位置搞封闭，6月9日大限+2027价格悬崖，这是用未来十年的客户信任换今天能锁住的预算。"
source_url: "https://www.forrester.com/blogs/sap-is-attempting-to-become-the-gatekeeper-of-enterprise-ai-cios-should-push-back/"
---

> — FORRESTER · MAY 20, 2026 —

三周前 SAP 发了一份 API 政策，叫 v.4.2026a，业内当时翻了一下，普遍觉得是法律部门写的纸面规定，没什么可执行的牙齿。Sapphire 2026 开完之后，Forrester 几位首席分析师联名发了一篇博客，把话挑明了——这份政策不是纸面文件，它后面挂着一整条产品线。

距离 6 月 9 日强制执行还有三周。今天先把这件事讲清楚。

## 先说政策本身在禁什么

SAP API Policy 把 SAP 指定路径之外的三类行为划成了红线：

- 第三方 AI Agent 调 SAP API，去做规划、选择、执行任何动作；
- 把 SAP 数据大规模导出到非 SAP 环境；
- 通过 proxy、网关、自定义代码或者身份冒充绕开上面两条（政策第 3 节专门点名了）。

6 月 9 日开始执行，靠的是一个安全补丁，技术性地把不合规的 ODP via RFC 调用堵死。背后的支撑文档是 SAP Note 3255746（4 月 21 日更新到第 11 版），自查工具是 SAP Note 3439624。

用大白话说就是：你要是用 Azure Data Factory 的 SAP ODP 连接器从 S/4HANA 抽财务数据进 Microsoft Fabric，6 月 9 日那一天这条管子就断了。

## 为什么这次有牙

这份政策第一次发的时候，圈子里都在等 SAP 露馅。因为它有四个明显的缺口——指定路径太模糊、第三方 Agent 怎么连没说清、收费层根本不存在、SAP 自己也没有像样的 AI 产品撑得起这个限制。

Sapphire 之后，四个缺口全焊死了。

Joule Studio 2.0 GA 了；Joule Work 跑在生产里；AI Agent Hub 第三季度 GA；MCP 网关第二季度上线；BDC 已经在卖了——CIO 现在可以拿着厂商的承诺对到具体日期。

最关键的一条：第三方 Agent 想碰 SAP 数据，必须经 Joule Agent 中转，走 A2A 协议（第四季度 GA）。Microsoft Copilot for Finance 想查 SAP 总账？得经 Joule。Salesforce Agentforce 想读 CX 客户主数据？得经 Joule。ServiceNow AI Agent 想触发 SAP 流程？还是得经 Joule。SAP 定义这个中介 Agent，管协议，给流量计费——十二个月之前这一层根本不存在。

> **数据以前流的是计算成本，6 月 9 日之后流的是计算成本 + SAP 计量层成本。** 这是 Forrester 给整件事下的一句结论。

## 2027 年的悬崖

这部分 SAP 自己没怎么讲。

Joule Studio 2.0 现在免费——免到 2026 年 12 月 31 日。Agent runtime 免费。A2A 互通免费且不限量。Forrester 直接说，这是 SAP 十年来商业上最激进的一步。

熟悉的味道。2017 年微软用同样的姿势推 Teams：先免费让你用上瘾，再一刀切收费。

2027 年那张账单具体长什么样，到现在没公开过。Agent Gateway 流量价格、A2A 消费定价、BDC 出口流量、Joule Studio 促销期结束之后的 SKU——四样东西都没数。多数客户的 2026 年 IT 预算根本没建模这块成本。

免费窗口，是 SAP 让你上高速的诱饵；2027 年那个收费站，是它知道你下不来。

## Forrester 为什么不买账

分析师团队的判断很直白：封闭架构在你赢的时候才好用。Apple 当年关 iOS，是站在产品领导力的高地上关的。SAP 现在关 API，是站在 AI 弱势的位置上关的。

证据藏在 SAP 自己的定价里。Joule Studio 2.0 免费、Agent runtime 免费、A2A 不限量。再看对比——Microsoft 365 Copilot 付费商业席位 2026 财年第三季度已经超过 2000 万，比上一季度的 1500 万涨了 33%。SAP 那 224 个 Agent + 51 个 Assistant，多数还在 GA、早期采用、preview 之间的混合状态。

Forrester 给出的历史参照，是 IE 浏览器和 Twitter API。两个都是用限制策略短期保住控制，长期把开发者和合作伙伴推向了开放替代品。

> SAP 完全可以靠 Joule、Knowledge Graph、BDC、Agent Gateway 的能力本身收钱，不用禁竞争对手。营收大概率会跟着能力来。**选了限制路径，等于是在用未来十年的客户信任，换今天能锁住的客户预算。**

2017 年那场间接访问争议，给买家留下了长期警觉——SAP 可能用客户没预算到的方式给"访问"定价。这次政策正在把同一种担忧重新唤起。担忧最后会以什么方式表现出来？续约谈判变慢、架构决策更多对冲、谈判桌上更难谈。

## 这事跟出海企业有什么关系

写到这里，要把镜头转一下。这条政策对中国出海企业的 IT 决策者，比对北美客户更刺手。

原因有三层。

第一层是数据架构。中国跨境电商和制造业里，相当一部分企业的玩法是 SAP S/4HANA 跑财务+订单，CRM 用 Salesforce、营销用各种本地或者第三方栈、客服一部分外包给 ServiceNow 或者更轻的工具。这种"SAP 当后端真相 + 多家前端 SaaS"的拼法，恰恰是这次政策最直接打击的对象。

第二层是 AI 落地节奏。中国出海品牌做欧美市场扩张时，这两年正好到了"上 Copilot/Agentforce/Gemini 试水生成式 AI"的阶段，相当一部分项目立项的前提是"它能直接读 SAP 数据"。6 月 9 日之后，这个前提要重新评估。

第三层是合同周期。中国客户的多年期 SaaS 合同里很少有 SAP carve-out 条款，因为以前没必要写。现在如果不补，2027 年那笔账要么自己吞，要么在续约时硬谈。

## Forrester 给 CIO 的五步

原文最后一段非常硬，几乎是写给董事会看的。挑出来逐条说说：

1. **这周就盘 ODP-RFC 依赖。** 所有从 SAP 流向非 SAP 的数据管道，都是 6 月 9 日的风险点。用 SAP Note 3439624 自查工具，列清单、排优先级、做迁移计划。
2. **冻结依赖 SAP 数据的多年期第三方 AI 合同。** 包括 Salesforce Agentforce、Microsoft Copilot for Finance、ServiceNow AI Agents、Workday Illuminate、Celonis。3 年以上承诺，没有 SAP 书面 carve-out 不签。
3. **升级到 SAP 高管层级。** 书面要四件事：现有集成的 grandfathering 条款、合理使用的量化阈值、2027 年商业条款、API 发布 SLA 承诺。给 14 天回复窗口。
4. **试点 Joule Studio，但锁住 2027 年的价格。** 免费窗口正好用来验证能力。所有 2026 年签的 SAP 续约，都加上促销期结束后的价格上限。
5. **上董事会。** 这件事比 IT 集成大。一个核心战略合作伙伴搞这种级别的供应商控制事件，影响 IT 预算和 AI 战略，董事会需要知道。

## 最后说一点

读完这篇博客，最让我印象深的不是 6 月 9 日那个时间点，也不是 2027 年的悬崖。是 Forrester 三位分析师在最后的那个判断——SAP 当然有理由这么做，封闭可以保住短期 ARR、可以让 Joule 不是免费午餐而是高速公路、可以让 BDC 真的成为 AI 的数据控制平面。

但选择限制而不是竞争，本质上是在赌客户没得换。SAP 全栈客户能到现在这个深度，是几十年的工艺积累；这种深度也意味着，一旦客户开始觉得续约里多了"不安"，他们的对冲不会立刻爆发，而是慢慢渗透到每一次架构决策里——下一个新业务上 Snowflake 还是 BDC？下一个 CRM 项目延伸 SAP CX 还是上 Salesforce？这些选择会变得比之前犹豫。

政策是计划。6 月 9 日是扳机。2027 年是账单到的那一天。

*SAP 这一手是赢面更大还是反噬更大，下个季度的续约谈判桌上会先给出答案。*

参考来源：https://www.forrester.com/blogs/sap-is-attempting-to-become-the-gatekeeper-of-enterprise-ai-cios-should-push-back/
