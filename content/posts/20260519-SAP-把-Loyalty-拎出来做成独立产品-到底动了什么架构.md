---
title: "SAP 把 Loyalty 拎出来做成独立产品，到底动了什么架构"
date: 2026-05-19T11:40:00+08:00
draft: false
tags: ["SAP CX", "CLM", "Customer Loyalty Management", "技术深度"]
description: "新独立的 SAP Customer Loyalty Management 不是 Emarsys 模块换皮——cloud-based wallet、积分负债会计、规则 DSL 与 ERP 直连，把忠诚度从营销动作升级成业务流。"
source_url: "https://www.sap.cn/topics/innovation-guide/h2"
---

很多人把 SAP 的会员体系等同于 Emarsys 里那块 Loyalty 模块。但 SAP 在 H2 2025 Innovation Guide 里悄悄把这件事重新定义了：SAP Customer Loyalty Management 被拎出来作为一个独立产品，规划 2025 年 11 月 GA。

这不是营销的换皮。Engagement Cloud 里的 Loyalty 是营销自动化的一部分，目标是"把会员留在 campaign 里"；新的 CLM 是把忠诚度做成一条独立的业务流——会员档案、积分负债、兑换核算，要直接连进 Cloud ERP 和 Business Suite。从架构师视角看，这是一次定位上的迁移：忠诚度不再是营销动作的副产品，而是一个有财务凭证、有清算责任的业务对象。

## 从一个具体问题切入：积分到底是谁的负债

做过零售 IT 的人都遇到过这个尴尬：营销团队在 campaign 工具里发了 100 万积分，财务月底关账时问"这 100 万的兑换准备金记在哪个科目"。多数情况下，营销侧只能给一个估算，因为积分发放系统和总账之间没有联动。

这是 SAP 这次产品分拆要解决的核心问题。原文里有一句话信息量很大：

> Integration with SAP Cloud ERP Private solutions and SAP Business Suite will provide visibility into loyalty-related financial metrics... Marketing teams can track promotion performance, redemption rates, and redemption liabilities.

注意 redemption liabilities 这个词。在会计上，已发放未兑换的积分是企业的或有负债，国际会计准则下要按公允价值入账（IFRS 15 把它当作合同负债处理）。绝大多数 CDP 或 marketing automation 工具不会算这个数，因为它们的视角是"营销 KPI"，不是"资产负债表"。把这件事提到产品层面去做，意味着 CLM 必须直接接入总账系统，给 redemption liability 一个会计科目，给每一次兑换一条对应的成本分摊凭证。

## Wallet 模型：为什么不是会员表

原文用了一个很有意思的词——cloud-based wallet。完整说法是 "capture customer data and loyalty engagement in a cloud-based wallet that acts as a loyalty profile for analysis"。

传统忠诚度系统的核心数据结构是会员卡表，一行一个会员，挂积分余额、等级、有效期。Wallet 的隐喻和这个很不一样：每个会员有一个数字钱包，钱包里装的是不同形态的资产——可消费积分、不可消费的状态分（status points）、限定品类券、合作伙伴权益、阶段性挑战进度。

架构上的差异在于：会员卡表是一张事实表，wallet 是一个聚合（aggregate）。聚合意味着这些资产之间有规则约束——比如"消费积分可以兑换 SKU，状态分只能升级等级，合作伙伴券只能用在指定门店"。把这些约束沉到会员档案模型里，避免了下游 Commerce Cloud 在结算时还要自己实现一套复杂的可用性判断逻辑。这是一种典型的 DDD 思路，把领域规则放在最贴近数据的地方，不是放在调用方。

## 积分引擎要解决的四类规则

原文里列了四种支持的忠诚度模型，"points-based programs, segmented offers, personalized rewards, and engagement incentives"，分别对应不同的规则形态：

- **积分制**：交易事件 → 累积逻辑（消费金额×倍率，特定品类×加成）→ 积分入账
- **分段权益**：会员属性 + 行为聚合 → 等级判定 → 享有的特定权益
- **个性化奖励**：用户画像 + 实时上下文 → 触发条件 → 定向发放奖励
- **互动激励**：非交易行为（评论、签到、内容创作）→ 行为评分 → 软性回报

真正难的是把这四类规则跑在同一个引擎上。零售场景里最常见的"踩坑"是：上线时只考虑了积分制，半年后业务要做付费会员（paid membership）就发现规则引擎扛不住，最后被迫拆出第二套系统。一个像样的 CLM 在选型时第一个问题就该问：你们的规则 DSL 能不能用同一套语法描述这四种模型？

简化一下规则定义的样子，大致是这样：

```yaml
# 一条典型的积分累积规则
rule "double_points_weekend_premium":
  when:
    event.type == "ORDER_COMPLETED"
    member.tier in ["GOLD", "PLATINUM"]
    event.timestamp.dayOfWeek in [SAT, SUN]
  then:
    points.credit:
      amount: event.netAmount * 2
      bucket: "redeemable"
      expiry: P12M
      liability_account: "2185-LOYALTY-LIAB"
      cost_center: "MKT-LOYALTY-2026"
```

注意最后两行。liability_account 和 cost_center 是把营销规则直接打在会计维度上的关键——这条规则触发的瞬间，财务侧就能知道增加了多少负债、归在哪个成本中心。这才是和 Cloud ERP 集成的真实含义，不是简单地把数据同步过去。

## 整体架构

![SAP CLM 架构图](http://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLXXuemJKwTxhbE1icOPsic0njTgNe33ReVVic3HibiaQIxPCK0aibLrDujD8tEia0v5NFbicB6XZOOg1deAZbYxpNwARVksRkUfAMSpB2Q/0?from=appmsg)

*SAP CLM 在 CX 套件中的位置：从触点层接收事件，引擎层处理规则与档案，财务层承接负债与成本*

这张图回答了一个常被问到的问题——CLM 和 Engagement Cloud 是什么关系？答案是：Engagement Cloud 是触点层和编排层，负责"什么时候、用什么内容、在什么渠道"和会员沟通；CLM 是引擎层，负责"会员目前的状态、规则触发后该发什么、负债怎么记"。

这种切分有一个明显好处：换前端不影响后端。哪天客户决定 storefront 不用 Commerce Cloud 了，改用自研的 Composable 架构，CLM 这条流不需要重写——只要新前端能往事件总线里发标准的 ORDER_COMPLETED、MEMBER_REGISTERED 事件，规则引擎就能继续跑。

## 和竞品的差异在哪

这两年独立的 loyalty 平台不少——Open Loyalty、Antavo、Annex Cloud、Yotpo，都打 API-first 和 headless 的旗号。SAP 这个产品在功能上未必更强，但它有两件事是别人很难做的。

**第一是和 ERP 的天然连通。** 积分负债的会计处理、兑换成本的归集、跨年度的促销 ROI 分析，这些事交给纯营销系出身的 loyalty 平台都很别扭，要么靠 connector 来回搬数据，要么靠 BI 工具事后拼接。SAP 把它放在自己的 ERP 体内，财务月结时不用额外做对账。

**第二是合作伙伴联盟（partner-coalition）的统一管理。** 原文专门提了一句"manage partner-coalition programs"。多品牌联合积分（航司+酒店+租车这种）在国内零售也越来越常见，难点不在技术，而在结算——A 品牌发的积分被会员在 B 品牌兑了，钱怎么走？跨品牌的积分清算需要一个中性的、可审计的引擎，而 SAP 在这件事上有现成的基础（账期、跨公司过账、税务）。

## 这个产品适合谁，不适合谁

**适合的场景：**

- 已经在用 SAP Cloud ERP 或 S/4HANA 的零售、消费品、航旅企业，会员体系需要和财务深度联动
- 有多品牌、多 BU 共享会员的复杂结构，需要跨主体清算
- 已经部署了 Commerce Cloud 或 Engagement Cloud，希望把 loyalty 从营销工具升级成业务流

**不要碰的场景：**

- 单纯需要积分商城的初创零售品牌——上 CLM 是杀鸡用牛刀，Shopify 上一个轻量插件可能更合适
- 后端 ERP 是 Oracle、Workday 或自研的客户——失去了和 SAP ERP 集成的核心价值，集成成本比独立买一个 Antavo 还高
- 业务规则极度定制、要求会员侧实时风控（反羊毛党）的场景——CLM 的规则引擎面向通用模型，极端定制场景仍要自研补一层

## 实施时要踩的几个坑

**负债科目的设计要前置。** 积分一旦上线就开始累积负债，等运营两三年再来补会计科目，会发现要做大量的历史调账。前期做需求时让财务、税务部门进来。

**积分有效期不是一个简单字段。** 滑动窗口（最后一次活动后 N 个月失效）、固定窗口（每年 12 月底清零）、混合（不同积分桶不同规则）三种策略对引擎压力差别很大。要先和业务对清楚，不要默认选最灵活的那种。

**2026 年 11 月 GA 不等于成熟。** SAP 的 1.0 版本通常要等到第二年下半年才在生产环境跑顺。如果现在就是关键项目的关键依赖，建议要么走 RISE 早期采用计划拿支持承诺，要么准备好用 Engagement Cloud 里的 loyalty 模块过渡。

把忠诚度做成独立产品并直接连进 ERP，这个方向 SAP 走得不算早，但走得比独立 loyalty 平台都更深。值得花时间观察的不是它的功能列表，而是它的会计模型、规则 DSL 和事件契约——这三件事决定了它最终能不能在企业级场景里站住脚。

参考来源：https://www.sap.cn/topics/innovation-guide/h2
