```markdown
---
title: SAP OMF 同步订单创建：当 Commerce Cloud 决定不再存订单
series: 技术深度解读
module: OMS
date: 2026-05-27
source_url: https://help.sap.com/docs/SAP_COMMERCE_INTEGRATIONS/18622cc7ce5f444b8436bad11400e8a1/7ace41db3b394473b3e78805b15725d0.html
source_title: Synchronous Order Creation in SAP Order Management Foundation
source_date: 2026-05-26
---

# SAP OMF 同步订单创建：当 Commerce Cloud 决定不再存订单

## 一、SAP Help Portal 上一条容易被忽略的更新

2026 年 5 月 26 日，SAP Help Portal 在 *What's New in Integrations Extension Pack* 里发布了一条标题很短的条目：

> Synchronous Order Creation in SAP Order Management Foundation —— A B2C customer places an order on the storefront. SAP Commerce Cloud bypasses its own persistence and directly saves the order in SAP Order Management foundation, in real time.

翻译过来：消费者在 Storefront 下单，Commerce Cloud **绕过自身持久化**，直接把订单写到 SAP Order Management Foundation（以下简称 OMF）。订单详情和订单历史，从 OMF 反向取回展示。

这一句话改写了 Commerce Cloud 已经稳定运行十几年的订单数据归属。

之前 Commerce 是单据的"原产地"——订单先落 Commerce 的 Order 表，再异步复制到下游订单中台、ERP、退货系统。现在这条新通道说的是：B2C 单据从一开始就只存在于 OMF，Commerce 表里是空的。

同期还有两条关联条目，串起来就是一整套架构动作：
- *Replicate Orders to SAP Order Management Foundation using the SAP OMF API Version 2.1.0*——OMF API 升到 2.1.0
- *Synchronous Order Replication without Sourcing and Reservation to SAP Order Management Foundation*——同步通道里多了一个"不带 Sourcing 信息"的旁路

三条放在一起看，OMF 这次不是在加 feature，是在挪订单数据的"出生地"。

## 二、之前订单的路径：Commerce 主，OMF 次

要理解这条变化为什么是大的，先回看 OMF 之前的位置。

OMF 在 SAP CX 里被定位成 *modular & composable Order Nervous Center*——可组合的订单神经中枢。按 SAP 官方的 Feature Scope Description（2025 年 9 月最新版），OMF 是一个独立的 BTP 产品，挂在四个 Packaged Business Capability 之下：Order Management Foundation 自己、Sourcing and Availability、Sales Transfer & Audit、Returns Orchestration，分别按 Orders Captured、Orders Reserved、POS Transaction、Returns Captured 计费。

这个架构里，OMF 是订单的"集散地"，不是"产生地"。订单的生产路径是这样的：

第 1 步：消费者在 Commerce Cloud 的 Storefront 下单，Commerce 把订单写进自己的 Order 库。

第 2 步：Commerce 通过事件机制把订单异步推送到 OMF，OMF 收到后做编排（拆单、合单、寻源、预留）。

第 3 步：OMF 编排完成，把履约请求下发到 S/4HANA Cloud Public Edition、Subscription Billing 或第三方履约系统。

第 4 步：履约状态、变更、退货事件再从下游回流到 OMF，OMF 同步给 Commerce、SAC 等系统。

这条路径的好处是 Commerce 自己保留了订单的全部信息，Commerce Backoffice 可以独立查单、改单、查询订单历史。代价是订单数据**有两份**：Commerce 一份、OMF 一份。两边状态保持一致、字段映射、事件失败重放，都是落地项目里要单独写代码处理的活。

异步通道还有一个绕不过去的体感问题：消费者点完"提交订单"，需要看到"订单已生成"的成功页。如果业务侧需要在成功页上同时展示"已分配仓库"或"预计到货时间"，就得等 Sourcing 决策完成。异步链路能不能在合理超时内拿到这些信息，取决于事件总线、消费者数量、OMF 处理延迟——一个项目调三个月并不少见。

## 三、新通道的核心动作：Storefront 直写 OMF

新发布的 Synchronous Order Creation 改了订单生产的源头。

按 SAP Help Portal 的描述，B2C 路径上发生的事情是：

消费者在 Storefront 下单，Commerce Cloud 在订单提交那一刻不写自己的 Order 库，而是通过 OMF API Version 2.1.0 同步调用 OMF，把订单结构（订单头、订单项、客户、配送地址、支付、价格）一次性提交。

OMF 同步完成订单创建（带 Sourcing 信息时同步触发 Sourcing & Availability 决策、生成预留），返回订单 ID 和状态。

Commerce 拿到 OMF 返回值，渲染订单成功页。订单详情页、订单历史页之后展示的内容，都通过 API 回头从 OMF 取——Commerce 自己的 Order 表里没有这条记录。

文档里有一句关键限定，可能比 feature 本身更重要：

> Orders created synchronously have no persistency in the SAP Commerce Cloud system.

同步创建的订单在 Commerce Cloud 里**没有持久化**。这意味着 Commerce 不再是这类订单的 source of truth，OMF 是。

启用这个能力的开关在 Backoffice，名字是 *Enable Synchronous Order Creation and Order History*。打开之后，B2C 流量走同步路径；同步不可用时（OMF 不可达、超时），Commerce 的容错策略需要项目层另外定义——文档里没有把降级路径写死，留给实施方决策。

文档里还有一句限制：

> Orders cannot be created synchronously in SAP Order Management foundation for B2B customers.

B2B 不走这条同步通道。SAP 给 B2B 的同步路径是另一条——SAP S/4HANA OData V4/V2，Storefront 同步写 S/4，订单价格、信用额度、库存可用量在下单瞬间确定。OMF 在 B2B 链路里只参与编排环节。

把这两条限制放在一起：B2C 同步走 OMF，B2B 同步走 S/4HANA。Commerce Cloud 同时是 B2C 同步通道的发起方和 B2B 同步通道的发起方，但订单实际落在哪个系统上，取决于客户类型。

## 四、Sourcing 信息的两种处理：带与不带

新发布里第二条容易被略过的细节是 Synchronous Order Creation 又分两种。

**第一种：带 Sourcing 和 Reservation。** Storefront 提交订单时，Commerce Cloud 调用 SAP Order Management for Sourcing & Availability，先决定订单按什么仓拆、每行预留多少，再把带 Sourcing/Reservation 信息的订单同步写进 OMF。OMF 收到的是已经决定好仓和预留的订单。

**第二种：不带 Sourcing 和 Reservation。** Commerce 直接把订单同步写进 OMF，订单里不带任何 Sourcing 决策。OMF 把订单复制到下游 S/4HANA，由 S/4HANA 在订单到达后做寻源、按 ATP 算可用量、生成预留。

这两种模式选哪一种，取决于业务的库存定位：

- 如果货分布在多仓、需要按"离消费者最近的仓"或"该仓库存最足"实时决策——第一种合适，Sourcing 是前置决策点。
- 如果库存集中在 S/4HANA 体系内、ATP 已经在 S/4 里做得很完整——第二种更轻，省掉一层 Sourcing 系统调用。

值得注意的是 OMF 的 Reservation Management 页面（2026-05-12 更新）提到 Orders API 提供两个 reservationId 字段：订单头的 sourcing 信息上挂一个，订单行上挂一个。这个设计是为多寻源服务并存留的——一笔订单里部分商品来自自营仓、部分商品来自第三方供应商，每段 Sourcing 决策独立，预留 ID 各自归属。多供应商电商场景里，这条字段定义直接影响数据模型设计。

## 五、OMF API 2.1.0 是这次同步通道的入口

支撑新通道的接口是 SAP OMF API Version 2.1.0，What's New 里独立列了一条：

> The integration of SAP Commerce Cloud with SAP Order Management Foundation enables replication of order-related data to SAP Order Management Foundation using the SAP OMF API Version 2.1.0.

API 版本号从之前的 2.0.x 升到 2.1.0，向 Commerce 以外的其他订单源也开了同步入口。这点对于已经在 BTP 上自建 Storefront、或者使用第三方电商前台对接 SAP 后端的场景，价值在于：同步写 OMF 不再是 Commerce 专属的能力，OMF API 2.1.0 直接面向所有订单源开放。

OMF 的整个能力范围（按 2025 年 9 月版 Feature Scope Description）是这样的：

- 订单处理：接收订单、搜索订单、延迟编排、履约中改单/取消、重新触发、自定义编排策略
- 订单活动管理：自定义活动类型、SAP 预置活动类型、活动生命周期跟踪
- 与履约系统集成：实物商品对接 S/4HANA Cloud Public Edition、订阅商品对接 SAP Subscription Billing、第三方履约
- 订单相关主数据复制：商业伙伴、产品主数据
- 分析系统集成：对接 SAP Analytics Cloud
- 自动化订单处理流程的管理与监控
- 自定义寻源规则配置（与 Sourcing & Availability 集成）
- 退货订单处理（2025 年 7 月新增）：创建、查看、监控、下发履约

API 2.1.0 现在覆盖的是"订单接收"这一项的同步入口。后续版本是否把"履约中改单"、"退货处理"也同步化，文档里没明说，但从这次升级方向看，逐步把异步通道补齐成"异步可选、同步可选"是大概率的事。

## 六、对中国出海企业落地实施的影响

需要先说清楚一个事实：OMF 在中国大陆没有数据中心。这个产品只对有海外业务、可以使用 SAP BTP 欧美区域数据中心的客户开放——出海电商、跨境零售、海外子公司支持业务都在这个范围内。

以下几个判断点，留给在评估这条新通道的团队：

**判断点一：订单数据是否还能拆两份。** 如果业务上有 Commerce 端独立的订单运营场景（运营人员在 Commerce Backoffice 直接处理订单），同步通道意味着这个场景没了——Commerce 表里没数据，运营得切到 OMF 端去处理。决策人需要先问运营团队：日常处理订单是只在一个系统里完成，还是确实需要两边？

**判断点二：成功页 SLA 谁背。** 同步通道的体感优势是订单成功页一次确认到位，但一次同步调用要在 200ms 内打通 Commerce → OMF →（带 Sourcing 时）Sourcing & Availability → 返回。任何一段超时或失败，下单页就报错。降级策略——失败时是阻断下单还是回退到异步路径——需要在选型阶段就和业务谈清楚。

**判断点三：B2B 和 B2C 路径是否分开做。** 如果业务同时有 B2C 和 B2B（不少跨境零售品牌都有 To B 批发分销线），按 SAP 现在的设计，这两条路径走不同的同步通道：B2C 直写 OMF，B2B 直写 S/4HANA。两条通道的数据归属、运营页面、客户服务工具都不一样，技术架构图上就是两套同步集成。

**判断点四：Sourcing 模式选哪种。** 带 Sourcing 的同步路径需要 Sourcing & Availability 这个能力（计费按 Orders Reserved），不带 Sourcing 的路径把寻源交给 S/4HANA。前者灵活但多一层订阅成本，后者轻但只适合库存集中在 S/4 的场景。库存分布拆得越散，越倾向前者。

**判断点五：OMF API 2.1.0 是否替代 Commerce。** 自建 Storefront 的客户现在多了一个选项——直接对接 OMF API 2.1.0，跳过 Commerce。这条路适合不需要 Commerce 商品目录、营销、Storefront 能力的客户（比如订阅服务、纯 B2C 品牌官网）。但要注意，跳过 Commerce 也意味着没有 Backoffice、没有 SmartEdit、没有 Customer Support Module——这些工具的替代方案要在项目里另算。

## 七、写在最后

这条更新本身不大，一行 What's New，加一个新版 API。但它把 OMF 在 SAP CX 体系里的位置又往中心挪了一格——从"订单集散地"挪向"订单生产地"。

对架构师来说，需要重新评估的事情是：B2C 订单的 source of truth 从 Commerce 迁到 OMF，对运营、客服、报表、退款链路、合规审计的下游影响是什么。这些影响散落在十来个工具和流程里，不是一次升级能一次说完的。

这条同步通道的设计目的是简化订单架构、提升下单体感，但它要求的前提是把订单数据的"产权"明确划给 OMF。这件事谁先想清楚，谁先动。

---

**参考来源**

- *Synchronous Order Creation in SAP Order Management Foundation*. SAP Help Portal · What's New in Integrations Extension Pack · 2026-05-26
- *Replicate Orders to SAP Order Management Foundation using the SAP OMF API Version 2.1.0* · 2026-05-26
- *Synchronous Order Creation without Sourcing and Reservation* · 2026-05-26
- *Reservation Management* · SAP Order Management Foundation · 2026-05-12
- *Feature Scope Description: SAP Order Management foundation*, version 6, 2025-09-30
```
