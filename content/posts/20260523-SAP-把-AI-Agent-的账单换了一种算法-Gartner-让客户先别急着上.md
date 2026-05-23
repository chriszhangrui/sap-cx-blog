---
title: "SAP 把 AI Agent 的账单换了一种算法，Gartner 让客户先别急着上"
date: 2026-05-23T22:00:00+08:00
draft: false
tags: ["SAP CX", "SAP Joule", "AI Agent", "Autonomous Enterprise", "Gartner", "Pricing"]
description: "Sapphire 2026 之后，SAP 把 AI 计费从按用户改成按 action。Gartner 5/19 简报点了三个洞：action 没定义、转换系数 SAP 留权调、Joule Studio 计量未公开。"
source_url: "https://www.theregister.com/saas/2026/05/19/sap-customers-warned-ai-agents-could-put-costs-on-autopilot/5242646"
---

想象一下这样一个画面。某个跨境零售品牌的市场总监周一早晨打开 SAP Engagement Cloud（原 Emarsys），跟里面的 Joule Assistant 说了一句"帮我看看上周欧洲站的复购率掉在哪个客群"。Assistant 调度后台几个 Agent，拉数据、跑分群、生成洞察、推一段建议给营销自动化模块——一气呵成。她低头喝了口咖啡，活儿干完了。

这个体验很爽，问题在月底。

那一句"看看复购率掉在哪个客群"，背后到底触发了多少次 action？拉数据算一次还是十次，调一次模型算一次还是按 token 切片算？她不知道，IT 同事也不知道，因为 SAP 现在还没把 action 这个词的边界画清楚。

5 月 19 日，The Register 的 Lindsay Clark 发了一篇报道，引用 Gartner 一份题为《First Take: SAP Moves to Higher-Value-Based AI Pricing, but Potential Cautions Remain》的研究简报。Gartner 高级首席分析师 Victoria Rowan 在文里直接把话挑明了：SAP 改了游戏规则，但游戏规则的关键变量还没公开。

## 改了什么：从按用户算钱，变成按动作算钱

SAP 在 Sapphire 2026 上正式推出了 Autonomous Enterprise，配套的不是单单 50+ 个 Joule Assistant 和 200+ 个 Agent，还有一套新的商业模式：不再按"有多少个被授权的用户"收钱，而是按"Agent 完成了多少个 action 产生了多少价值"收钱。

这个转变本身是合理的。AI Agent 没有"用户"概念，一个营销总监问一个问题，可能背后就是十几个 Agent 在并发跑活，跟传统 SaaS 按席位计费的逻辑天然对不上。SAP 引入了一个叫 AI Unit 的统一计量单位——你买一定量的 AI Unit，调用任何一个 SAP Premium AI 服务都按各自的转换系数扣减。

配套还有一个叫 Autonomous Domain Blueprints 的工具，号称能给客户一个 T-shirt size 级别的成本估算（small/medium/large 那种）。SAP 跟 The Register 确认，AI Unit 的购买量是按"一个自治领域里预期的 agent action 数"来估的。

- 旧模式：你买多少个用户授权，就付多少钱，无论这些用户用得多狠
- 新模式：你买多少个 AI Unit，Agent 跑活时按 action 扣，扣完续买
- 表面像云服务的弹性计费，实际像出租车的计价器

## Gartner 在担心什么

Rowan 这份报告的核心担忧不是"按动作收费"这个方向，而是三个还没填上的洞。

**第一，action 怎么定义**。Gartner 的原话翻出来是这样：取决于 SAP 怎么定义 action，产生计费的事件数有可能很快就螺旋式上升。如果 SAP 还在用户超出合同承诺量之后再按更高的单价收 AI Unit，或者 AI Agent 触发了 Digital Access 许可证，那情况会更糟。更关键的是，一个动作执行下来给客户带来的价值，可能跟 SAP 给这个动作定的价根本对不上。

> 市场总监问一句"看看上周复购率"，可能算 1 个 action，也可能因为后台拉数据、跑分析、调模型、写回数据库这一连串拆成 7 个 action。两者的账单可以差出一个数量级。

**第二，AI Unit 的转换系数 SAP 自己留了调整权**。Gartner 在报告里指出，客户买的 AI Unit 会按转换比换成实际消耗的 SAP Premium AI 服务的计量单位。但合同条款给了 SAP 在合同期内调整这些转换系数的权利——这意味着今天买的额度，明年用起来可能"更不耐用"了。SAP 发言人对此的回应是，转换系数变更只在续约时生效，"具体由 AI Units 订单表上的条款决定"。

**第三，自建 Agent 的运行成本完全不透明**。Joule Studio 是 SAP 给客户和合作伙伴用的 Agent 开发平台，理论上你可以在里面拼装自己的业务 Agent。但 SAP 发言人确认，Joule Studio 的运行时计量规则目前还没公开。Gartner 直接说：只要这个状态还在，预测和控制运行成本就会很难。

## 为什么这事跟做客户体验的人特别相关

很多人看到这条新闻第一反应是"这是 ERP 的事"，其实不然。SAP 公布的 50+ Joule Assistant 五大领域里，customer experience 是单独一个域。Sales Cloud V2、Service Cloud V2、Commerce Cloud、SAP Engagement Cloud (Emarsys)，几乎每个产品都已经摆上 Joule Assistant 或者专门的 Agent。

这些场景天然是"高频高量"——一个营销活动可能触发上百万次个性化推荐，一个客服中心一天上万通对话，一个电商促销夜里几十万次实时分群。每一次背后都可能是若干个 Agent action 在跑。如果 action 的颗粒度切得碎，账单膨胀速度会比 ERP 财务模块快得多。

中国出海品牌的处境更尴尬一点。一家做欧洲市场的快消品牌，欧洲数据中心、欧洲合规、跨境合同、欧元结算——AI Unit 的转换系数浮动一次，叠加汇率波动和合同周期错配，财务部门预算表里多出来的那栏数字解释起来会很费力。

## 现在能做的几件事

Gartner 在报告末尾给了几条具体建议，整理一下，给中国出海客户的版本是这样：

- **翻一下现有的 SAP 合同**，找一下 S/4HANA Cloud 或者 CX 各产品的"价格保护条款"——通常是约定续约时的涨幅上限。这条之前可能没人特别在意，今后是关键。
- **从 SAP Trust Center 拿一份当前的 AI Services List**，把每个 Premium AI 服务的当前转换系数存档。这是一个基线，未来如果 SAP 调系数，至少有据可查。
- **跟 SAP 商务团队明确谈 action 定义**。在合同里争取把 action 的颗粒度文字化，至少在最常用的几个 Joule Assistant 上。
- **POC 阶段就要做 action 计数**。不要光看功能 demo，拉一份"一个典型业务流跑下来消耗了多少 AI Unit"的数据，把这个换算成年化成本，再决定是不是上规模铺开。

还有一个被忽略的点：Digital Access 许可证。SAP 老 ERP 客户都熟悉这个东西——非人类系统调用 SAP 接口要单独算钱。如果 AI Agent 大规模触发 Digital Access，那是另一笔账，跟 AI Unit 不是一回事。Gartner 在报告里特意点了这个，说明不是危言耸听。

## 回到那个市场总监的周一早晨

如果 SAP 在 2026 下半年真的把 Autonomous Enterprise 落地了，那个市场总监的周一早晨会变成什么样？

最理想的版本：她还是那一句"帮我看看上周欧洲站的复购率掉在哪个客群"，活儿照样一气呵成，但她的屏幕右上角会有个小角标——"本次任务消耗 0.3 AI Unit"。她对成本心里有数，月底财务也对得上账。

不太理想的版本：所有体验都很丝滑，直到月末预算超支 30%，财务总监跑来质问 IT，IT 跑去问 SAP，SAP 给一份明细，明细解释为什么这次升级了某个 Agent 之后单 action 变贵了。

两种版本之间的差距，恰好就在 Gartner 这份报告点的那几个洞上。能不能填上，要看 2026 下半年 SAP 公布的细则。

体验是 SAP 的卖点，账单是客户的隐患。上线 Joule Assistant 之前，先把计费规则掰扯清楚，比演示视频里那段丝滑动作要重要得多。

参考来源：https://www.theregister.com/saas/2026/05/19/sap-customers-warned-ai-agents-could-put-costs-on-autopilot/5242646
