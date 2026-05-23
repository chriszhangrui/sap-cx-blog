---
title: "Composable Storefront 不是换前端，是换地基"
date: 2026-05-23T16:00:00+08:00
draft: false
tags: ["SAP CX", "Commerce", "Composable Storefront", "技术深度"]
description: "JSP Accelerator 2026 年 9 月从 2211 移除，WCMS Cockpit 已经从 latest 拿掉。把它当一次前端换皮做，十八个月后还要再迁一次。"
source_url: "https://www.contentful.com/blog/sap-commerce-storefront-sunset/"
---

问一个具体的问题：手上一套跑了七八年的 SAP Commerce，前端是 JSP Accelerator 模板，内容运营靠 WCMS Cockpit，现在要不要动？

SAP 给的时间表已经压到桌面上了。JSP Accelerator 在 2205 版本就被标记 deprecated，2026 年 9 月从 SAP Commerce Cloud 2211 中正式移除，extended support 到 2028 年截止。WCMS Cockpit 走得更快——它已经从 2211 latest 里拿掉了，没有 extended 这一段缓冲。如果还在跑 on-premise，2205 是最后一个版本，主线维护到 2026 年 7 月就结束。

两条线在 2026 到 2027 年集中收口。这是这两年所有跑 SAP Commerce 的项目都要回答的问题。但真把它当作一次"换前端框架"的工程项目来推，十八个月以后大概率会发现：钱花了，架构没动，下一波数字化需求来的时候还得再迁一次。

## 这次不是换前端，是地基拆了一层

很多人第一反应是：JSP Accelerator 退役，那就上 Composable Storefront（可组合店铺架构，前身 Spartacus）替换掉模板层就行了。这个理解只对一半。

JSP Accelerator 不只是一套模板，它是一种紧耦合的前端形态——每一次页面调整都意味着改 JSP、重新打包、整套应用一起部署。前端逻辑、内容结构、商品展示、营销位全部混在 Java 工程里。它能跑，是因为 WCMS Cockpit 在内容侧补了一层结构化能力，但这两个东西本来就是为同一个时代准备的：单一品牌、单一渠道、内容更新频率以周为单位。

Composable Storefront 完全是另一套形态。它是基于 Angular 的 PWA（Progressive Web App，渐进式网页应用），通过 OCC REST API（Omni Commerce Connect）跟后端 Commerce Core 通信，部署节奏跟后端解耦——前端可以独立发版、独立扩缩容、独立做实验。

所以 SAP 这次动的不是模板引擎，是前端跟后端之间的契约层。从"前端是后端的一部分"变成"前端是后端 API 的一个消费者"。一旦切到这个模式，前端可以是 Spartacus，也可以不是——这才是 Composable 的真正含义。

![Composable Storefront 架构分层](http://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLUco8mCCOQJAVek4hqiavdyuSEczrJZLTbcVuyTRbse0Go4RbiaPOHgx31Rf0wKpFgay1eEgWIwbNnNjeywriak5LXeMdclwshy4M/0?from=appmsg)

▲ 表现层与内容层同步换代，中间 API 与 Commerce Core 保持稳定

## WCMS Cockpit 退场，是更值得关注的暗线

前端换框架的故事讲得多，内容层的迁移反而经常被一笔带过。事实上后者影响更大。

SAP 给出的官方替代是 SmartEdit，一个 in-context 的所见即所得编辑器，覆盖页面拼装、个性化规则、内容版本控制这些基础能力。对单一品牌、单一店面、内容主要靠商品本身撑起来的项目，SmartEdit + Composable Storefront 这套组合够用，而且留在 SAP 生态内意味着支持链路最短。

但 SmartEdit 仍然是跟 Commerce 紧耦合的内容工具——它管的是商品页和着陆页这一层。一旦内容开始有这些诉求，它就开始吃力：

- 多市场、多语言、多品牌的内容需要在不同店面间复用
- B2B 场景下要管大量产品技术文档、规格说明、白皮书
- 需要把同一份内容供给网站、APP、客服 Agent、对话式 AI 多个渠道
- 营销团队希望脱离开发排期独立发版

这时候真正合适的是把内容层完全抽出来，放到一个 headless CMS 上。Contentful 是 SAP 官方背书的合作伙伴之一，但选哪一个不是关键，关键是你认不认可"内容跟商品平台解耦"这件事。

## 三种路径，对应三种业务形态

不是每个项目都要走最远那条路。Valtech 在他们的迁移实践里把客户分成三类，这个分类挺实用：

**第一种：单品牌 B2B，目录驱动。** 一个店面，主要靠商品和价格转化，内容更新不是核心战场。这类项目最务实的路径就是 Spartacus + SmartEdit，留在 SAP 生态内，把废弃组件换掉就行。强行上 headless CMS 是过度设计。

**第二种：多市场 B2B 制造业，内容复杂。** 跨国卖技术零部件、工业品，每个市场要本地化的产品规格、技术文档、合作伙伴专区、行业方案页。Valtech 的案例里有一个全球 B2B 工业制造商，内容更新慢、各市场不一致、完全依赖开发团队，迁到 Contentful 之后内容生产周期缩短超过 60%，campaign 上线时间缩短约 70%。这一类是 headless CMS 的甜蜜点。

**第三种：押注 AI 与个性化的企业。** 已经在规划 AI Agent、对话式商务、深度个性化推荐。这一类对内容的要求是"结构化、可通过 API 取用、跨渠道一致"——AI Agent 要能实时检索和组装内容片段，没有结构化内容层的话，AI 项目会撞墙在内容那一层。这一类客户基本上没得选，必须走 headless。

## 放在出海与跨境项目的语境下

SAP Commerce Cloud 是 SAP 客户体验产品线里少数在中国本地有数据中心的产品，但实际上选它的项目大部分都是跨境出海或在华外企。从这个语境看，Composable 的价值更直接。

出海品牌的典型场景是：一个 Commerce Core 在背后撑住目录、定价、订单、ERP 集成，前端要根据不同市场长出不同的样子——欧洲市场的隐私合规、东南亚的本地支付、中东的 RTL 布局、北美的 PWA 离线体验。如果前端跟后端耦死，每个市场都要做一个分支版本，维护成本会指数级上升。Composable Storefront + 独立内容层这套架构，本质上是把"市场差异"集中到前端和内容层去吸收，让 Core 保持稳定。

B2B 跨国制造业的场景更直接。海外经销商门户、技术资料库、行业解决方案页面，本来就跨多个市场、多种语言。WCMS Cockpit 那种树形目录的内容模型在这种场景下早就到极限了。

## 落地建议

把这几条放在项目立项的第一页：

- **先做内容审计，别先选框架。** 原 WCMS 里有相当一部分内容（经验值约 30%）应该是直接退役而不是迁移。先砍干净，再决定迁哪里。
- **前端切换不要跟 JDK 21、Angular 21、Node.js 22 升级捆在一起做。** 2026 年初 Node 20 EoL、2026 年 2 月 Angular 21、2026 年 8 月 JDK 21 截止——这几条线已经够紧了，再叠加内容层迁移，项目风险会失控。
- **单店面 B2B 项目不要硬上 headless。** Composable 的复杂度不是免费的——API 治理、跨系统监控、前后端独立发版的 CI/CD，这些能力没建好的话，composable 等于把架构债换了个地方放。
- **如果内容是竞争力的一部分，不要折中。**"用 SmartEdit 先过渡一下"是最容易踩的坑。SmartEdit 跟 Commerce 是绑死的，过几年想再剥离会比现在直接上 headless 贵很多。
- **AI Agent 路线图必须在架构决策里出现。** 三年内打算上 Joule、上对话式商务、上检索增强生成的项目，内容层一定要走 headless——结构化内容是 AI 能用上的前提，不是 AI 项目启动后再补的事。

最后回到开头那个问题：旧 Commerce 要不要动。要动，但怎么动取决于你下一个三到五年要打的仗是什么。把它当作一个 EoL 截止日去赶，是把战略问题降级成技术债务问题。这通常是 SAP Commerce 项目最贵的那种省钱方式。

参考来源：https://www.contentful.com/blog/sap-commerce-storefront-sunset/
