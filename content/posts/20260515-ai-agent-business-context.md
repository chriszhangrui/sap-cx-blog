---
title: "AI Agent听不懂你的业务，问题出在哪"
date: 2026-05-15T21:05:00+08:00
draft: false
tags: ["SAP CX", "SAP Business AI", "Knowledge Graph", "Agentic AI"]
description: "大模型擅长写文章，但面对ERP里7.3百万个数据字段的关系网，它就是个刚入职的实习生。SAP用Knowledge Graph给Agent装了一个老顾问的脑子。"
source_url: "https://futurumgroup.com/insights/precision-over-prose-why-sap-knowledge-graph-is-the-secret-to-production-ready-ai/"
---

企业AI落地这件事，有一个老问题一直卡在那里：大模型很聪明，但它不懂你的业务。

这话说了两年了。从GPT-3.5到现在，每一轮AI浪潮里，企业客户最常问的问题还是那一个——"它怎么知道我们的采购流程里，德国工厂的BOM变更会触发哪些税务规则和下游物流调整？"

答案是：它不知道。至少在两周前，大部分企业AI系统都不知道。

但最近，这个老问题有了一个值得认真看的新进展。

## 向量搜索救不了ERP

先说背景。过去两年，企业AI的主流技术路线是RAG（检索增强生成）。简单说就是把企业文档切碎，变成向量，让AI在回答问题前先"检索"一遍相关片段。

这个方法对非结构化文本效果不错——找合同条款、查政策文件都行。但面对ERP系统，它就废了。

为什么？因为ERP里的数据不是"文本"，是**关系网**。一个采购订单背后连着供应商主数据、合同条款、预算中心、审批流程、税务规则。这些关系是层级化的、有逻辑依赖的。向量相似度搜索对这种结构性逻辑根本无能为力。

Futurum Research的分析师Brad Shimmin在Sapphire 2026后写了一篇很有料的深度分析，直接点破了这个要害：传统RAG能处理文本相似度，但它搞不定多跳推理——理解"实体A连接着流程B，而流程B受制于策略C"这种关系链。

![SAP Sapphire 2026 现场](https://news.sap.com/wp-content/blogs.dir/1/files/2026/05/12/SAP_SAPPHIRE25ORL_16296.jpg)

## SAP的解法：把老顾问的脑子变成一张图

SAP在Sapphire 2026上推出的**SAP Knowledge Graph**，做的就是这件事——把7.3百万个数据字段以及它们之间的业务关系，编织成一张结构化的语义地图。

这不是什么花哨的新概念。Knowledge Graph这个东西Google十几年前就在用了。但SAP做的这张图，specificity不一样——它编码的是企业业务领域的专有知识：流程之间的依赖关系、数据实体之间的归属层级、权限规则、安全边界。

CEO Christian Klein在现场讲了一个梗：一个AI生成了一只三只耳朵的独角兽。意思很明白——通用大模型80%的准确率放在日常聊天没问题，放在财务关账或者供应链调度里，就是灾难。

Knowledge Graph的价值，用Shimmin的话说，就是给AI Agent配了一个"高级顾问的部落知识"——它知道哪些字段之间有因果关系，哪些操作会触发什么下游影响，哪些数据在什么场景下不能碰。

## 这对CX意味着什么

说到这里，做CX的人可能觉得这是后台的事。

不是的。

想想这个场景：一个AI Agent接到任务，要"优化某客户的服务体验"。如果没有Knowledge Graph，这个Agent只知道客户最近投诉了一次。但有了这张图，Agent能看到——这个客户三个月前换了一个新的服务合同，合同里有SLA条款变更，而这个变更影响了他的工单优先级计算逻辑。投诉的根因不是服务态度，是系统里的优先级排序出了问题。

这才叫"懂业务"。不是能回答"我们的退货政策是什么"，而是能在复杂关系网里找到真正的因果链。

![SAP Sapphire 2026 演示](https://news.sap.com/wp-content/blogs.dir/1/files/2026/05/12/SAP_SAPPHIRE25ORL_17020-1.jpg)

## 配套动作：RPT-1.5和主数据治理

Knowledge Graph不是一个单独的产品，它是整个SAP Business AI Platform的"骨架"。围绕它还有两个关键配套：

- **SAP-RPT-1.5**——一个专门处理结构化表格数据的AI模型。它用的技术叫RAP（Retrieval-Augmented Prediction），能做到列级别的可解释性。就是说它告诉你："我的预测是基于这三个字段得出的"，而不是黑箱吐一个数字。

- **Reltio收购**——把多域主数据管理拉进SAP Business Data Cloud。Shimmin说得很直接：在Agentic AI时代，MDM不再是后台优化任务，而是生存前提。如果Agent分不清采购系统里的"供应商X"和财务系统里的"供应商Y"是同一家公司，那它做的所有"自主决策"都是在错误数据上跑的。

## 方向对了，但别高兴太早

Knowledge Graph这条路，我觉得方向是对的。它解决的是一个真实痛点——让AI从"能聊天"进化到"能干活"。

但有几个现实问题需要泼冷水：

- 7.3百万个数据字段听起来很唬人，但这张图的覆盖度对非SAP系统有多大？很多企业的数据有一半甚至更多散落在SAP体系外。Knowledge Graph能不能跨出SAP的围墙，是决定它到底是"企业大脑"还是"SAP专用大脑"的关键。

- Agent-to-Agent互操作。Shimmin提到要观察A2A和MCP协议标准的落地情况——Joule能不能安全调用微软或Google的第三方Agent，同时利用Knowledge Graph的上下文。这一步如果做到了，SAP真的有机会成为企业的中枢神经系统。做不到，就只是又一个锁定生态。

- 落地的前提还是Clean Core。没做完云迁移的客户，这张图对你来说就是一张画饼。SAP说Agent能把ERP迁移工作量减少35%，这个数字能不能兑现还得观察。

Knowledge Graph是SAP从"卖软件"到"卖结果"这条路上最关键的一块基础设施。它不花哨，但它解决的问题是真实的。只是从产品发布到客户真正用上它自动关账或者自动排单，中间还隔着一大段"干脏活"的过程。

Shimmin文章最后给了一个很好的观察角度：盯住Autonomous Close Assistant的实际表现。如果它真能用Knowledge Graph把几周的人工对账压缩到几天，那SAP这步棋就算下活了。

参考来源：https://futurumgroup.com/insights/precision-over-prose-why-sap-knowledge-graph-is-the-secret-to-production-ready-ai/
