---
title: "Sapphire Madrid 200个Agent摆台上，真把8.5万人推上Joule的只有几家"
date: 2026-05-23
source: https://news.sap.com/africa/2026/05/ai-unleashed-as-companies-showcase-business-impact-at-flagship-sap-event/
---

等一下——把节奏放慢一点。

SAP 五月在马德里办了 Sapphire 欧洲场，台上摆出来的数字是这样的：**200 多个专用 Agent、50 多个 Joule 助手**，覆盖财务、供应链、采购、HR、客户体验五条业务线。一份从架构到产品组合到合作伙伴名单一应俱全的发布稿。

但你打开 SAP 非洲分部那条 5 月 22 日的发文，会看到另一个版本的故事。这版故事不讲 200，它讲一个数字：**85,000**。

这是 Ericsson 已经活在 unified Joule 上的真实用户数。爱立信，那个全球 180 个国家的电信设备巨头，今年正好 150 周年，全世界 40% 的移动流量从它的网络上走过。

8.5 万个真实员工，不是 demo 账号，不是 license，是每天打开 Joule 干活的人。这个数据放在整个 SAP 客户群里，已经是金字塔尖那一小撮。

更有意思的不是数字本身，是台上 Ericsson 那位副总裁说的一句话。

她叫 Esra Kocatürk Norell，头衔是 Vice President, Customer Experience, Enterprise IT。注意这个 title——她不是供应链的、不是财务的，她是企业 IT 里管 Customer Experience 那条线的副总。她在台上说了一句让我反复看了三遍的话：

> Once you scale AI, it stops being an AI problem—and becomes a data problem.

一旦你把 AI 推到规模上，它就不再是 AI 的问题了——它变成了数据的问题。

这句话听起来朴素，但它背后是 SAP 整个 Sapphire 故事的底牌被翻过来给你看。

## PoC 跑得很漂亮，规模化的时候崩了

我们来拆一下。

过去两年，企业 AI 项目最常见的死法是什么？是 PoC 阶段跑得很漂亮，演示给老板看一切完美，到了规模化部署那一步突然崩了。崩的原因写在每家公司的事后复盘里都差不多：数据质量不一、字段定义打架、跨系统语义对不上、权限规则散落在十几个系统里没人能讲清楚。

模型本身没问题。Claude 也好、Gemini 也好、SAP 自家的 RPT 也好，丢一个干净的样本进去，准确率都不差。问题出在你把它接到真实业务环境的那一刻——它面对的不是干净样本，是一团二十年累积下来的乱麻。

Ericsson 是怎么做的？他们没有先去挑模型、没有先去挑 Agent，他们先去铺了一层东西，叫 **business data fabric**，业务数据织布。用的工具是 SAP Business Data Cloud，也就是 BDC。

## BDC 干的事，是"定义一次，到处生效"

BDC 这个产品过去一年我写过几次，但每次都觉得讲不透。直到看 Ericsson 这个案例，我才找到一个比较准的表达——

**BDC 干的事，是让你"定义一次，到处生效"。**

营收是什么意思？市场结构怎么划分？访问规则谁说了算？这些问题在传统企业里，是每个系统一套答案。CRM 系统里的"客户"和 ERP 系统里的"客户"对不上号、HR 系统里的"区域"和销售系统里的"区域"边界不一样，这种事情我接触过的客户里至少有八成都遇到过。

BDC 的逻辑是把这些定义提到一个统一的语义层，下面的数据该在哪还在哪——这就是 federated data architecture，联邦式数据架构。**数据不搬家，但语义对齐。**

> With SAP Business Data Cloud, we can define what data means once—from revenue to market structures and access rules—and apply it consistently across the enterprise.

定义一次，全企业一致使用。听起来像是一句营销词，但当你把它和"85,000 用户上 Joule"连起来看的时候，逻辑就通了。

8.5 万人能在同一个 AI 助手里问问题、得到一致的回答，前提是他们看到的数据语义必须一致。否则你就会看到这种诡异的场景：销售大区的 VP 问 Joule"我们北区这个季度营收多少"，财务的 VP 问"北区营收"，两个人得到两个不同的数。

这种事一旦在真实环境里发生一次，整个 AI 项目的可信度就崩了。Ericsson 是先把这一步做完了，才敢把 8.5 万人放上去。

## "我们很早就投了"——这个 timing 很关键

这里有个细节值得多说两句。

Esra 那句话还有后半截，她说：*"That's why we invested early in a business data fabric."*

我们**很早**就投了。

这个 timing 很重要。Ericsson 不是看到 Sapphire 2026 之后才开始铺 BDC，他们是在 SAP 把 BDC 推成主力产品之前就在做这个工作。换句话说，他们把数据底座当成了一个独立的、值得投入的工程，而不是把它当成 AI 项目的"前置任务"。

这个思路差异有意思。绝大多数客户做 AI 项目的时候，数据治理是被 AI 倒逼的——上 Agent 之前发现数据有问题，临时去补。问题是临时补的数据治理，做出来的东西就是临时治理的水平。Ericsson 那个反着来：先把数据治理做扎实，AI 上来的时候是水到渠成。

我接触过的项目里，有这种思路的客户不多。大多数还是想着"先上一个 Agent 看看效果"，然后发现效果不行的时候再回过头来想数据的事。

## 围绕端到端流程，而不是单点功能

Ericsson 还和 SAP 在做一个联合创新项目，叫 intelligent goal recommendation——智能目标推荐。这个能力是在 SAP SuccessFactors 里。SuccessFactors 不是 CX 产品，所以这块我不展开。但有一个细节值得记下来：他们做联合创新的方式，是**围绕端到端业务流程组织**，而不是围绕单点功能。

这是规模化 AI 项目的另一个隐性要求——你的 AI 不能只优化一个步骤，它必须能跑完整个端到端流程。否则就是把人工瓶颈从 A 转移到 B，没有真正释放产能。

## 翻译给中国出海企业的 CX 决策者

**第一**，Ericsson 这个案例的可复制性其实没那么强。他们是 1500 亿瑞典克朗营收量级的公司，能投一个独立的数据 fabric 工程。但是它揭示的逻辑——"AI 规模化的瓶颈在数据层，不在模型层"——是普适的。这个逻辑对中国出海品牌做全球营销云、跨境电商做客户数据统一、跨国制造企业做全球分销网络的客户主数据，都是同一套底层规律。

**第二**，BDC 的"语义一次定义"思路，在出海场景里价值会被放大。海外业务跨多个市场、多套法规、多种渠道，你的"客户"在欧洲是一个法人对应一个 legal entity，在东南亚可能是一个家族对应多个采购实体，在美国是按州划账户。这种语义差异不在 BDC 这层管住，下面 Sales Cloud、Engagement Cloud、Commerce Cloud 任何一个产品里出报表都会打架。

**第三**，"先铺数据再上 AI"这个顺序，在中国客户那边推起来会比较辛苦。因为 AI 项目通常是 CEO 或者 CMO 拍板的，预算压力大、要快速看到效果。但数据治理这种活，做半年看不见东西。这种节奏冲突是真的存在，没法回避。

我看到的折中做法是：把第一个 AI 用例选得窄一点。不要一上来就做"全公司的 Joule 助手"，而是选一个数据本来就比较干净的领域——比如纯线上营销活动的归因分析、比如标准化的客服 FAQ——先跑出第一个看得见结果的项目。然后用这个项目挣来的信任，去推更基础的 BDC 投入。

## 200 个 Agent 是结果，BDC 才是地基

回到 Sapphire 那 200 个 Agent。

SAP 自己其实知道这件事。Klein 在 keynote 上讲 Autonomous Suite 的时候，反复在强调一个词——**governed**，受治理的。Knowledge Graph、Joule Studio、Agent Hub，这些发布里很大的一部分，都是在解决"AI 怎么不胡说八道"和"AI 怎么知道你公司在干嘛"这两个问题。

而这两个问题的底层答案都是同一个：**你得有一层干净的、对齐的、可治理的数据。**

200 个 Agent 是结果，BDC 才是地基。8 万人能用起来 Joule 是结果，data fabric 才是底座。

Ericsson 那位 CX 副总裁讲的就是这件事。她讲完以后，台下大概有一半人鼓掌，另一半人在心里默默算自己公司的数据情况——多半是脸色不太好看的那种。

---

参考来源：https://news.sap.com/africa/2026/05/ai-unleashed-as-companies-showcase-business-impact-at-flagship-sap-event/
