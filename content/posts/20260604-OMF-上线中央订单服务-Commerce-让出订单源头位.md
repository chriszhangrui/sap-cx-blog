---
title: "OMF 上线中央订单服务：Commerce 让出订单源头位"
date: 2026-06-04T12:30:00+08:00
draft: false
tags: ["SAP CX", "OMS", "OMF", "技术深度"]
description: "Q3 这一波 OMS 的更新里 Central Order Service 和 API V3 是一对——前者把订单的主权从 Commerce 挪到 OMF，后者是为多上游写入重写的吞吐发动机。"
source_url: "https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/sap-commerce-cloud-q3-25-release-highlights/ba-p/14231991"
---

# OMF 这次上线的 Central Order Service，把 Commerce 的订单源头位置拿掉了

做过几个 omnichannel 项目的人都遇过同一个尴尬：客户在网上下了一单，去门店退货，店员在 POS 里查不到订单，得反向打电话让客服把订单号读过来手抄进 ERP。这不是配置没做好，是底层根本就没有"一份订单台账"——线上订单待在 Commerce Cloud 的 `Orders` 表里，门店交易在 POS 自己的 TLOG 里，第三方渠道又走自己的 webhook。BORIS（Buy Online, Return in Store）这个口号喊了快十年，技术底盘一直差一截。

SAP Commerce Cloud 2025 年第三季度的 release 里，OMF（Order Management Foundation）多了一个不太显眼的能力：**Central Order Service**。配套的还有一个被称作"performance & scalability 重写"的 **API V3**。这两件事放在一起看，比单看任何一个都重要——它意味着 OMF 第一次把自己摆在了"订单的真源头"位置上，Commerce Cloud 反过来变成了一个写入方。

## Commerce Cloud 不再是订单的家

按老的 OMF 模型，订单是先在 Commerce Cloud 创建，再通过 OMF API 同步给中央订单管理。Commerce 这边留一份，OMF 留一份，两边靠 ID 对齐。这个设计有个隐性假设：所有订单都来自 Commerce。但只要客户上了门店、上了 marketplace，这个假设就破了。

Central Order Service 把订单的"主权"挪到了 OMF。原文写得很直白：

> The release of Central Order Service for SAP Order Management Foundation introduces a robust foundation for Unified Commerce, seamlessly integrating online and offline transactions. Additionally, the Central Order Service empowers businesses with a comprehensive 360-degree view of customer purchase histories.

把这段话的潜台词拆开：在线订单、门店订单、第三方渠道订单，第一次共用一份订单台账。这才是 BORIS / BISRO（Buy in Store, Return Online）能在一个账户视角里执行的前提。客户在门店退一笔网单，店员看到的不是"网订单的代理记录"，而是和原下单完全等同的一条订单。

这一步的副作用是 Commerce Cloud 那边的角色变了。它不再是订单的"创建者"，更像是一个"在线渠道的下单入口"。这对原本把 Commerce 当订单中枢的实施模式是一次方向性的修正——你的订单逻辑要写在 OMF 这一层，写在 Commerce 里只会越走越窄。

## API V3：不是版本升级，是吞吐重写

如果 Central Order Service 是新职责，那 API V3 就是新发动机。原文一句带过：

> The rollout of API V3 for SAP Order Management Foundation significantly enhances the solution's performance and scalability.

为什么要重写 API？因为承接的负载不一样了。V2 时代 OMF 只接 Commerce 一家，QPS 主要看你网站的下单峰值。换成 Central Order Service 之后，POS 的 TLOG 流（一个百货门店一天几万行）、marketplace 的回单流、退货流、availability push 反向流，全部要从 API 这条管子里灌进去。一个为单一上游设计的 API，撑不住四五个上游同时进来。

V3 的几个明显变化是事件驱动的强化和异步化。今年早些时候 SAP Community 那篇 Asynchronous Request-Reply for Order Management 已经把方向交代过：订单创建从同步阻塞改成异步回调。V3 把这套异步范式做成了默认。

注意几个关键节点：

- Commerce Cloud 走 API V3 写入 Central Order Service，TLOG 走 Sales Transfer & Audit，第三方渠道走同一套 API V3——三条上游汇成一份订单
- 中央订单台账抛事件，Sourcing & Availability、Returns Orchestration、S/4 走事件驱动而非批量轮询
- 入 ERP 之前要过 Type Code 校验（Q3 同时上线的能力），把脏数据挡在 OMF 这一侧

## Availability Push：寻源那块也在反向化

Sourcing and Availability 这一块 Q3 同时上了一个 **Availability Push**。原文描述：

> The Availability Push feature in SAP Order Management for Sourcing and Availability enables proactive delivery of real-time product availability updates to external systems, such as web shops and marketplaces. This ensures that these external systems always have the latest stock information, reducing the risk of order cancellations due to outdated data.

这件事的方向值得品味。过去库存可见性的标准做法是 web shop 主动调 OMS 查库存（pull），高并发情况下要么把 OMS 打爆要么自己 cache 一份过时数据。Availability Push 反过来：OMS 这边库存一变，按 channel / product / 频率配置把 delta 推出去。

这是把库存可用性从"调用关系"改成了"订阅关系"。和 Central Order Service 一起看就明白了——OMF 在 Q3 这一波动作里，整体在朝**事件驱动的中央服务**这个目标走。订单进来是事件，库存出去是事件，退货发起是事件。中间状态全靠 BTP 这一层的事件总线串。

## Type Code 校验：把脏数据挡在 OMS 这一层

Sales Transfer & Audit 那边 Q3 也上了一个 **Type Code 校验**：每个客户配置一份允许的 ERP type code 列表，TLOG 进来的时候先校验，不合法的直接 halt 等审计员处理。

这个动作很小，但配合 Central Order Service 看出意图：把数据质量的边界从"ERP 入口"挪到"OMS 入口"。你 ERP 那边再也不需要写一堆 BAdI 来挡脏数据。OMS 直接拒绝，配置 UI 就在 SAP 标准产品里。这意味着对 SI 来说，**ERP 侧的 type code 客制清理逻辑可以下线了**——前提是你愿意把这层校验搬到 OMS 配置里。

## 几个判断

把上面几件事串起来看：

**第一，Commerce 项目的订单架构需要重新画。** 老项目里"Commerce 是订单源头，OMF 是后台"的画法过时了。如果你正在 2026 年新启动一个 Commerce + OMF 的实施，把订单的权威拷贝放在 OMF 是更稳的选择。

**第二，API V3 不要拖。** 这不是兼容性升级，是一次为多上游、多并发重写的版本。仍然用 V2 的项目在线下接入 POS 之后会很快撞到吞吐墙。新项目直接 V3。

**第三，事件驱动的接入要提前规划 BTP 这一层。** Availability Push 推出来的事件，谁订阅、走哪条总线、怎么 dead-letter，这些不是 OMF 自己解决的问题。BTP Event Mesh / Advanced Event Mesh 那边的容量和拓扑要在项目立项阶段就放进图里。

## 适合谁，先别碰谁

适合现在动的：

- 已经在用 OMF V2 且面临 omnichannel 退货场景的出海零售品牌
- 跨境零售客户，多个 marketplace + 自营独立站 + 海外门店的组合
- 在华外资零售集团，全球总部要求统一订单口径但本地需要落到本地 ERP 的

先别碰的：

- 单一在线渠道、订单量不大的项目——Commerce 自己的 order 管理够用
- ERP 还在 ECC 没上 S/4 的客户
- 完全没有海外业务的纯内贸客户，OMF 的 BTP 数据中心在中国大陆没有部署

OMF 这一年的几次更新都在朝同一个方向走：把订单中枢从 Commerce Cloud 的"内嵌模块"改造成 BTP 上的"独立中央服务"。Central Order Service 是这个方向上最重要的一块拼图。

参考来源：https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/sap-commerce-cloud-q3-25-release-highlights/ba-p/14231991
