---
title: "Agent Inbox 上去之后，Service 的路由就变了"
date: 2026-05-23T10:00:00+08:00
draft: false
tags: ["SAP CX", "Service", "技术深度"]
description: "Agent Inbox 不是换皮，是把 Case、Task、Service Order 拉到一个上下文里。再叠 Case Management Assistant 和 S/4 资产联动，路由从规则匹配变成上下文驱动。"
source_url: "https://www.savictech.com/insights/sap-autonomous-cx-sales-service-cloud-v2-joule-agents-2026/"
---

做过 Service 项目的都遇到过这一幕：客服坐在 Outlook、CRM 工单、知识库门户、ERP 资产卡之间反复切窗。一通电话挂掉，光把这几个上下文凑齐就花了两分钟。SAP 在 Q1 2026 把 Service Cloud V2 的入口换了——Agent Inbox 上线，把 case、task、service order 拉到一个面板。

看上去是 UI 改造，但牵动的是路由层和数据层的重新分工。这一篇把架构拆开看。

![Service Cloud V2 Agent Inbox 架构](https://www.savictech.com/insights/sap-autonomous-cx-sales-service-cloud-v2-joule-agents-2026/)

## 一、Agent Inbox 不是新皮肤，是被抬到中台位置的入口

传统 Service Cloud 的工作面是按模块切的：case 队列一个 tab、task 一个 tab、外勤工单又一个。客服为了处理一通电话，得在三四个 tab 之间来回找上下文。这套布局有它的来历——早期 CRM 的对象模型就是这么独立切分的，每个对象有自己的列表视图。

Agent Inbox 的变化在于，它把 case、task、service order 三类对象的实例放到同一个实时面板。再叠两层视图：一是工作量看板（按优先级、按账龄、按 SLA 风险分布），二是个人队列的智能排序。SAP 公布的数据是单人日均 case 处理量上升，原因写得很直白——减少了 context-switching 的开销。

更关键的是，Inbox 不只是显示层。它在架构上被抬成了一个 orchestration 层——下游对接 Case Management Assistant、Service Management Assistant、Self-Service Assistant 三个 Joule 助理（计划 2026 年 11 月 GA），上游收编邮件、电话、自助表单、聊天等所有渠道。这个位置过去一直空着，要么是路由引擎独立部署，要么是 BPM 流程引擎硬接。Inbox 接管之后，路由这件事的定义就变了。

## 二、路由从"规则匹配"变成"上下文驱动"，这是核心架构判断

老式的 case 路由几乎都是规则引擎：按服务等级、按产品线、按地域、按客户分级，写一组 if-else 把 case 推到对应队列。这套机制的问题是，规则越多越难维护，而且只能用 case 上能查到的元数据做判断。客户上一次报修过什么、当前合同里那台设备的保修状态、知识库里有没有现成解决方案，规则引擎都看不到。

Case Management Assistant 给路由加了一个上下文聚合的前置步骤。在分派之前，它要做四件事：聚齐这个 case 相关的客户历史交互、检索知识库找候选解决方案、识别处置建议（含升级判据）、安排跟进。换句话说，case 进入队列时已经带着一份"作业前置清单"，agent 看到的不是一行 subject 加 priority，而是一份准备好的处置上下文。

配置层的概念也跟着变。SAP 的解决方案指南里把这套机制描述为 routing keys for entities——不再是单一的 case 字段触发某条规则，而是用 entity 维度的 key 去匹配技能、队列、负责人。这意味着一旦你的客户主数据、产品主数据、合同数据维护得不齐，路由引擎能用的信号就很少。这是落地时最容易踩的坑：上线前没人会承认数据脏，上线后路由结果一塌糊涂全怪 AI 不智能。

```
# 配置维度（解决方案指南）
Entity         Routing Key 字段          作用
─────────────────────────────────────────────
Case           Priority / Type / SLA     基础匹配
Account        Tier / Region / Owner     客户分层
Product        Line / Category           技能匹配
Installed Base 设备型号 / 保修状态       连接 S/4
Knowledge      Topic / Solution Hit      处置前置
```

## 三、知识库不是独立产品，是 Mashup Provider

Service Cloud V2 的知识库接入方式在 setup guide 里写得很清楚：Configure Knowledge Base Provider。它不强制你用 SAP 自家的知识库产品，而是定义一个 provider 接口，可以把 Confluence、ServiceNow KB、自建文档站，甚至 SharePoint 接进来。这套设计的好处是已有知识资产不用迁，坏处是检索质量取决于 provider 那一端的能力。

到了 Joule 时代，这个 provider 接口又承担了 RAG（检索增强生成）的作用。Case Management Assistant 给出处置建议的前提，是它能从知识库里检索到相关的解决方案。如果 provider 那头没有向量化、没有打 embedding，AI 处置建议的命中率会很难看。这不是 SAP 的问题，是知识管理本身的问题——只是 V2 把它放大了。

做出海项目的客户经常忽略一件事：知识库里多语种条目的检索一致性。中文资产、英文资产、当地语言资产如果没有做语义对齐，agent 在不同区域看到的处置建议会差一截。这不是开关问题，是数据治理问题。

## 四、与 S/4 的资产联动，决定 Service 能不能闭合到工单

纯客服场景到此打住——case 解决完关闭就行。但凡涉及实物维修、配件更换、技师上门，case 必须能向下连到 S/4HANA 的 Plant Maintenance 或资产管理模块。Service Management Assistant 这条线的作用就在这里：它管的不是 case 本身，是 service order——服务订单创建、排程、配件可用性、派工。

数据流走向是双向的。从 V2 这一侧看：客户来电 → 在 Inbox 里识别这台设备的 installed base 记录 → 拉 S/4 那边的保修状态、上次维护记录、备件库存 → 在 V2 创建 service order → 同步回 S/4 生成 PM order → 技师上门后回写完工记录。每一跳都涉及一次集成调用。

这套链路对接的是 S/4 的标准对象，但 V2 用 communication scenario 的方式管理 API 凭据。Setup guide 强调 communication user 的特点：仅 API 访问、不开 UI、绑定单一 communication scenario。这是个好习惯——不要用一个万能 service account 接所有集成，每条链路单独建用户、单独建场景，出问题时审计能直接定位到点。

```
# Communication User 的最小授权原则
Communication User    →   单一 Scenario   →   单一目标系统
─────────────────────────────────────────
SVC_TO_S4_PM         CS_PM_ORDER         S/4 维保订单
SVC_TO_S4_BP         CS_BP_DATA          S/4 业务伙伴
SVC_TO_KB            CS_KB_SEARCH        知识库 provider
# 一个用户对一个场景，凭据泄露的爆炸半径就被框住了
```

## 五、外部 AI 接进来：Parloa 集成给的信号

Sapphire 2026 公布的 Parloa 合作方式值得单独看一眼。Parloa 是个 AI-native 的呼叫中心平台，它的 voice/chat agent 直接接 Service Cloud V2，调取实时业务数据。客户来电讲一句"我上周下的单到哪了"，Parloa 这一侧就直接走 V2 的 API 把订单状态、退货资格、可触发的服务流程一起取回来，不需要人工 agent 介入。

这件事的架构意义是：SAP 没把 AI 客服锁死在自己的 Joule 里，而是开了一条让外部 AI 平台直接挂载实时业务数据的通道。前提是 V2 的 API 暴露面足够、communication scenario 的颗粒度合理。

出海企业里那些已经在用 Parloa、Cresta、Replicant 等本地 AI 客服平台的客户，可以借此把 SAP 留在 system-of-record 的位置，前端 AI 渠道按当地市场偏好挑。这比把所有 AI 能力都押在 Joule 一家上要灵活。

## 六、落地建议：什么样的项目适合先上、什么时候不要碰

适合先上 Agent Inbox + Case Management Assistant 的场景：

- 跨境电商的售后中心：渠道杂、case 量大、SLA 严，路由负担重，上下文聚合能立竿见影
- 跨国制造的全球服务台：多语种、多时区、多区域队列，Inbox 的 workload insights 帮 manager 实时调度
- 已经在用 SAP S/4 资产模块的客户：Service Management Assistant 那条线的价值要靠 S/4 这边的 PM 数据撑起来

不建议先碰的场景：

- 客户主数据、产品主数据、知识库都还没整理的项目：路由引擎拿不到信号，AI 处置建议命中率会很难看
- 纯轻量级客服（FAQ、订单状态查询为主）：Self-Service Assistant 加个 chatbot 就够了，没必要上完整 V2
- 现有 Service Cloud V1 还能跑、定制化又重的：等到 Joule Assistant 2026 年 11 月 GA 之后再评估，避免 V1 → V2 期间二次返工

最后一个踩坑警示：communication scenario 的设计不要图省事。我见过项目把所有外部集成挂在两三个用户上，上线后任何一条链路出问题，要在日志里翻半天才知道是谁在调谁。前期多花几小时拆细，后期省几十小时排障。

参考来源：
- https://www.savictech.com/insights/sap-autonomous-cx-sales-service-cloud-v2-joule-agents-2026/
- SAP Help Portal: Solution Guide for SAP Service Cloud Version 2 (2026-05-19)
- SAP Help Portal: Set Up Guide for SAP Service Cloud Version 2 (2026-05-12)
