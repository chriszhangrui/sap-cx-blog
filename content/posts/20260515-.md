---
title: "SAP自研AI模型RPT-1.5来了，RAG的表格版你听说过吗"
date: 2026-05-15T15:10:00+08:00
draft: false
tags: ["SAP CX", "SAP Business AI", "Joule", "RPT-1.5", "Knowledge Graph"]
description: "Sapphire上SAP掏出了自研预测模型RPT-1.5和RAP技术，Joule Work要变成新的操作界面，Knowledge Graph进生产——技术栈的变化比口号重要得多"
source_url: "https://www.techtarget.com/searcherp/news/366642887/The-AI-technology-behind-SAPs-Autonomous-Enterprise-pitch"
---

今天看到一条消息值得聊聊。

TechTarget在Sapphire现场采访了SAP Business AI产品管理VP Richard Grandpierre，聊的不是那些"自主企业""Agent无处不在"的大词，而是具体的技术架构——Joule Work长什么样、RPT-1.5模型到底升级了什么、Knowledge Graph怎么进的生产环境。

说白了就是：SAP终于愿意掀开引擎盖，给外面看看里面装的到底是什么。

## Joule Work：不是聊天框了，是操作台

之前Joule给人的感觉就是个对话窗口，你问它问题它给你答案。Grandpierre说现在的定位完全变了——Joule Work是一个workspace，你可以在上面配置tiles、拉数据、做交易，像个动态仪表盘。用户不再需要跳转到不同的application去做事，所有工作在一个界面里完成。

更激进的说法是：SAP不再围绕"给用户堆更多功能"来开发了。以前是不断增强ERP的feature set让它更强大，现在是反过来，把busy work自动化掉，让用户变成"流程的控制者"而不是执行者。

这是一个架构层面的范式转换。用户和SAP系统的交互模式从"操作应用"变成"描述目标"——你告诉Joule你要什么结果，它来编排后面的agent去执行。

![SAP Business AI Platform](https://news.sap.com/wp-content/blogs.dir/1/files/2026/05/11/BAIP2.jpg)

## RPT-1.5：SAP自研的预测模型，搞了个"RAP"出来

这个是我觉得最有意思的部分。RPT全名叫Rapid Prediction Transformer，是SAP自己训练的基础模型，专门用来做表格数据的预测。不是做自然语言的，是做结构化数据预测的——比如预测某个字段的值、某个订单的结果。

1.5版本三个关键升级：

- **RAP（Retrieval Augmented Prediction）**——你可以把它理解为RAG的表格数据版本。以前模型的上下文窗口有大小限制，现在通过RAP机制可以处理任意大小的数据集。大的数据表也能用来生成预测了。

- **可解释性**——模型不光给你预测结果，还用LLM来解释它是怎么得出结论的，置信度多少。而且现在可以直接对着模型聊天，问它"为什么这么判断"。

- **Tabular AI多模型编排**——跟LLM选模型一样，表格预测也可以选不同厂商的模型了。Prior Labs（SAP刚收购的德国AI公司）的模型已经接进来了，加上Gemini、OpenAI等，可以按质量需求编排使用。

![SAP Deployment Architecture](https://news.sap.com/wp-content/blogs.dir/1/files/2026/05/11/Deployment.png)

## Knowledge Graph终于进生产了

Knowledge Graph这个东西SAP讲了有一阵了，但之前更多是概念。这次Grandpierre明确说：已经bake进了新版Joule里面，覆盖S/4HANA public cloud、private cloud、Ariba和SuccessFactors。

它干的事情其实很关键：SAP系统里有成千上万的表和视图，如果让LLM直接去查数据，它根本不知道该查哪张表。Knowledge Graph是所有API和数据结构的语义地图，告诉AI"这个数据在哪里、表和表之间是什么关系"。有了这个，Joule查数据的准确率就上来了。

之前客户反馈说Joule有些查询能回答、有些回答不了，原因就是那些领域的Knowledge Graph还没覆盖到。现在正式进了生产环境。

## On-prem客户也能用AI了？有条件的

这个话题很现实。SAP宣布说正在迁移中的客户（ECC/S4 on-prem）也可以获得一些AI能力。但Grandpierre非常坦率：条件是你必须已经把50%以上的维护合同转换到了云订阅，说明你确实在迁移路上。

而且他强调这并不轻松。云环境下SAP知道API endpoint在哪里、怎么配置，开箱即用。但ECC系统的技术层完全不同，客户可能还有大量定制化，要把AI能力交付到on-prem需要"大量双方的重活"。

说白了就是：能给你用，但别指望跟云客户一样的体验。这更像是一个过渡期的安慰奖，而不是对等的能力交付。

## 我的判断是

这次Sapphire上SAP讲"自主企业"的声音很大，但真正让我觉得有东西的是这些技术细节。RPT-1.5说明SAP在自研AI模型上是认真的——不光靠Anthropic和OpenAI的LLM，在结构化数据预测这个自己最擅长的领域也在建护城河。

RAP这个概念特别值得关注。企业里90%的决策数据是表格形式的，不是文本。如果SAP能把表格数据的AI预测做到LLM处理文本那样好用，这个价值比多装几个Agent大得多。

Knowledge Graph进生产也是个里程碑。没有它，所有Agent都是在黑暗中摸索。有了它，Agent才真正知道去哪里取数据、数据之间是什么关系。这是Agent可靠运行的地基。

至于Joule Work——如果真的能替代传统的SAP GUI和Fiori界面成为主要操作入口，那才是用户体验层面的革命。但这个还需要时间验证。

参考来源：https://www.techtarget.com/searcherp/news/366642887/The-AI-technology-behind-SAPs-Autonomous-Enterprise-pitch
