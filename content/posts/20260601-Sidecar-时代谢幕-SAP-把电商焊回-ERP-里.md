---
title: "Sidecar 时代谢幕：SAP 把电商焊回 ERP 里"
date: 2026-06-01T12:10:00+08:00
draft: false
tags: ["SAP CX", "SAP Commerce Cloud", "Sapphire 2026", "B2B 电商", "ERP"]
description: "Sapphire 2026 SAP 推 Commerce Cloud cloud ERP edition：90 天上线，电商焊进 ERP。这一刀砍向 composable 阵营和靠集成吃饭的中间件商。"
source_url: "https://www.shashi.co/2026/05/the-death-of-sidecar-commerce-saps-move.html"
---

大家都觉得 SAP 这次最大的动作是 200 个 AI Agent，是 Joule 改名 Joule Work，是把 BTP 焊进 Business AI Platform。但其实最值得一看的，是它悄悄推出了一个新版本的 Commerce Cloud——叫 cloud ERP edition——并且用一种几乎挑明的方式宣布了一件事：过去十年企业电商赖以生存的"sidecar 架构"，要被它自己亲手埋掉了。

Info-Tech Research Group 的研究总监 Shashi Bellamkonda 5 月 28 日发了一篇独立分析，把这件事讲得比较透。他说 Sapphire 2026 其实是一场葬礼，只是没人愿意这么叫它。葬的就是过去十年那种"电商平台坐在 ERP 旁边"的玩法。这个判断挺重，但我读完原文之后，反倒觉得有些道理值得展开聊聊。

## Sidecar 架构是怎么来的

先说清楚 sidecar 是个什么东西。从 2010 年代初开始，但凡上规模的企业做电商，几乎都是这个模式：左边一个 ERP（SAP、Oracle、或者更早的某个东西），右边一个电商平台——Hybris、Magento、Salesforce Commerce Cloud，后来又加上 commercetools。两个系统之间夹着一层中间件，负责把价格、库存、客户身份、订单这些东西"对齐"。

这个模式给中间件厂商和实施伙伴养出了一整个产业，活得很滋润。但代价是，企业自己得同时维护两个根本就不打算彼此承认的"真相之源"——电商平台知道客户点了什么，ERP 知道客户欠你多少钱，两边的数据永远没法在同一时刻完全对齐。每一个个性化项目最后都撞同一堵墙：商城里看到的价格和 ERP 里跑出来的价格不一样，库存信号晚了几个小时，客户在两边是两个 ID。批处理任务跑挂了一个晚上，第二天早上整个 B2B 团队都在打电话道歉。

## SAP 这次到底发了什么

这次发的产品叫 SAP Commerce Cloud, cloud ERP edition。名字看着像是个版本号变化，其实里面的架构是从头重写的。SAP CX 的 CMO Jessica Keehn 在 Sapphire 上把话讲得很直接——这个版本的目标不是再做一个能"配 ERP 用"的电商，而是让电商"运行在 ERP 之上"，不是 beside it，而是 with it。

具体表现是什么？这个版本原生连 SAP Cloud ERP 和 Business Suite，从商品发现、定价、报价、下单、履约一直到售后，全链路用同一份数据。客户不管是通过 EDI、邮件、电话、销售员还是数字商城下单，看到的价格、订单状态、发票、账户都是同一份。SAP 给出的实施目标是 **90 天**。这个数字之所以能讲，前提是把传统电商项目里耗时最长的那一段——集成设计——直接消灭掉。这是这个版本的核心论点。

起步覆盖的是已经在用 S/4HANA Public Cloud 的中小型 B2B 企业。Public Cloud 版 Sapphire 上发，Private Cloud 版排在 2026 年底。

## 7 月 31 日那一刀

这个时间点不是巧合。SAP Hybris 本地部署版本的主流维护服务 **2026 年 7 月 31 日** 终止——之后没有常规安全补丁，没有更新，没有主流支持。本地 Hybris 还活着的客户，要么忍受技术债持续累积、安全风险一路上扬，要么必须在这个时间窗口前后做出迁移决定。

SAP 把 ERP Edition 的发布时间正好踩在这个截止日期之前几个月。这不是产品路线图的偶然，是个很有计算的动作——存量 Hybris 客户被迫做选择：往新的 SAP Commerce Cloud 上迁、跳到 commercetools/Shopware 这类 composable 平台、或者走这条新的 ERP Edition 路线。

## 这把刀砍向谁

真正有意思的部分在这里。Shashi 在他文章里直接点了名——如果 SAP 这一招走通了，输的不只是 Hybris 的存量竞争对手。

第一类是 composable 阵营的厂商——commercetools、Shopware 这些 MACH 派系。它们这一年都在抢同一拨 Hybris 迁移客户。它们的论点也站得住脚：SAP Commerce Cloud 即便加了 Spartacus storefront 和一些 SaaS 组件，commerce 内核仍然是相对单体的架构。Gartner 在最近一份分析里也认了这一点。但 SAP 这次的反论是：composable 在你需要在多个厂商之间拼最佳组件时，确实是优势；但当你的主数据已经在 SAP 里、你的主要买家是企业采购团队、他们要的是价格准确和订单透明而不是商城创新速度——这时候 composable 那点架构灵活性，性价比就没那么高了。集成税并不会因为换了 composable 就消失，它只是换了个供应商关系继续存在。

第二类，是中间件厂商和靠"我们能让 SAP 跟你的商城对话"吃饭的那一批 SI 实践组。这个生意过去十年很赚钱，因为 sidecar 模式天生就需要有人来缝合。但如果 SAP 真的做到了 90 天上线、集成阶段被消灭——这个收入模型就站不住了。原话是："That practice area does not survive at current scale if SAP eliminates the problem it is solving."

中国的 SAP 生态里，做 Hybris 集成的合作伙伴并不少。这一刀切下去会有多疼，要看接下来 12-18 个月里 ERP Edition 在中国出海客户、在跨国制造和分销企业里的实际落地速度。如果落地路径真的简化到位，靠"我们写得了 Hybris 客制化逻辑"就能赢项目的日子，确实快到头了。

## 90 天这个数字得打个问号

Shashi 自己也在文章里给这个数字泼了点冷水。90 天上线值得在它变成销售承诺之前先掂量一下。德国 SAP 用户组 DSAG 已经公开提了几个开放问题——围绕 Business Data Cloud 和 Customer Data Cloud 的迁移，以及 Private Cloud 交付时间。这是合作伙伴生态的一个早期信号：keynote 上的故事，和实际落地能跑出来的样子，中间差距永远存在。

值得一看的另一个点是 SAP 自己引用的数据：连续 11 年（自 2014 年起）被 Gartner 数字商务魔力象限评为唯一持续上榜的厂商。这个连续性在采购信号上是有分量的，即便底层架构在过去十年被无数次拷问。

## 谁该先动，谁可以再等等

ERP-native commerce 的论点在哪几类客户身上最站得住？三类：B2B 制造企业，定价和合同结构复杂；分销商，库存准确度本身就是商业差异化能力；强监管行业，订单可审计性是底线。这三类客户的共同点是：他们要的不是商城多酷炫，是数据要对、要快、要准。

论点最弱的是哪些？B2C 主导、品牌体验导向、时尚奢侈和 DTC 这一类。SAP 在这些行业本来就一直没站稳脚跟，ERP Edition 这一步也不会改变这个格局。这个判断挺直白，但确实是事实。

最该动的，是已经在 RISE with SAP 上、却一直把电商现代化往后推、还在跑老 Hybris 基础设施的那一批。7 月 31 日的维护截止是真的，不是营销话术。中国出海企业里，做跨境 B2B 分销、做 OEM/ODM 全球渠道管理、做工业品出海的客户，正好是这个画像。

至于其他人——Shashi 的建议是看 H&M。Sapphire 主舞台上的 H&M 案例不是 vision slide，是生产环境信号。如果"统一商务 + Agentic"这个故事在 H&M 这种全球零售的生产环境里跑通了，sidecar 架构的寿命可能比 SAP 现在瞄准的 B2B 中端市场要短得多。

## 最后一句

SAP 这次想讲的论点——"电商能力嵌进 ERP 不是限制，是设计原则"——这句话其实在 2018 年 Hybris 还是 SAP CX 主角的时候，它就想讲，但讲不清楚。现在它能讲出来，是因为 S/4HANA 的 clean-core 架构、BDC 的数据层、Joule 的 Agentic 接口，把这个论点的结构性可信度补齐了。这一点客观上是变了。能不能在企业的真实落地里跑通，那是另一个问题。但至少这一次，sidecar 这个词作为一个时代的名字，确实开始进入历史阶段了。

参考来源：https://www.shashi.co/2026/05/the-death-of-sidecar-commerce-saps-move.html
