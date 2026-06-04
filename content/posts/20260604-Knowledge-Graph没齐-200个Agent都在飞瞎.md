---
title: "Knowledge Graph没齐，200个Agent都在飞瞎"
date: 2026-06-04T14:10:00+08:00
draft: false
tags: ["SAP CX", "SAP Business AI", "Joule", "Knowledge Graph", "BDC", "Autonomous Enterprise"]
description: "Sapphire 2026 公布 200+ Agent 的自治叙事，但语义层 Knowledge Graph 要 2026 下半年才在 BDC 里完整 GA。中间这段窗口期，Agent 拿到的只是裸表。"
source_url: "https://e3mag.com/en/sap-joule-and-the-traveling-salesman-problem/"
---

十年前一个老问题：ERP 数据为什么这么难查。表多、关系复杂、字段名一半是缩写一半是德语习惯，连资深 ABAP 顾问都得查半天才敢回答业务一个简单的问题——这单到底卡在哪。

这个老问题最近有了新进展。SAP 在 Sapphire 2026 上把宝押在了一个叫 SAP Knowledge Graph 的东西上，说它就是让 AI 能"看懂"ERP 数据的那把钥匙。

钥匙听起来很美。但有一个细节，主舞台没大声讲：这把钥匙现在还没造出来。

## 200 个 Agent 上面那一层，叫语义层

先把 SAP 这套自治叙事的栈拆给你看。最上面是 Joule 加 200+ Agent，用户讲一句自然语言就能跨域办事；中间是 Autonomous Suite——CX、财务、供应链、HR 都接进来；底下是 Business Data Cloud（BDC），把数据先攒到一个统一的地方。

问题是：Agent 怎么知道一张叫 VBAK 的表里那些缩写字段意味着什么？怎么知道"促销订单不能退"这种业务规则藏在 Z 字段里？怎么知道客户 X 在欧洲叫 SoldTo，在拉美叫 Payer，但其实是同一家公司？

这一层就是 Knowledge Graph 干的活。它不是另一个数据库，是把数据、元数据和业务对象（客户、订单、发票……）之间的关系用 ontology 显式建模出来——让大模型"看到"业务上下文，而不是去裸表上瞎猜。

为什么是图（graph），不是向量？向量数据库做的是相似度匹配，"这两个东西看起来像"——但关系是隐式的、解释不清楚的。图是显式建模："订单 A 由客户 B 下单，使用支付方式 C，发往地址 D，引用合同 E。"每条边都能查、能解释、能审计。在企业级 AI 里，可解释这一项压倒一切，因为审计师不接受"模型觉得"这种回答。

## DSAG 在伤口上按了一下

德语区 SAP 用户组（DSAG）执行委员 Thomas Henzler 把话挑明了：过去 Joule 之所以经常答非所问，根源就是 SAP 数据库太复杂，AI 抓不住业务逻辑。

SAP 的回应是：Knowledge Graph 会大幅减少手工建模的工作量，给模型喂上业务上下文。

但 E3-Magazine 这篇社论捅破了一个时间点：独立分析显示，Knowledge Graph 目前在 BDC 里还没完整上线，要等 2026 下半年。也就是说，Sapphire 上发布的那批"自治 Agent"，从现在到下半年这几个月里，等于在没拿到地图的情况下出门跑业务。原文用的词更狠——"在语义上飞瞎"（semantically flying blind）。

这个词看起来刺眼，但贴近现实。一个跑在 Service Cloud V2 上的客服 Agent，如果它读到的客户主数据没有显式的 ontology 告诉它"这家在欧洲叫 X、在亚太叫 Y、但同属一个全球账户"，它会把同一个客户当成两个工单去处理。回答可能不算错，但跨域协作的能力出不来。

## 这次真的不一样吗

挑战派的话讲完了，得平心而论看 SAP 这边的牌面。

Knowledge Graph 这个方向，技术上是对的。HANA 里的 Graph Engine 不是新东西，过去几年一直存在感不强，没几个客户真用——因为没有 AI 拉它入场，图查询的价值很难变现。Business AI 把它从冷板凳拉到了第一线，加上 OpenCypher 和 SPARQL 能跟传统 SQL 混合查询，不用先做大规模数据迁移就能把语义层叠在现有 HANA 之上。这条路在架构上是干净的。

问题不在方向，问题在执行节奏。

2026 年 5 月发布主舞台叙事，2026 下半年才把语义层在 BDC 里凑齐——这中间有半年的"叙事先行、地基在挖"的窗口期。在这半年里，所有上 Joule 和 Agent 的客户都得自己想办法补语义层：要么手工维护 ontology，要么靠传统的 ETL 把上下文硬塞进数据产品里。手工建模这件事极考验数据质量——主数据一旦不一致，graph 不会修复你的烂数据，它只会把错误关系按图索骥地放大。

所以中国出海客户在评估 Joule 落地的时候，得问几个具体的问题：BDC 里我们已有的数据产品，下半年 Knowledge Graph 上线时是自动接入还是要重做建模？欧洲、北美、东南亚的客户主数据现在归一了吗，还是各自为政？跨地区的同一客户、同一 SKU、同一订单，在 ontology 里是同一个节点吗？

## 不是不看好，是别买叙事

> Joule 这套自治不是不行，是地基还在浇。先让数据产品、主数据归一、ontology 治理跑起来，等 Knowledge Graph 在 BDC 里完整 GA，Agent 才真正"看得见"业务。

这事在 SAP CX 这条线上尤其要紧。Sales Cloud V2、Service Cloud V2、Engagement Cloud（Emarsys）的 Agent 都需要跨域上下文——一个销售 Agent 想知道客户的服务工单状态，得能跨 Service Cloud；一个营销 Agent 想做下一个 best action，得能拿到 ERP 里的发货历史。这些跨域调用，靠 API 拉数据可以，但要让 Agent "理解"为什么这单不该再发促销邮件，必须靠语义层。

下半年是分水岭。Knowledge Graph 真在 BDC 里完整 GA、并开放给客户做客制 ontology 的那一天，自治 CX 的故事才算讲完。在这之前，Joule 跑的所有 demo 都建议带一个问题去看：这个上下文是预先在演示数据里硬建的，还是真的从 graph 实时推理出来的。

区别很大。前者是 demo，后者是产品。

参考来源：https://e3mag.com/en/sap-joule-and-the-traveling-salesman-problem/
