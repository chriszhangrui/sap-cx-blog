---
title: "ExxonMobil 当众建议 SAP 客户：AI 先靠后站"
date: 2026-05-22T10:15:00+08:00
draft: false
tags: ["SAP CX", "Sapphire 2026", "Joule", "AI Agent", "Clean Core", "Autonomous Enterprise"]
description: "Sapphire 2026 第二天，SAP 把 PPT 撤了搬来一张沙发。Lockheed、阿根廷机场、ExxonMobil、Levi's 四家 CIO 把自治企业从概念拽到了地上。"
source_url: "https://itwire.com/business-it-news/data/sap-sapphire-day-2-the-customer-couch-where-the-autonomous-enterprise-stopped-being-a-slide"
---

所有人都在比谁的 AI Agent 多。Levi's 把 1,000 个 Agent 摆上 Sapphire 2026 的舞台，Christian Klein 把 200 个专属 Agent + 50 个 Joule Assistant 写进了 keynote 的第一屏，每家 SI 合作伙伴都在朋友圈晒"我们也部署了 N 个 Agent"。然后在 Orlando 第二天的客户主旨会场，ExxonMobil 的副总裁 Bill Keillor 上台，说了一句让全场安静的话：我们公司确实在做一些 AI，但我故意把它放在边上，等到数据底座、clean core、上云这些事干完之前，AI 都先靠后站。一个 150 年历史、全球能源巨头的 IT 二把手，在 SAP 自家最大的 AI 嘉年华上，公开建议大家不要急着上 AI。这事比任何 Agent 数量都值得琢磨。

先把 Sapphire 2026 第二天那场客户主旨会的设置说清楚。SAP 的 CCO Thomas Saueressig 和负责美洲区客户成功的 Jan Gilg 把 PPT 撤了，搬来一张沙发。请上来四个客户：**Lockheed Martin**、**Aeropuertos Argentina**、**ExxonMobil**、**Levi Strauss**。没有动画图表、没有花哨的产品 reels，就是四个真在干事的 CIO 坐下来聊。这种格式本身就在传递信息：自治企业不再只是 SAP 自己讲的故事，而是客户带着数字来背书。

## 阿根廷机场用一个月跑出 MVP，背后藏着 SAP 的销售逻辑

阿根廷机场的 CIO Gustavo Sabato 讲的故事最有画面感。阿根廷一年下雪八个机场停摆，影响 14,000 个航班、140 万旅客，原来的处理流程靠人工调度、流程零碎、化学除冰剂用得爽。Sabato 团队和 SAP 一起做了个叫 SNOW 的 Agent（Smart Network Operating Winter，工程师的恶趣味命名），把气象数据、跑道传感器、维护流程、塔台通讯全揉到一个编排层里。SAP 在 2 月找上门，3 月做出 MVP，Sapphire 结束两周后开始上线两个机场，明年冬天前再铺六个。

这速度对一个传统印象里"上 SAP 至少 18 个月"的客户来说，反差有点大。Sabato 自己讲了原因：2023 年他们已经把 SAP R3 升到了 S/4HANA，并且把客制化都搬到 BTP 上做侧扩展。他用一句话总结："如果你有 clean core，一切都会更快更容易。"这句话在那一天的舞台上被四个客户用各自的版本重复了一遍。Lockheed 的 Maria Demaree 说"AI 必须嵌进基础里，由流程负责人持有，不是 CIO 临时贴上去就能 scale"；Levi's 的 Jason Gowans 用一句更狠的——"标准化和敏捷不是对立的，标准化是敏捷的前提"。

![Sapphire 2026 customer couch](https://media.itwire.com/2026/05/20/sap-sapphire-day-2-the-customer-couch-where-the-autonomous-enterprise-stopped-being-a-slide-inline-1-a5b3b0374d1c.png)

为什么这事 SAP 一定要在台上讲清楚？因为它解决了 SAP 销售口最头疼的一个问题：客户问"我做完 S/4 还要做 BTP，做完 BTP 才能跑 AI Agent，这是不是又要花我两年？" 现在 SAP 用一个客户的真实时间线告诉所有人——已经踩过 clean core 那一脚的客户，AI Agent 上线就是按月算的；没踩过的客户，先回去把那一脚踩好。这是一个非常聪明的话术设计：把 clean core 从一个"技术债务"重新包装成"AI 准入门票"。

## ExxonMobil 那段反潮流，把 CX 项目的真实困境讲透了

ExxonMobil 的 Keillor 是当天唯一一个公开表态"AI 现在还要等等"的客户。他原话："数据是被困住的资产。我们想把它释放出来。如果我们没把这个底座搞对，未来会一直为此付代价。"他的逻辑很简单：现在做的所有 AI 投入，如果数据底座没整理好、流程没标准化、ECC 还没上 RISE，等下一波 AI 能力来的时候，要么得推倒重来、要么就只能在烂数据上跑一个看起来很美的 demo。所以 Exxon 的策略是"先把基础工程做完一遍，下一波能力进来当成一个配置项接住"。

这话对做 SAP CX 项目的客户和顾问都太熟悉了。我接触过的客户里，跨国零售、跨境电商、出海制造，几乎每一个都遇到过同一个困境：业务部门看了 Joule for Sales、Joule for Service 的演示之后激动得不行，回头让 IT 部门一个月内上线，然后发现 Sales Cloud 的客户主数据有 30% 重复、Service Cloud 的工单分类一团乱、Commerce Cloud 的产品目录在三套系统之间漂——这种状态下硬上 AI Agent，结果就是 Demaree 说的"tack on"——临时贴上去的，跑不起来。

![Levi's at SAP Sapphire 2026](https://media.itwire.com/2026/05/20/sap-sapphire-day-2-the-customer-couch-where-the-autonomous-enterprise-stopped-being-a-slide-inline-2-00764f971907.png)

Keillor 的另一个细节也值得拿来当案例。Exxon 现在在做的事情是"重建运营模型"——更简单的组织结构、更快的决策路径、更干净的数据。注意顺序：组织、流程、数据，AI 是排在第四位的。这个排序在中国出海企业里其实是反直觉的，因为大部分中国总部的逻辑是"先上工具，业务部门用着用着自己就会调整流程"。Exxon 直接告诉你这条路在大型企业行不通，工具会被流程的混乱反噬。Keillor 给了三条规矩：执行层对齐的清晰战略、覆盖每周几千个微决策的硬治理、把项目当 win-win 而不是工单来做的合作伙伴。这三条对中国制造企业海外分销网络上 SAP CX 同样成立。

## Levi's 把 1,000 个 Agent 摆上台，但真正的狠活在前面

Levi's 的 Gowans 那段是当天最有"flex"感的：1,000+ AI Agent 已部署、4,000 名员工动手培训、9 个独立 ERP 实例正在合并成一个共享的数据底座、S/4HANA 全球 rollout 明年完工。最让我留意的不是 Agent 数量，而是他给的一个具体业务数字——批发订单处理。原来 80% 自动化，剩下 20% 是中小零售商发来的 PDF、邮件、Excel 表，老流程一单 2 到 5 天，现在用 Agent 跑在 SAP 上面，一单 20 到 30 分钟。这不是 10% 提效，这是数量级的飞跃，并且打到的是过去 SAP 一直没碰好的尾部长尾客户。

但请注意：Levi's 是先把 9 个 ERP 砍到 1 个共享数据底座，才有资格上 1,000 个 Agent。Gowans 那句"标准化是敏捷的前提"不是漂亮话，是项目实施的硬序。中国出海品牌做 Sales Cloud V2 落地、做 Engagement Cloud 全球营销自动化、做 Commerce Cloud 多市场扩张的项目里，最容易踩的坑就是反过来——先上花哨的 AI 功能，后回头补数据治理。然后两年后发现，钱花了，效果没出来，因为 Agent 在烂数据上跑出来的全是垃圾。

## SAP 自己也在用 Agent 解决迁移这个老大难

客户沙发结束后，SAP 的 Soeren Ruder 演示了新的迁移和现代化助手——七个专属 Agent 跑在 S/4HANA 迁移的各个项目阶段：系统分析助手扫描模块和客制化代码、数据管理助手找不一致并提建议、自定义代码助手把客制化语义化映射到标准、配置助手按最佳实践预配 S/4HANA、测试管理助手对接 Signavio、上线助手做并行多工厂多市场推进、项目管理助手提前识别风险。Ruder 的卖点是"原来要几周准备的事，现在可以在一个周末搞定"。

这个动作其实比舞台上的所有 keynote 都重要。SAP 把自己最痛、客户最怕的"迁移本身"用 AI 解决，等于是在告诉那些还没上 RISE、还没上 BDC、还卡在 ECC 上的客户："你之前不上的理由——太贵、太慢、太痛苦——我现在用 AI 给你解决了。"对中国出海客户来说，这个工具集落地之后会直接影响那些跨国总部要做全球 SAP 系统统一的项目时间表。

## 那这场客户沙发到底说了什么

第一，SAP 不再单方面推自治企业的概念了，它把麦克风递给了四个不同行业的客户去验证。第二，clean core 这个原本是 SAP 内部话术的词，现在被客户用各自的语言重复了四遍——这是话术成功外溢的标志，意味着接下来 12 个月所有 SAP 项目里这个词会被反复提到。第三，ExxonMobil 那段"AI 暂缓"是这场会最有勇气的发言，也是对所有"急着上 Agent"的客户最值得拿出来对照的样本。第四，Levi's 1,000 个 Agent 的数字游戏会在接下来一年蔓延，但真正决定项目成败的是那个数字之前——你的数据、流程、组织有没有准备好。

Saueressig 把麦克风交回去之前留了一句话："技术也许加速了我们能达成的事，但赋予它意义的还是我们的人性。"然后 keynote 收尾视频里冒出最后一句话："未来不是 AI 写的，是我们写的。"一家用了两天时间论证自治企业的厂商，最后选择把这两句话留在台上，本身就值得读一读。

参考来源：https://itwire.com/business-it-news/data/sap-sapphire-day-2-the-customer-couch-where-the-autonomous-enterprise-stopped-being-a-slide
