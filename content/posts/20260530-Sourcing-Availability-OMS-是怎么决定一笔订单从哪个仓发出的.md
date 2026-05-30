---
title: "Sourcing & Availability：OMS 是怎么决定一笔订单从哪个仓发出的"
date: 2026-05-30T10:00:00+08:00
draft: false
tags: ["SAP", "OMS", "Order Management", "Sourcing", "技术深度"]
description: "OMS 把订单引擎拆成四块独立计费的能力包，Sourcing & Availability 是其中最少被讲清楚的一块——它解决的是一个老问题：履约该走哪个仓。这次它不再藏在 ERP 里。"
source_url: "https://community.sap.com/t5/sap-learning-blog-posts/eager-to-learn-how-sap-ensures-the-right-product-at-the-right-time/ba-p/14348214"
---

## 一个出海品牌每天都在做的判断

一笔订单进来。德国仓有货、波兰仓也有货、第三方 3PL 在荷兰还压着一批。先发哪个？

听上去像运营问题，落到系统里其实是一道实时计算题：库存准确度、运费、SLA、关税、剩余产能、退货率——每一项都在变，每一秒都在变。过去这道题塞在 ERP 里靠规则跑，规则一多，没人敢碰。

SAP Order Management Services（OMS）把这道题单独拎了出来，做成一块独立计费的能力，叫 **Sourcing & Availability**（S&A）。

## OMS 的四块能力，先理清楚

OMS 现在不是一个大单体，而是四块按消耗计费的 PBC（Packaged Business Capability）：

● **Order Management Foundation（OMF）**：订单捕获、状态机、生命周期管理。按 *Orders Captured* 计费。

● **Sourcing & Availability（S&A）**：决定订单从哪发、能不能承诺。按 *Orders Reserved* 计费。

● **Sales Transfer & Audit**：门店 POS 交易回传与审计。按 *POS Transaction Captured* 计费。

● **Returns Orchestration**：退货编排。按 *Returns Captured* 计费。

四块能力各自独立订阅、独立扩容，整个 OMS 跑在 BTP 上，对外是 API + Events，没有 UI 强绑定，前端可以是 Commerce Cloud、可以是自研站、也可以是任何 headless 前台。

## Sourcing 引擎的核心：KPI 加权评分

S&A 决定 "从哪发" 的方式不是 if-else 规则树，而是一套加权评分。每一个候选发货地点（门店、DC、3PL 仓），引擎会算一个分数：

```
score(location) = Σ(KPI_i × weight_i)

KPI 维度示例：
  - 库存可用量（Available Quantity）
  - 距离收货地（Distance）
  - 履约成本（Fulfillment Cost）
  - 历史 SLA 达成率（On-time Rate）
  - 当前积压（Backlog Pressure）
  - 渠道优先级（Channel Priority）
```

权重由业务侧维护——不同品类、不同渠道、不同促销期可以挂不同权重模板。促销季想清库存，把"库存可用量"权重拉高；平销期想压成本，把"履约成本"拉高。规则不写死在代码里，这是它和老式 sourcing 引擎最大的区别。

## Sourcing Simulation：先模拟再下决定

S&A 还有一个常被忽略的能力：**Sourcing Simulation**。订单还没真正提交前，可以先发一个 simulate 请求，引擎返回一个候选发货方案、预计到达时间、预计成本——但**不锁库存**。

这个能力的价值在 PDP 和购物车页面：用户还在看，前台已经知道"如果你现在下单，会从波兰仓发，明天到，运费 7.9 欧"。结账转化率会涨，是因为不确定性被前置消解了。

技术上，simulate 调用走的是只读路径，不进 reservation 表，不计入 *Orders Reserved* 配额——这是它和正式下单的关键区别。

## Re-sourcing：发货前最后一刻还能换

订单已经分配到德国仓，但德国仓在拣货前发现实物盘亏，或者突然来了一笔更高优先级的渠道订单——这时候系统能不能换到波兰仓重新发？

S&A 的 **Re-sourcing** 路径就是干这个的。触发条件可以是：

● 库存事件（盘亏、损耗、被抢占）
● SLA 风险事件（拣货延迟、承运商爆仓）
● 人工干预（客服强制改派）

re-sourcing 会把原 reservation 释放，重新跑一次评分，挑新仓，重新锁库存。整个过程对前端透明——订单号不变、客户感知只是"发货地变了"。

## Orders Reserved 的计费逻辑要算清楚

S&A 按 *Orders Reserved* 计费，这里有两个容易踩坑的细节：

● **Simulation 不计费**，只有真正进入 reservation 才计——所以前台可以放心高频调 simulate。

● **Re-sourcing 算几次**？官方口径是同一个 order 的多次 re-source 计为 1 次 Orders Reserved，不会因为你换了三次仓就被收三次钱。

合同谈判时这两条要写进 SOW 里，不然预算很容易测算偏。

## 集成的几个硬约束

S&A 不是一个孤岛，它要吃数据、要吐事件：

● **库存数据源**：S&A 自己不存现货数量，靠 Central Inventory（OMS 内部组件）或外部 ERP/WMS 实时同步——同步频率和准确度直接决定评分质量。

● **事件总线**：所有状态变化（reserved、re-sourced、released、shipped）都通过 Event Mesh 广播，下游系统订阅消费。

● **API 风格**：REST + JSON，OAuth 2.0，全异步——下单后拿到的是一个 order id 和一个 polling/webhook 入口，不是同步的发货结果。

这意味着前端工程上要做异步状态管理，不能再按"提交订单 → 立即拿到发货仓"这种同步思维写代码。

## 什么样的企业值得上 S&A

适合的：

● 多仓、多渠道、跨境履约的出海企业，仓位 ≥ 5
● 有明显促销季 / 大促节奏，sourcing 策略需要按期切换权重的
● Commerce 或自研站需要在 PDP / Cart 页面展示交付承诺的
● 已经在用 OMS Foundation，想分阶段把履约决策权从 ERP 抽出来的

不适合的：

● 单仓、单渠道，sourcing 决策每天稳定的——继续用 ERP 规则就够了
● 履约完全外包给 3PL，自己不做 sourcing 决策的
● 国内单一市场、订单量小、没有跨境——OMS 在中国没有数据中心，强行上反而徒增延迟

## 结尾的判断

Sourcing & Availability 的真正价值，不是"算得快"，而是把履约决策从 ERP 的 batch 思维里解放出来，变成一个可以被前台、被客服、被运营实时调用的服务。

它解决的不是新问题，是把一个老问题用 composable 的方式重新做了一遍。对于已经在 BTP 上的客户，它是 OMS 模块化路线里最该先评估的一块。
