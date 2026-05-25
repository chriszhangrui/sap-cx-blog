---
title: "Sales Cloud V2 把 ERP 集成从项目题变回产品题"
date: 2026-05-25T16:10:00+08:00
draft: false
tags: ["SAP CX", "Sales", "Sales Cloud V2", "S/4HANA", "BTP", "技术深度"]
description: "出海制造企业最痛的不是没 CRM，是销售看到的价格、库存、订单永远比 ERP 慢半拍。V2 把这一段重写了：一条 BTP 标准集成内容，把价格、ATP、订单历史直接拉进销售工作台。"
source_url: "https://www.spadoom.com/en/blog/sap-sales-cloud-v2-for-manufacturing/"
---

做出海制造业项目，有一个反复出现的场景：销售在客户现场被问"500 件这个料号现在多少钱、什么时候能交"。他打开 CRM，看到的是三周前导出的价格表和一份手工维护的库存清单。两天后采购部门打回来——价格变了、库存早被另一个区域占用了、报价作废。

这个问题不是没有 CRM，而是 CRM 和 ERP 之间隔着一层人为同步。SAP Sales Cloud V1（C4C，Cloud for Customer）也接 ERP，但走的是夜间批量、缓存表、自定义中间件那一套。Sales Cloud V2 把这一段拆掉重写，给出的答案是：BTP Integration Suite 上跑一套 SAP 出厂的标准集成内容，销售工作台里的价格、ATP（Available-to-Promise，可承诺库存）、订单历史，全部从 S/4HANA 实时拉。今天这篇深读，就讲这条链路怎么搭、放弃了什么、什么样的项目适合上。

## V2 不是 V1 升级，是把 ERP 集成这件事从"项目题"变回"产品题"

原文里有一句话点得很直接：V2 不是 C4C 涂层翻新，是从地基重建。具体到 ERP 集成这一块，意思是这样的——

- V1 时代：每个项目自己去画 BAPI（Business Application Programming Interface）调用图、自己实现增量同步、自己处理重试和幂等。中间件经常是 PI/PO 或者第三方 ESB。
- V2 时代：SAP 把 Sales Cloud ↔ S/4HANA 的关键场景做成 BTP Integration Suite 上的标准集成内容（pre-built integration content），项目上做的事是配置和扩展，而不是从零写接口。

这个区别看起来只是工程方式的迁移，但实际影响很大。原文有一句细节值得划重点：这套集成内容同时支持 S/4HANA Cloud、S/4HANA on-premise，以及还在跑 ECC 6.0 EhP7/EhP8 的客户。这意味着对于很多还没上 S/4HANA 的出海制造企业，Sales Cloud V2 不是"以后等你升 ERP 再说"，而是现在就能接。

![Sales Cloud V2 与 S/4HANA 集成链路](http://mmbiz.qpic.cn/mmbiz_png/lWqJzSMIBLWBAF1oQH9UaSHTavZVr3YXnwNRsMbR9zrbY4K4COYSPSIKc5GJtWibbPsGF8MCyEg0JPX8LZLEscMhNwtt9899eYTow7eG84oA/0?from=appmsg)

## 实时定价、ATP、订单历史，链路上其实是三件事

把"实时拉 ERP"这句话拆开看，Sales Cloud V2 在销售工作台里其实在做三件不太一样的事，链路走法和成本也不同。

**第一件：实时定价。** 销售在 opportunity 里建一条 quote item 时，前端触发一次 OData V4 调用，经 BTP Integration Suite 的 pricing iFlow，进 S/4HANA 的物料主数据和定价过程，把客户专属价、阶梯折扣、币种换算后的最终价拉回来。这一段是同步调用，对 P95 延迟很敏感——销售在客户面前，不能等 5 秒。

**第二件：ATP 库存查询。** 销售点开"可用性"，链路类似定价，但落到 ATP check，按工厂（plant）、仓库、储位（storage location）过滤。这里有一个被忽略的细节：制造业很多客户是多工厂结构，ATP 不能只回一个总数，要按可发货工厂分别返。BTP 上的标准 iFlow 内置了这层结构，不需要项目自己拼。

**第三件：订单历史时间线。** 这一段不是同步的。销售打开客户账户，时间线里出现的销售订单、发货单、发票，背后是 S/4HANA 通过事件机制推到 BTP 的事件总线，再写进 Sales Cloud V2 的客户聚合视图。读的时候是本地查，写的时候是事件驱动。这种读写分离的设计对长尾客户特别有效——你不会因为某个大客户有十万条历史订单，就把销售工作台拖到打不开。

从 V1 OData V2 切到 V2 OData V4，对接接口的写法也变了。一个最小的实时定价请求大概长这样：

```http
GET /sap/c4c/odata/v4/sales/quote-items('QI-2026-00731')/PriceLookup
Authorization: Bearer eyJhbGciOi...
Accept: application/json
Prefer: return=representation

// 返回示例（节选）
{
  "@odata.context": "...$metadata#PriceLookup",
  "MaterialId": "MFG-VALVE-DN50-SS316",
  "CustomerSpecificPrice": "342.80",
  "Currency": "EUR",
  "ConditionType": "PR00",
  "ScaleApplied": "TIER_500_PLUS",
  "PriceDate": "2026-05-25T08:14:32Z",
  "SourceSystem": "S4H_PRD_120"
}
```

对从 V1 过来的项目，最大的迁移成本就在这里。OData V4 的查询语法、批处理、错误模型都和 V2 不一样，每一个跟 Sales Cloud 通讯的中间件适配器都要重写。这不是"升个版本"能糊过去的，原文用的词是"period"——没得商量。

## 销售流程层：territory、CPQ、Joule 各自在哪一层介入

除了 ERP 集成，Sales Cloud V2 还有几块东西值得分开看，因为它们在架构上属于不同层，混在一起讲就糊了。

**Territory & Quota（区域与配额）：核心 CRM 之上的元数据层。** 区域定义可以按地理、产品线、客户分段或组合，单个销售可以同时挂多个 territory，territory 之间允许重叠（典型场景：大客户经理跨区独立挂一个）。配额按层级 cascade——VP → 区域总监 → 团队负责人 → 个人，进度自动上卷。这块的设计不算革命性，胜在和 lead routing rule 联动得彻底，新 lead 进来自动按规则分配，不再需要"周一早会分单"。

**SAP CPQ：复杂产品的配置定价层。** 制造业有大量 variant configuration（变量配置）的产品——一个阀门可能有口径、材质、涂层、附件四五个维度。Sales Cloud V2 自己不做这件事，而是和 SAP CPQ 集成，让销售走一条 guided configuration（引导式配置）流程，最后产出技术合法 + 价格正确的报价。CPQ 是另一个独立的产品，不要把它当 Sales Cloud 的子模块——它有自己的规则引擎、产品建模和发布流程。

**Joule（生成式 AI）：横跨多层的 copilot 层。** 原文提到 Joule 在 V2 里负责三件事：基于真实赢单率训练预测模型、识别停滞商机、客户拜访前生成简报。需要注意的是，Joule 不是"再做一个 BI"——它读的是 V2 已经聚合好的数据（订单、商机、服务工单、互动记录），所以要 Joule 用得好，前提是底下的 ERP 集成、time line、活动记录都已经接通了。先有数据，再有 AI，顺序不能反。

## 哪些出海项目适合上、哪些不要碰

把这套架构换算到中国出海企业的实际场景，下面这几条是值得放在桌面上的判断。

**适合上：**

- 已经在跑 SAP ERP（不论 S/4HANA 还是 ECC 6.0 EhP7/8）的出海制造企业，海外销售团队 30–200 人区间最甜——大到值得做正经 CRM，小到不至于陷入多年项目。
- 海外分销商网络复杂、客户专属价和阶梯折扣多的工业品企业，ERP 实时定价拉过来这件事直接顶掉一个常年扯皮的痛点。
- 已经规划上 SAP CPQ 的复杂产品厂商，V2 的报价层是天然衔接点，不需要再搭独立 BTP 集成。

**不要碰或先别碰：**

- 后端不是 SAP ERP（用 Oracle、Infor 或自研系统）的企业，集成优势完全用不到，那条 BTP 标准内容白白浪费，剩下的 CRM 能力相比 Salesforce 没有压倒性优势。
- 还在 V1（C4C）上跑得不错、且没有出海拓展计划的客户：V1→V2 是数据模型、API、扩展框架、UI 框架的全套重写，不是升级，是重做项目。除非有明确业务驱动，不要主动去碰。
- 海外销售场景里离线工作不重要的纯总部办公团队：V2 的移动端 + 离线同步是亮点，但如果场景用不上，亮点变不成业务价值。

> 实施侧的踩坑点（来自原文，反复印证）：第一，环境申请慢，V2 租户和 BTP subaccount 的 provision 不是即时的，要在 Discover 阶段就发起申请，否则 Prepare 阶段会卡住；第二，第一阶段不要超过 3–5 个 pipeline stage，制造业项目特别容易把内部销售方法论里的 8 段 + 子状态全部搬过来，结果是销售用不起来；第三，UAT 名单要在 Explore 阶段就锁定并真的让他们参加 workshop，等 UAT 才出现的人，一定会提一堆改动需求把上线计划打乱。

## 结语

Sales Cloud V2 这次最值得看的不是 UI 或 AI，而是它把 ERP 集成从"每个项目自己造"重新变成"SAP 出厂内容 + 项目侧扩展"。这种产品化是有代价的——它对客户的 ERP 现状、对集成中间件的选择、对 V1→V2 的迁移路径都提出了硬约束。但对一个海外销售场景跨多国、产品复杂、ERP 是 SAP 的出海制造企业来说，这套约束换来的是销售那一刻看到的真实价格和库存。这件事的业务价值，在客户会议室里被问到的那 30 秒内就会兑现。

参考来源：https://www.spadoom.com/en/blog/sap-sales-cloud-v2-for-manufacturing/
