---
title: "SAP没自己做PIM，却给Inriver盖了个章，这步棋怎么看"
date: 2026-06-03T12:00:00+08:00
draft: false
tags: ["SAP CX", "SAP Commerce Cloud", "SAP BTP", "PIM", "Inriver", "S/4HANA"]
description: "Inriver通过SAP BTP官方认证打入S/4HANA Cloud + Commerce Cloud全栈，不需要客户自建iFlow。Commerce项目里那块产品数据老缺口，SAP这次决定让生态来补。"
source_url: "https://www.globenewswire.com/news-release/2026/05/28/3302933/0/en/sap-certified-s-4hana-cloud-connector-by-inriver-is-certified-by-sap-as-built-with-sap-business-technology-platform.html"
---

数据孤岛这个老问题，被翻译成各种新词讲了快十年。CDP、Knowledge Graph、Data Fabric，再到现在的 Data Products，名字换了一轮又一轮。但**在 SAP Commerce Cloud 项目里，最早暴露这个问题的，从来不是客户数据，而是商品数据。**

5 月 28 日 Inriver 发的这条 SAP 认证消息，不少人扫一眼就划过去了。它不带"AI"、不带"Agent"，标题里全是缩写。但放在 SAP CX 这条产品线的演进里看，这是过去半年最值得记一笔的产品集成动作之一。

**先说事实。** Inriver 这家做企业级 PIM（产品信息管理）的瑞典公司，宣布它的 S/4HANA Cloud Connector 通过了 SAP 集成与认证中心（SAP ICC）认证，认证类别是 "Built with SAP BTP"，并通过 BTP 直接对接 S/4HANA Public Cloud。再加上之前已经认证的 Commerce Cloud Connector，整条产品数据链路第一次在 SAP 体系里被官方认可成一条端到端通路。

关键的一句话藏在新闻稿里：**认证集成通过 SAP BTP 实现 —— 不需要客户自己写定制开发，也不需要自建 iFlow。** 做过 Commerce 项目的人看到这句应该会停一下。

**为什么停一下？** 因为商品数据这块，过去十几年在 SAP 客户的电商项目里几乎是个空白带。早期的 hybris PCM 模块淡出之后，SAP 自己一直没有重新发一个完整 PIM 产品。但凡商品 SKU 多一点、属性维度多一点、上架渠道多一点的项目，PIM 都得另外选型——要么客户自建，要么接 Riversand、Stibo、Akeneo、Inriver 这些第三方。

而"接"这件事，过去基本上等于"在 BTP 上自己搭一堆 iFlow"。每一个属性映射、每一个媒体资源同步、每一次价格变更通知，都是一段集成代码，每个项目重复造一遍轮子。SI 报价里那张密密麻麻的接口清单，大半是这个东西。

现在 SAP 把"通过 BTP 直连 S/4HANA Cloud + Commerce Cloud"这条路，给 Inriver 盖了个官方认证章。这意味着对走 SAP Public Cloud 全栈的客户来说，PIM 这块第一次有了一个准官方的"组合拳"路径。

**更值得拎出来看的是 SAP 的表态。** 新闻稿里直接引述了 SAP CX & CRM 首席专家 Alex Timlin 的话——他说："客户数据和交易数据一直是焦点，但和 Inriver 的合作让我们看到，**产品数据是企业的核心资产**。"

这句话信息量很大。Sapphire 2026 上 SAP 把所有 AI 故事都叠在 BDC（Business Data Cloud）上面，谈的是怎么把客户 360 视图、交易上下文喂给 Joule。Product data 这一维度，主舞台上几乎没怎么单独讲过。Inriver 这次借着官方认证，让 SAP 出面给 PIM 在那张图里补了一个位置。

Alex 说的"右产品在右地点右时间"，听上去像零售老话术。但放在 Agentic Commerce 的语境里——一个 AI Agent 要替顾客挑选商品、要回答配置问题、要在 B2B 场景下做技术参数比对——产品数据的颗粒度、准确度、时效性，全是底层燃料。语料不行，再聪明的 Agent 也只能编。

**这步棋下给谁看？** 我的判断是三类人。

- **第一类：S/4HANA Public Cloud + Commerce Cloud 双栈客户。** 这是 SAP CX 现在主推的"参考架构"组合，也是这次认证最直接的目标群。出海制造企业、跨境品牌商、做全球分销的中国大厂，如果两边都在 SAP 体系内，PIM 选型清单上 Inriver 现在变成了那个最容易过架构评审的选项。
- **第二类：SI 合作伙伴。** 对做 SAP CX 集成的伙伴来说，这次认证既是机会也是压力。机会是项目里 PIM 这块再也不用从零评估、教育客户；压力是过去那张"PIM 集成"工时表，可能要被砍掉一大块。技术活没了，活就得往咨询和数据治理上走。
- **第三类：还在观望 Composable Storefront 迁移的客户。** 这次认证给的是一条新链路：S/4 Public Cloud 当系统记录、Inriver 当产品中枢、Commerce Cloud 当电商前台、全部跑在 BTP 上。这是一条比传统 ECC + Hybris 更"云原生"的路径。

**但也别把这事吹得太大。** 认证不等于成熟，新闻稿里没披露任何参考客户的部署案例，也没讲性能边界。Inriver 在中国市场的本地服务能力依然薄弱，这对中国出海企业是个现实约束——总部用得很顺，到了亚太分子公司可能要重新评估实施伙伴。

而且这件事的潜台词其实有点微妙。SAP 没有自己做 PIM，反而把一个第三方厂商捧到"官方认证"位置——这背后多少能读出 SAP 在产品广度上的取舍：核心做 ERP、做 Commerce、做 Engagement，PIM 这种垂直能力让生态来补，自己不再下场。这跟 Salesforce 收购 Commerce Cloud（前 Demandware）然后又自己收购 Vlocity 的路数完全不一样。

**对中国出海客户的实际影响是什么？**

如果你是一家产品 SKU 上万、上几十个国家电商站点的中国制造商或品牌方，正在评估 SAP Public Cloud 全栈替代当前杂牌系统，这次认证降低了一个集成风险点。原本要花三个月对接 PIM 的事，现在大概率能压缩成几周配置加几周数据迁移。

如果你已经在用 Inriver、但 SAP 那边还在 ECC 或者 S/4 Private Cloud，那这次认证不直接覆盖你的场景。需要等 SAP 后续把"Built with SAP BTP"的认证范围扩展到 Private Cloud 部署。

如果你正在把电商前台从 Hybris 老 Storefront 迁到 Composable Storefront，这次认证给你提供了一个把 PIM 一并重构的窗口期。前台动了，后台数据源不动，半年后你会发现下一轮架构债又来了。

数据孤岛这个老问题，从来不会被一次认证解决。但 SAP CX 这条线最近一年在生态对接上的动作，比过去三年加起来都多。这个方向，至少是对的。

参考来源：https://www.globenewswire.com/news-release/2026/05/28/3302933/0/en/sap-certified-s-4hana-cloud-connector-by-inriver-is-certified-by-sap-as-built-with-sap-business-technology-platform.html
