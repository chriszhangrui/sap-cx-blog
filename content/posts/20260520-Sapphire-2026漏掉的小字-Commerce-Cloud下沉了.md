---
title: "Sapphire 2026漏掉的小字：Commerce Cloud下沉了"
date: 2026-05-20T18:08:00+08:00
draft: false
tags: ["SAP CX", "Commerce Cloud", "Mid-Market", "Sapphire 2026", "出海"]
description: "所有人都在讨论SAP的AI叙事，但Innovation Guide第43页那块小字才是改变中国出海生意的真正变量——Commerce Cloud开始下沉到中端市场。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/sap-sapphire-2026-announcements-only/ba-p/14396190"
---

> SAPPHIRE 2026 · 那些不在主keynote里的

在Sapphire 2026的所有发布里，主舞台讲的是Anthropic、200个Agent、1亿欧元生态基金。

但翻到Innovation Guide第43页那块小字——**SAP Commerce Cloud for Mid-Market — Q3 2026**——这一行才是改变中国出海生意的真正变量。

为什么这么说？

## Commerce Cloud以前是大企业的专属品

SAP Commerce Cloud这个产品，过去几年在中国市场的客户画像高度集中——年收入十亿美金以上的跨国制造、出海大牌、全球分销网络的工业品制造商。

原因不复杂。它是个Composable Commerce的重型平台，独立部署，要把它和ERP、CRM、CDP、支付都接起来，集成成本动辄百万级人民币起。一个完整B2B商城项目，做完三年都很正常。年收入5亿以下的出海卖家根本啃不动。

这次Sapphire把Commerce Cloud的产品形态拆成了三个：原版的Composable继续服务大客户，新增了一个Mid-Market版本跑在Cloud ERP上，再补一套Composable Commerce Services让品牌按模块取用购物车、结账等能力。

![Commerce Cloud 三个交付形态](https://community.sap.com/t5/technology-blog-posts-by-sap/sap-sapphire-2026-announcements-only/ba-p/14396190)

真正的变化是**"跑在Cloud ERP上"**这几个字。

## "跑在Cloud ERP上"翻译过来意味着什么

Mid-Market这个版本不是把原来的Commerce Cloud砍掉一半功能再卖便宜——这种简化产品历来不成功。它做了更狠的一步：让Commerce直接寄居在SAP Cloud ERP的同一个平台栈里。

这意味着商品主数据、库存、订单、客户、价格这些核心对象，不需要中间件做异步同步。商城底下就是ERP的同一份真相。

对一个出海中端品牌来说，这件事意义在哪？

- 实施周期从12-18个月的级别，可能压到3-6个月
- 集成成本不再吃掉总预算的60%以上
- ERP升级和Commerce升级不再各跑一个节奏，互相打架
- 数据一致性问题——比如海外站发货后国内ERP库存没扣减——从架构层面消失

这些痛点过去是需要中型出海企业花几年时间一边踩坑一边喂经验给SI才能优化掉的。Mid-Market版本相当于把这些坑提前焊死了。

## 第三块拼图：Composable Commerce Services

如果说Mid-Market是给"想要一站式"的客户准备的，那Composable Commerce Services就是给另一类客户的——已经有自己前端店铺，但需要后端能力的。

这套服务把购物车、结账这些核心交易能力做成了模块化的服务，可以单独取用。配上同期发布的Vercel storefront加速、Adyen/Checkout.com/PayPal嵌入式支付，这条线明显是冲着已经在用Shopify、自研前端、或者Headless架构的卖家去的。

用一个不那么准确但容易理解的类比——SAP在做Commerce领域的"AWS化"。把以前必须整套买的产品，拆成可以按需调用的能力包。

这一步走得不快。Salesforce的Commerce Cloud在Composable Storefront、Headless API这块走了好几年。SAP现在补这一课，是被市场逼出来的。

## 为什么这件事对中国出海卖家是个信号

先说一个不太招人喜欢的事实——SAP CX除了Commerce Cloud之外的所有产品，在中国大陆没有数据中心。Sales Cloud、Service Cloud、Emarsys这些都是欧洲或美国的数据中心。

这意味着只有**做出海生意的中国企业**才是SAP CX真正的客户群——你的目标客户不在中国大陆，数据合规问题就不会卡住你。

这个客户群的画像过去十年其实悄悄变了：

- 一是真正出海做品牌的中端制造商，比如做小家电、户外、宠物用品的，海外DTC站点起来了，年销售1-5亿美金
- 二是跨境电商从亚马逊起家做到一定规模，开始想绕开平台搞自有渠道
- 三是传统外贸B2B工厂，海外分销商越来越多，需要一个全球统一的订单和价格管理

这三类客户过去看Commerce Cloud都觉得"产品好但用不起"。Mid-Market版本如果实施周期、定价、合作伙伴生态做得起来，是真的有可能切到这块市场的。

## 值得提前盯的几个不确定性

话说回来，Q3 2026 GA还有几个月，几件事现在没答案：

- 定价模型——能不能真的下沉到中端，关键看SAP是不是愿意把License价格往下放
- Cloud ERP在中国出海客户的部署形态——是装在RISE的公有云？私有云？还是BTP？这影响海外站点的延迟
- 实施伙伴的生态——大型SI做不了Mid-Market的快进快出，需要新一批中型实施商进来
- 跟Composable原版的功能差距——Mid-Market砍了多少自定义能力，决定了客户能不能往上长

最后这一点最关键。如果Mid-Market砍得太狠，客户长大了之后没有平滑迁移到Composable的路径，那这个产品本质上是个临时方案，不会成为主流。

## 回到那个被忽略的小字

所有人都在讨论SAP的AI叙事——Joule、Anthropic、200个Agent、Autonomous Enterprise。这些是大故事。

但企业软件的真正变化，往往不在主舞台。它在第43页的产品路线图小字里。

Commerce Cloud下沉到中端市场，是SAP第一次在一个CX核心产品上明确说"我要打的客户群从五百个变成五千个"。这个客户群的扩张方向——出海中端品牌——刚好是中国企业过去三年增长最快的那群人。

Q3见。

参考来源：
- https://community.sap.com/t5/technology-blog-posts-by-sap/sap-sapphire-2026-announcements-only/ba-p/14396190
- https://www.sap.com/mena/topics/events/sapphire/innovation-news-guide-2026
