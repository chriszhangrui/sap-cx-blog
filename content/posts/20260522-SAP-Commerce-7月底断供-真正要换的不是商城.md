---
title: "SAP Commerce 7月底断供，真正要换的不是商城"
date: 2026-05-22T18:10:00+08:00
draft: false
tags: ["SAP CX", "SAP Commerce Cloud", "Hybris", "PIM", "出海电商"]
description: "2026年7月31日 SAP Commerce on-premise 主流维护结束。跑了十年的 Hybris 客户，真正要换的不是商城前端，是埋在底下的产品数据模型。Phase 2 必须早于 Phase 4。"
source_url: "https://apollon.de/en/artikel/sap-hybris-end-of-life-migration-playbook-en"
---

先说一个画面。

某中国家电品牌在欧洲卖了八年，主力市场德国法国意大利。他们的电商架构师 Mark 周一早上九点打开邮件，第一封是德国 IT 总监转过来的 SAP 公告：**2026 年 7 月 31 日，SAP Commerce on-premise 主流维护正式结束**。第二封是德国子公司的运营总监，问能不能在那之前把波兰站和瑞典站再上线。第三封是总部 CTO，问"我们 Hybris 还能撑几年"。

Mark 看着这三封邮件，想到的不是 Hybris 本身要不要换。他想的是：过去八年，德国团队把 PIM 逻辑、价格规则、变体编排、ERP 适配器全都焊在了 Hybris 的 PCM 层里。现在如果只换一个商城前端，所有这些复杂度会原样搬到下一个系统。再过五年，又是同一个剧本。

这就是 2026 年所有 Hybris 客户的共同处境。德国 PIM 厂商 apollon 上周发了一篇行业 playbook，把这件事说得很直白：**真正的难题不是商城，是数据模型**。

## 7 月 31 日到底结束什么，又没结束什么

先把事实摆清楚，因为这件事在客户那边经常被搞混。

2026 年 7 月 31 日结束的是 **SAP Commerce on-premise 2205 版本**的主流维护，这是 on-prem 线的最后一个版本，2022 年 5 月发布。这之后会停的事情包括：

- 安全补丁不再发了
- Bug 修复和合规更新停止
- JVM 和第三方库的认证矩阵不再维护
- 新功能从 2022 年起就只进云版了
- 不能在 EoMM 版本上构建新应用（SAP KBA 3242156）

没结束的是 SAP Commerce Cloud 公有云版本——它每季度还在更新。如果客户在 2024 或 2025 年已经做过 lift-and-shift 上云，这次 EoMM 跟你没关系。

这里有个细节值得划重点：SAP 没有给 Commerce 提供类似 ECC 或 S/4HANA 那种延长维护轨道。所谓的 Customer-Specific Maintenance 是个备选项，但它不保证新补丁、不做法律变更、不针对当前 OS 和数据库做重新认证。**它买的是时间，不是安全。**

![SAP Commerce 替换路线图：5 个阶段不能交换顺序](https://apollon.de/wp-content/uploads/2026/05/hero-hybris-migration-2026-05.jpg)

## 为什么 Hybris 客户在 2026 面对的不是商城问题

Hybris 比你想象的更老。1997 年瑞士楚格创立，2013 年 SAP 收购——价格官方没确认过，外界报道是 13 到 15 亿美元。2018 年起品牌正式改成 SAP Commerce Cloud，Hybris 这个名字退役。2022 年 12 月的 2211 release，SAP 把开关全切到云端：从那以后，新功能、安全补丁、创新都只进云版。2026 这个 EoMM 是这条路线的逻辑终点，不是突发事件。

Gartner 那边的标签更直接——Peer Insights 里这个产品现在叫 "SAP Hybris Customer Engagement and Commerce (Legacy)"。分析师不会随便给"Legacy"这个词。

但如果你真在跑一个 Hybris on-prem 系统，你知道还有另一面。八到十二年的运维之后，一份典型的 B2B Hybris 部署里可能有：几百个自定义扩展、和促销逻辑搅在一起的 PCM 层、跟 ERP 紧耦合的存储模板、对接 OMS / WMS / DAM 的接口，可能还有第二套主数据系统。这不是"一个商城"，这是一个以 Hybris 为中心长出来的数据生态。

2026 年的战略陷阱就在这里。如果把 replatforming 当成纯粹的商城问题——"哪家平台能替代我的 Hybris"——你就把所有复杂度照搬到了下一个系统。

> 真正的问题不是商城。真正的问题是数据模型。

## PCM 这个隐形的天花板

Hybris 当年有个优势，后来变成了刹车。Product Content Management 深度内嵌在 Hybris 栈里——分类、多层级 taxonomy、本地化属性、变体逻辑、资产关联，全部在一个数据库、一个领域模型、用 Hybris cockpit 维护。这在 2013 年的架构标准下是先进的。在 2026 年的现实里，这是反过来的。

当产品数据住在商城平台里，三个战略决策被绑在了同一个迁移选择上：换维护界面、换数据分发、换前端。Replatforming 不再是"换个商城"，而是**边开车边造车，而且引擎还焊在方向盘上**。

解法很简单但结论很重要。在你动一行商城代码之前，先把产品数据从 Hybris 里取出来，放到一个独立的数据层——一个现代 PIM。一旦产品数据独立于商城，你就拿到三件事：商城选型不再有数据风险；新旧系统可以并行跑同一个数据源，迁移是渐进式而不是一刀切；你做的这笔投资是能留下的——五年后再换一次商城，PIM 还在。

这个理念叫"PIM as decoupling layer"，apollon 这套方法论无论目标商城是 Shopware、Spryker、commercetools、Elastic Path，甚至是继续留在 SAP Commerce Cloud，都成立。它不是一个 vendor question，是一个 architecture question。

## 五个阶段的 playbook，顺序不能换

apollon 给出的迁移流程分五步，关键是 **Phase 2 必须在 Phase 4 之前**，这是整篇文章里最重要的一句。

**Phase 1 — 盘点**。在迁移之前先把现状画出来。三件事并行：产品数据模型审计（用了哪些分类体系、属性类型、变体维度）、接口图（ERP / OMS / WMS / DAM 进出哪些数据）、自定义代码审计（哪些是标准、哪些是 bespoke、哪些是"只有那个已经离职的同事知道")。在 apollon 自己的 PCM 审计里，1500+ 个属性是常态，**其中 60–70% 是冗余、过期或重复的**。这阶段一般跑 4 到 8 周。

**Phase 2 — PIM 整合**。把 PCM 从 Hybris 里搬出来，建立成产品数据的单一真相源。数据模型翻译、数据质量中枢、初始数据迁移、然后让 PIM 反过来给老 Hybris 推数据——作为过渡数据源。这一步的精妙在于：从 PIM 上线第一天就开始产生价值，而且不冻结任何商城决策。

**Phase 3 — 解耦数据模型**。所有产品数据消费者（商城、App、Marketplace、B2B 门户、印刷目录、销售配置器）都通过定义清晰的 API 接入。Headless / API-first 作为目标架构，渠道分发逻辑放在分发层，不放在商城里——否则就是下一个 Hybris 堵车现场。

**Phase 4 — 商城选型**。到这里才开始选。因为数据层已经独立，这时候选型才能诚实地比较。Shopware、commercetools、Spryker、Elastic Path、SAP Commerce Cloud 各有各的位置，但选型不再涉及数据模型——那件事 Phase 2 已经做完了。

**Phase 5 — 切换**。并行运营、SEO 迁移、上线前数据签收、上线后 30 天硬化期。一个被 apollon 反复强调的数据点：**SEO 切换没做好的 B2B 制造商，平均损失 6 到 12 个月的自然流量**。这条要进 business case。

## 这件事对中国出海企业意味着什么

SAP Commerce Cloud 是 SAP CX 五朵云里唯一在中国有数据中心的产品。但跑 Hybris on-prem 的客户分布逻辑跟这件事关系不大——很多中国制造企业的欧洲分销网络、跨境品牌的多市场站点、在华跨国公司全球总部的统一商城平台，都可能是 2010 年代中后期上的 Hybris on-prem。

这些客户里有相当一部分，过去几年一直在拖。理由很常见：商城跑得好好的，没有立刻动力换；自定义代码改了一茬又一茬，没人愿意是踩雷的那个；总部 IT 想换云，本地业务怕断货。

7 月 31 日这条线之后，安全补丁断供这件事会以一种很具体的方式回到桌面上——下一次 Java 库爆出 CVE，下一次 OS 升级，下一次 PCI-DSS 审计。这些不是"明年的事"，是季度内会发生的事。

所以站在中国出海客户的位置，一个值得被认真讨论的问题不是"我们要不要换"，而是**"我们换的时候，要不要把这次当成一个修地基的机会"**。如果只是 lift-and-shift 到 SAP Commerce Cloud，技术债会一起搬过去；如果先做 PIM 解耦，再考虑商城选型——哪怕最后还是选 SAP Commerce Cloud——这次投入的复利会延续到下一个十年。

apollon 那篇文章里反复说"Phase 2 before Phase 4 is the single most important strategic statement of this article"。我把这句拿过来转给做这块决策的客户：在你启动 RFP 之前，先问一个问题——产品数据现在住在哪？如果答案是"住在 Hybris 里"，那 RFP 启动得太早了。

## Mark 的周一早上

回到开头那个画面。

如果 Mark 把 7 月 31 日当成一个换商城的 deadline，他这一年会在选型 RFP 上烧掉所有时间，年底带着一个新合同回来，然后用三年时间把 Hybris 的数据复杂度复刻到新系统里。

如果 Mark 把这件事当成一个修地基的窗口，他这一年的第一个项目会是 PIM——不管最后商城选谁。波兰站、瑞典站可以挂在 PIM 后面，先用老 Hybris 上线；商城选型走到 2027 年也不算晚；总部 CTO 问"还能撑几年"的时候，答案不是"撑"而是"我们已经在迁移路径上"。

这两条路差在哪？差在五年后这家公司的电商架构能不能再 replatforming 一次而不伤筋动骨。

7 月 31 日这条线，是个 deadline，也是个分水岭。看你怎么用。

参考来源：https://apollon.de/en/artikel/sap-hybris-end-of-life-migration-playbook-en
