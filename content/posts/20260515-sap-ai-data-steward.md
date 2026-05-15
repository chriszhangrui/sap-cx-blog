---
title: "SAP要当AI的数据管家，但钥匙可能锁在自己口袋里"
date: 2026-05-15T12:16:00+08:00
draft: false
tags: ["SAP CX", "SAP Business Data Cloud", "Forrester", "Dremio", "Prior Labs", "Agentic AI", "数据治理"]
description: "Forrester警告：SAP通过收购Dremio和Prior Labs构建AI数据控制平面，客户在12-24个月内必须做出架构选择，否则锁定不可逆。"
source_url: "https://www.forrester.com/blogs/sap-is-targeting-the-ai-data-control-plane/"
---

Forrester在5月14日发了一篇博客，标题直白得有点刺眼：SAP Is Targeting The AI Data Control Plane。四位分析师联名，在SAP宣布收购Dremio和Prior Labs之后，给了一个不太客气的定性——SAP不只是在买公司，它在抢占你的AI数据控制平面。

## 两笔收购，一个野心

Dremio给SAP Business Data Cloud（BDC）带来的是Apache Iceberg原生的湖仓能力和开放数据目录。说白了，SAP终于有了自己的数据湖底座，不用再完全依赖Databricks或Snowflake来处理非SAP数据。

Prior Labs带来的是表格基础模型——专门针对结构化商业数据的AI能力。这玩意儿的产品化路径已经画好了：SAP AI Core → BDC → Joule，形成一条从数据到决策到执行的完整链路。

把这两件事合在一起看，SAP的意图就很清楚了：它不想只做数据的存储层或治理层，它想做**数据的控制平面**——谁能访问什么数据、数据意味着什么、数据如何驱动决策，这些全归我管。

## Forrester看到了什么风险

Forrester的措辞很讲究——"whether this delivers durable value or deepens dependency"。翻译成人话：这套东西用好了是加速器，用不好就是镣铐。

几个具体的风险信号：

- **架构引力固化**：当语义定义、血缘追踪、访问控制全部跑在BDC上时，迁移成本会指数级上升
- **Dremio的中立性存疑**：被SAP收购后，Dremio的路线图必然要和BDC对齐，非SAP客户的优先级会往后排
- **定价权集中**：数据访问+AI执行+治理全捏在一个vendor手里，谈价的筹码就少了
- **和Snowflake/Databricks的重叠**：你可能在BDC和Databricks上跑着同样的工作负载，双重付费却浑然不觉

Forrester给了一个时间窗口：**12-24个月**。在agentic AI把这些架构选择写死在业务流程里之前，你还有机会做主动的决策。过了这个窗口，改的成本就不是技术问题了，是组织政治问题。

## 站在CX售前的角度想想

说实话，这篇分析让我有点五味杂陈。

好的一面：如果BDC真能把SAP和非SAP数据统一起来并给到Joule，那CX场景的AI落地会快很多。想象一下，一个销售AI agent能直接访问ERP里的库存数据、CDP里的客户画像、还有外部市场数据——这个体验是现在各系统割裂状态下做不到的。

但不好的一面也很明显。我们的客户里有大量混合架构——SAP ERP + Salesforce CRM + Databricks数据平台这种组合不少见。如果BDC强势崛起，这些客户要面对的不是"要不要用BDC"的问题，而是"我的数据主权到底归谁"的问题。

Forrester的建议很务实：先把可移植性设计好，再考虑深度绑定。具体到我们做售前方案的时候，可能需要开始和客户聊一个以前不太聊的话题——你的语义层和治理层打算放在哪儿？这个决定，比选哪个AI模型重要十倍。

> 说到底，这不是一个技术选型问题，是一个控制权博弈问题。SAP在下一盘很大的棋，而棋盘上每一个格子，都是客户的数据资产。

参考来源：https://www.forrester.com/blogs/sap-is-targeting-the-ai-data-control-plane/
