---
title: "CX 用户每天切 5 个系统，Joule Work 想焊成一个"
date: 2026-05-20T23:15:00+08:00
draft: false
tags: ["SAP CX", "Joule", "Joule Work", "Autonomous Enterprise", "AI Agent", "SAP Sapphire 2026"]
description: "SAP 把 CX、财务、采购、供应链、HCM 装进同一个 Autonomous Suite。LC Waikiki 案例：员工 10 分钟变 3 秒。"
source_url: "https://news.sap.com/2026/05/future-enterprise-autonomous/"
---

SAP 在 5 月 13 日发了一个新东西，名字很普通，叫 Joule Work。第一眼看像是另一个聊天框、另一个 Copilot 的皮，没什么好聊的。但等等——这玩意儿出来当天，SAP 顺手把客户体验、财务、采购、供应链、HCM 这五条产品线全部塞进了一个叫 Autonomous Suite 的盒子里。Joule Work 是这个盒子的入口。

这就有意思了。

如果你做过 SAP CX 项目，应该能立刻想到一个画面：一个营销专员要做一次促销，她得先去 SAP Engagement Cloud (Emarsys) 里看上次活动的转化数据，再切到 Sales Cloud 看销售管线最近有哪些重点客户，然后跳到 Service Cloud 看这些客户最近有没有投诉，最后回到 Commerce Cloud 改一下落地页。四个标签页，每一个都要单独登录、各自加载、互相不通气。这是 SAP CX 客户每天的日常。

Joule Work 想干的事，是把这四个标签页焊成一个工作空间。你说一句话——"我要给上季度高客单但本月有过工单的客户做一次挽回"——它自己去拉数据、串流程、给方案，然后把执行入口直接摆在你面前。

## 一个真实数字：从 10 分钟到 3 秒

SAP 这次没有讲宏大叙事，反而抛了个具体案例：土耳其的快时尚零售商 LC Waikiki。

这家公司有几万员工，全球门店遍布欧亚。他们每天遇到的问题特别琐碎：员工要回答一个采购订单的状态，得在销售系统、采购系统、库存系统里来回切，一个问题至少 10 分钟。NTT DATA Business Solutions 帮他们用 Joule Studio 搭了一个定制 Agent，现在员工只问 Joule，3 秒拿答案。

SAP 公布的数字是运营效率 +70%、手工错误 -50%。这数字不算夸张，因为它替换的本来就是切系统、复制粘贴、等加载这种纯浪费时间的动作。

LC Waikiki 这个例子之所以值得拎出来说，是因为它是个跨国零售商。这种企业最怕的就是"系统一多就乱"——不同国家的库存、不同区域的促销、不同语言的客服，每多一个市场就多一组系统。Joule 这种"按意图组装数据"的能力，对于做全球化扩张的中国出海品牌来说，价值不在于聊天，而在于减掉信息流转的中间层。

## 三层架构：上面给人用，中间建 Agent，下面是五个域

把这次发布的几块东西画在一张图上，你会发现 SAP 的牌局其实挺清楚。

**最上面是 Joule Work**，给人用的入口。SAP 给它定义为"动态工作空间"——意思是它不像传统的应用界面（左边导航、右边表格），而是按你的意图临时拼装的一组东西。今天你做营销活动它给你拉一组组件，明天你处理客户投诉它换一组。Joule Work 的桌面应用计划在 H2 2026 GA。

**中间是 Joule Studio 加 AI Agent Hub**。Studio 是建 Agent 的地方。这次 SAP 把它对开发者社区开得很大：支持 LangGraph、AutoGen、LlamaIndex 这些主流 Agent 框架，也支持 VS Code、MCP 工具链——说人话就是开发者可以继续用自己熟悉的工具，但跑出来的 Agent 能挂到 SAP 的业务流程里。Agent Hub 是个"户口本"，不光管 SAP 自己产的 Agent，还要管 Salesforce、ServiceNow、OpenAI 这些第三方的。SAP 自己说目前 150 家公司在它上面管了超过 10 万个 Agent。

**底层是 Autonomous Suite**，五个域：Finance、Spend、Supply Chain、HCM、Customer Experience。这一层值得单独说一句：SAP 这次很明确把 Customer Experience 列为五个并列域之一。意思是 Joule Agent 在协同工作时，CX 不是孤岛，要和 Finance、Supply Chain 直接联动——你做客户挽回，需要查库存能不能补货、查应收能不能延期付款、查物流能不能加急，这些动作 Agent 都要能跨域执行。

## A2A 协议：SAP 这次没把门关死

这次发布里还有一个细节，容易被忽略。Joule 的双向 Agent-to-Agent（A2A）协议预计 Q4 2026 GA，这意味着第三方 Agent 可以安全调用 Joule Agent，反过来 Joule 也能调用别人的 Agent。

这个表态和几周前那篇"SAP 把 Agent 大门关上了，Salesforce 和 ServiceNow 却开着"形成了一个微妙的对比。当时市场判断 SAP 在搞封闭生态，现在看，它至少在协议层面留了门。

为什么这事对客户体验类客户重要？因为做出海的中国企业里，几乎没有谁是纯 SAP 栈。常见组合是 SAP Commerce + 第三方 PIM + Salesforce Service Cloud + 自建营销系统。如果 SAP Joule 能调用别人家的 Agent，A2A 协议就成了一个真正的协作面，而不是封闭花园里的人造草坪。

## 两个潜台词

第一个，Joule Studio 用了一个新东西叫 SAP Domain Models，这是 SAP 自己训练的、基于 SAP 代码、客户数据、元数据和业务流程的领域模型，计划 Q3 2026 GA。这是一种"懂 SAP 业务的 LLM"，不是通用大模型。Agent 在这上面跑，给出的判断是真正基于业务上下文，而不是泛泛的语言推理。

第二个潜台词更值得琢磨。Joule Work 这种"动态工作空间"模式，本质上在挑战 25 年来 SaaS 应用界面的设计假设——"每个产品有自己的菜单、表单、流程"。如果 Joule Work 真做成了，未来用户可能再也不会进 Sales Cloud 的界面、不会进 Service Cloud 的界面，他们只对 Joule 说话，Joule 自己去后台拼。这意味着 SAP 自己的产品线之间，UI 的边界会模糊掉，这是一个非常激进的内部改造。

> 值得记一笔的产品时间线：Joule Work 桌面应用计划 H2 2026 GA；A2A 双向协议 Q4 2026 GA；SAP Domain Models Q3 2026 GA；AI Agent Hub 已 GA。新一批 Joule Assistants 和 Joule Agents 会在 2026 年底前陆续推出。

## 对中国出海企业的实际意义

说点实在的。

做跨境电商和出海的中国卖家，现在最大的问题不是缺工具，是工具太多。Shopify 有自己的后台、TikTok Shop 有自己的后台、Amazon Seller Central 有自己的后台、Klaviyo 有营销后台、Zendesk 有客服后台——一个人一天要登录 6 到 8 个系统。如果你用 SAP Commerce + Engagement Cloud + Service Cloud 这套，理论上 Joule Work 就是冲着这个痛点去的。

但要泼一盆冷水：Joule Work 桌面 GA 在 H2 2026，等真正能用起来还要等。而且 SAP CX 在中国市场的可用性——除了 Commerce Cloud 在国内有数据中心，其他几个产品都得用海外节点，所以 Joule Work 的真实落地场景，主要还是在华跨国公司的全球总部、或者中国出海企业海外业务这两类客户里。

在华外资企业的总部对接人，可以重点关注 H2 2026 之后的早期采纳计划——SAP 提到 Joule Work 已对 Early Adopter Care 客户开放，这是占住试点位置的窗口。SAP 生态的 SI 伙伴，则应该现在就开始建 Joule Studio 的实施能力，因为 LC Waikiki 的案例里，NTT DATA 是承接落地的关键角色，这种能力会成为未来 18 个月最值钱的服务交付能力之一。

最后留一个问题：当用户不再进入产品界面、只对 Joule 说话，那 SAP CX 各个云的产品边界还重要吗？这是 SAP 自己留给行业的一个悬念。

参考来源：https://news.sap.com/2026/05/future-enterprise-autonomous/
