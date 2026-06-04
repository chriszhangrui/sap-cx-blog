---
title: "Engagement Cloud 这次把数据底座悄悄重做了"
date: 2026-06-04T16:00:00+08:00
draft: false
tags: ["SAP CX", "Engagement Cloud", "技术深度", "Emarsys", "CDP"]
description: "Q2 2026 这次没改渠道也没动 AI 路线图，真正动的是数据层。USB、内嵌 CDP Audience Builder、Flexible External Contact ID 这三件事联起来，营销执行层第一次能直接消化外部数据库。"
source_url: "https://emarsys.com/learn/blog/sap-engagement-cloud-q2-2026-release/"
---

Engagement Cloud Q2 2026 在 6 月初挂出来。看完发布说明，第一感觉不是「又加了几个 AI 能力」——AI 那一段反而是这次最容易预测的部分，Joule 接进来、Gemini 用于内容生成、Email 个性化进 pilot，路线图早写过。

真正值得停下来看的，是**数据层这次重新动了一遍**。

Universal Schema Builder（USB）从 pilot 出来，CDP Audience Builder 内嵌进 Engagement Cloud（GA），Flexible External Contact ID Management 转 GA，再加上 SAP CDP Connector 的 Business Area 映射也 GA。这四件事单看每一条都不大，但联起来，等于是把 Engagement Cloud 这一层和外部数据系统之间的耦合关系换了一种写法。

如果做过 Emarsys 的实施，看到这里大概就明白这件事的分量。

## 为什么这件事重要：旧链路的两个堵点

Emarsys 时代——以及改名后的 Engagement Cloud 在 Q1 之前——外部数据进入营销执行层走的基本是两条路。

第一条是 Predict / Smart Insights 那一路，依赖一个相对固定的商品和行为数据模型，导入的是用户行为流。这条路对零售场景很顺，但凡业务模型不太「零售」——比如 B2B、保险、教育、订阅服务——schema 就不那么贴。

第二条是 Relational Data，把外部表导进来挂在 contact 旁边。问题在两处：一是数据要落到 Engagement Cloud 内部，本质上是**数据搬家**；二是每个 campaign 拿这块数据做分段或个性化的时候，相当于每条 campaign 各自起一份引用关系，并发场景下要么撞到限制，要么导致数据重复。

这就是为什么很多客户在 Emarsys 旁边还跑着一个独立的 CDP——把跨系统的身份和分段问题在外面先解决掉，再把分段结果当成 list 灌进来。这个架构能跑，但代价是 **Engagement Cloud 自己的分段引擎一半是闲置的**。营销人员真正用的是 CDP 那一头的分段能力，Engagement Cloud 退化成一个发送通道。

Q2 这次动的就是这一条。

![Q2 2026 数据层架构](https://emarsys.com/learn/blog/sap-engagement-cloud-q2-2026-release/)

## Universal Schema Builder：把外部数据库变成 schema 入口

USB 这次的关键描述是这一句——「use data from external databases and combine segmentation and personalization queries into a single schema」——以及「USB supports concurrent campaigns，meaning marketers can use the same Universal Schema across multiple campaigns simultaneously」。

把这两句技术化一点理解：

- Universal Schema 是一个抽象层，对接外部数据库（按现有 SAP 集成栈来推断，BigQuery / Snowflake / S/4 视图都可以是它的数据源）
- 分段查询和个性化查询**共用同一个 schema 定义**，不再像 Relational Data 那样各自映射
- 一个 USB 可以并发服务多个 campaign，意味着 schema 是 **campaign-agnostic 的中间层**，而不是 campaign-bound 的数据快照

在发布说明里它特别强调「This improves upon existing Relational Data functionalities」，意思也很明白——Relational Data 不是被废弃，是被让位。新项目应当走 USB。

这件事对架构的影响是：**Engagement Cloud 第一次有了一个稳定的、外部数据 in-place 的查询通道**。不再被迫把 BigQuery 里的会员消费明细每天 ETL 进 Engagement Cloud，schema 注册一次就好。

USB 现在是 Pilot——这个状态的意思是接口、性能边界和 SLA 都还在动。如果有 POC 想做，要明确两个问题：

- 外部数据源的查询延迟会不会冲击实时分段（Engagement Cloud 的分段重算节奏）
- schema 变更的版本化怎么处理（外部表 alter 之后 USB 是否需要 rebuild）

## CDP Audience Builder 内嵌：分段从外面挪进来

第二件事是 SAP CDP Audience Builder 直接嵌进 Engagement Cloud（GA）。

发布说明里有一句很关键：「The contacts will appear as a CDP segment type once created, ready for use in campaigns, personalization, and automation.」

注意这里用的是 **segment type**——不是 list、不是 import。这是一个原生类型。意味着 CDP 那边创建的 audience，到 Engagement Cloud 里不是被翻译成本地分段，而是被识别为另一种分段类型，仍然由 CDP 这一端拥有定义和刷新。

这个设计回答了之前那个老问题：客户买了 SAP CDP（Customer Data Platform，前身 Hybris Profile）之后，CDP 的分段能力和 Engagement Cloud 的分段能力之间怎么分工？

Q2 给出的答案是——**不分了**。CDP 拿身份和跨系统画像，Engagement Cloud 拿渠道执行和 campaign 工程化。CDP Audience Builder 内嵌之后，营销人员在同一个界面里既可以用 Engagement Cloud 自己的分段，也可以直接调 CDP 的分段，两种 segment type 都能进 campaign。

这种合一对外资品牌在中国市场的实施有特别意义。SAP CDP 在中国不落地（数据中心在欧美），但跨境电商和出海品牌用 CDP 把欧洲、东南亚、北美的身份归一是常规做法。这次内嵌之后，**营销团队不再需要在两个 UI 之间切换**来对照分段定义，对实施工作量是实打实的减少。

## Flexible External Contact ID：身份层第一次说人话

第三块是 Flexible External Contact ID Management 转 GA。这一条放在三个变化里块头最小，但对实施的影响最直接。

以前 Emarsys 的 contact 模型有一个隐式假设——**contact 的主键由 Emarsys 自己定义**。外部系统的 customer ID（比如 Commerce Cloud 的 customer PK，或者自建会员系统的会员号）进来之后，要么被翻译成 Emarsys 内部 ID，要么作为某个 custom field 自己想办法对齐。

听起来不是大事，但实际项目里这是数据双写最容易出错的一环。一个客户在 App 里下单（Commerce Cloud 主键 A），又在邮件里点击优惠券（Emarsys session 用了 hashed email B），如果中间没有一个稳定的身份解析机制，**很容易在 Emarsys 这边出现两个 contact**，然后画像和频次控制就开始失真。

这次的「flexible mapping between your external system contact identifiers and SAP Engagement Cloud contact records」实际上是把外部 ID 升级为 contact 模型的**一级映射键**，并且明确支持多个外部系统并存——配置里可以挂多个 external system，每个有自己的 ID 字段。

这件事的工程价值是把「身份解析」这一步从 iFlow 里拆出来，挪回到了 Engagement Cloud 自己身上。换句话说：

```
// 旧链路（CPI 里做映射）
Commerce → CPI iFlow → resolve(externalId → emarsysId) → Engagement Cloud

// 新链路（Engagement Cloud 自己接外部 ID）
Commerce → CPI iFlow（pass-through） → Engagement Cloud (external ID as native key)
```

CPI 这一段从「业务逻辑层」回退到「传输层」。这对维护的人是好事——iFlow 里的身份解析逻辑历来是项目交接最容易掉链子的地方。

## Business Area 映射：CDP Connector 这次补上了路由

最后一件事相对小：SAP CDP Connector 增加了 Business Area 映射（GA），同时还加了 ID Mapping Support for Standard Sales Data（Early Adopter）和 Default Business Area 重命名（GA）。

Business Area 是 Engagement Cloud Enterprise Edition 引入的多事业部隔离能力。**在 Q2 之前，CDP 同步过来的 contact 没办法按 Business Area 自动路由**——要么所有 contact 进默认 BA，要么靠 CPI 在中间插一层 BA 字段补全。

这次 CDP Connector 直接支持源端 BA 映射到目标 BA。对集团型客户（控股下面挂多个独立运营品牌的那种）来说，这是「配置可用」和「项目可用」的差别。

## 把四件事联起来：身份层从 iFlow 收回到 Engagement Cloud 自己

如果只看单条更新，每一条都像是「又增强了一个能力」。但叠在一起看，**Q2 实际上完成了一次身份层的归位**。

Emarsys 时代的隐含分工是：身份解析在外面（CPI / 自建 ID 服务 / CDP），Emarsys 拿到的就是「已解析过的 contact」。Q2 之后这个分工变成：

- 外部数据：USB 直接对接，不必复制
- 外部分段：CDP Audience Builder 内嵌，不必导出
- 外部 ID：Engagement Cloud 自己当成 native 字段处理，不必中间翻译
- 外部组织结构：CDP Connector 自己 map BA，不必中间路由

结果是：CPI 这条管子被减薄了，Engagement Cloud 这一层的责任范围扩大了。

也可以反过来理解——这次发布所反映的 **SAP 内部的产品边界** 正在变化。CDP 和 Engagement Cloud 不再是松耦合的两个独立产品互发数据，而是 **在功能视图上合并成一个体验，但底层仍然是两个独立部署的服务**。这种「前台一体化、后台还分家」的形态，是 SAP CX 这两年的典型套路（Sales Cloud V2 和 Service Cloud V2 也是这个路子）。

## 落地建议：哪些项目应当现在就动

结合在出海客户里看到的实际情况，可以这样判断：

- **已经在用 SAP CDP + Emarsys 双产品组合的**——CDP Audience Builder 内嵌（GA）值得马上规划。这是前几年自己用 iFlow 拼出来的东西，现在变成产品能力，能省一份运维。
- **外部数据量大、之前一直在用 Relational Data 的**——USB 还是 Pilot，可以先在非核心 campaign 上做 POC，等 GA 之后做 Relational Data 的迁移评估。注意 USB 的查询延迟和外部数据库的承压能力是两个新变量。
- **还在 iFlow 里维护身份解析逻辑的**——Flexible External Contact ID 是一个明确的简化机会。但迁移过程要谨慎：把已经在 iFlow 里跑了几年的身份逻辑搬走，意味着要重新验证一遍 contact 唯一性。预留至少一个 sprint 做对账。
- **集团型客户、原来 BA 用得很重的**——CDP Connector BA 映射 GA 之后，可以把之前在 CPI 里的 BA 路由代码逐步退役。

反过来，如果项目**还没用上 CDP，并且外部数据需求不大**——这次更新的影响有限。Engagement Cloud 的核心 campaign 流程没动，Joule 那条线继续是 EAC 阶段，可以观望。

## 一个容易被忽略的点：USB 改变的不只是数据接入

发布说明里有一句话很短：「personalization fields can now be used independently without relying on the same USB segment data, giving you more flexibility in how you build and deliver personalized experiences.」

翻译过来：分段查询和个性化查询**虽然定义在同一个 schema 里，但运行时可以独立调用**。

这其实是一个很重要的解耦。旧的 Relational Data 模式下，做分段拉到 1000 个 contact，发邮件时给每人填的个性化字段必须从同一份分段结果里取。USB 之后，**分段拉一次，个性化时每条邮件可以重新查 USB 拿最新值**——意味着发送时刻的库存、价格、促销状态可以是 fresh 的，而不是分段刷新时刻的快照。

对零售客户来说，这是个性化能力的一次实质升级。值得在 POC 里验证清楚的是：高并发下个性化查询打到外部数据源的 QPS 上限怎么定，超出之后是降级到分段时刻快照，还是直接发送失败。

## 小结

Q2 2026 给 AI 留了头条，但真正改变实施工作量的是数据层这四件事。这是一次**少见的「边界后撤」**——SAP 没有把 Engagement Cloud 推到更外面去当 CDP 用，而是把它和 CDP 的协作方式重新梳理了一遍，把过去靠 iFlow 拼起来的部分变成了产品自带能力。

营销执行层这一次终于不只是「通道」。

参考来源：https://emarsys.com/learn/blog/sap-engagement-cloud-q2-2026-release/
