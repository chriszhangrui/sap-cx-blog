---
title: "Snowflake 和 Databricks 正在抢 SAP BDC 的活"
date: 2026-06-01T20:15:00+08:00
draft: false
tags: ["SAP CX", "SAP BDC", "AI Agent", "Snowflake", "Databricks", "System of Intelligence"]
description: "AI 软件栈被拆成 5 层，最关键的中间层成了模型厂商、SaaS 厂商、数据平台三方混战的战场。SAP BDC 的位置在哪，CX 客户该怎么看。"
source_url: "https://siliconangle.com/2026/05/30/personal-agents-light-fuse-snowflake-databricks-move-ai-stack/"
---

这就像 ChatGPT 突然宣布要做 ERP——听起来荒谬，但 SiliconANGLE 的 Dave Vellante 和 George Gilbert 在最新一篇深度分析里正是这么暗示的：在 AI 这场仗里，模型厂商、SaaS 厂商、数据平台厂商正在挤进同一个房间，争一个之前没人单独做过的层。这一层叫"系统智能层"（System of Intelligence），SAP Business Data Cloud 想坐在这把椅子上，但椅子边上至少还有四五个人虎视眈眈。

类比当然不完美。但它的合理之处在于：当 AI 不再只是聊天框，而是要真的"动手"——查数据、调工具、跑流程、做决策——它就需要一个比数据库更厚、比应用更横向、比模型更结构化的中间层。谁占住这一层，谁就掌握 Agent 在企业里活动的"地图"。

## 个人 Agent 是导火索，不是终局

SiliconANGLE 这篇文章的判断很直接：AI 这波浪潮和 PC 时代有一个相似点——都是从底层个人生产力开始的。三十年前是 Excel、Word、PowerPoint 让普通员工把自己的活儿接管过来；今天是 Agent、开源工具、可复用的 Skill，让员工不等公司启动正式的"数字化转型"，自己先把工作流跑起来。

但这次有个根本不同：Agent 不只是产文档，它能动手。它可以读数据库、调 API、操作应用、触发流程。如果每个员工、每个部门、每个供应商都搭自己的 Agent 小岛，那企业过去几十年想消灭的"数据孤岛"问题，会以更快的速度变成"智能孤岛"。

> 原文里有句话很扎心：个人 Agent 会点燃这把火，但可持续的企业价值，最终会归属于那些把企业知识组织成"系统智能层"的平台。

## 把 AI 软件栈拆开看，到底有几层

作者画了一张栈图，把这场仗的边界讲清楚了。

从下到上看：最底层是数据平台和记录系统（ERP/CRM/Lakehouse/Warehouse），这是过去三十年企业 IT 的主战场。再往上是治理与目录层，Snowflake Horizon 和 Databricks Unity Catalog 在这一层。再往上是关键战场——系统智能层（System of Intelligence）：业务对象的统一映射、规则引擎、机构记忆、决策建议、Agent 反馈学习，这一层是 Agent 真正"活在里面"的世界。

再往上是交互入口层（Snowflake Intelligence、Databricks Genie，以及 ChatGPT 的 Codex、Claude 的 Cowork），这是新的"操作系统外壳"。最顶层是代理执行层（Microsoft Agent 365、Bedrock AgentCore、Gemini Enterprise Agent Platform）。

谁最有动力抢中间这一层？答案是所有人。

## 三方混战：每一方都在越界

这是文章最有意思的地方。模型厂商（OpenAI、Anthropic、Google）原本待在最顶层，但他们发现光有模型不够——Agent 真正要落地，必须要有结构化的企业上下文。所以 Codex 和 Cowork 这种"Coding Agent 外壳"开始往下挖，想直接接应用、接数据。

SaaS 厂商方向相反，他们害怕变成"被动的数据源"——如果 Salesforce、SAP、Workday 只是给 Snowflake 喂数据，那定价权就被夺走了。所以他们必须把自己的"业务智能层"做厚，让 Agent 在自己的地盘上跑高价值工作流。这就是为什么 SAP 在 Sapphire 2026 上反复强调 Joule 必须接 SAP Knowledge Graph、必须基于 BDC——本质上是在守自己的智能层。

数据平台厂商（Snowflake、Databricks）则是从最下面往上爬。他们已经过了"卢比孔河"——从单纯的分析工作负载，往治理、上下文、智能层一路推。Snowflake 的 Cortex、Databricks 的 Genie，都不再只是"查数据"，而是想成为业务知识的组织者。

> 原文打了个比方：这是一场"叠叠乐"游戏。每一方都想把自己的智能层叠在别人上面，让 Agent 在自己的领地里干活。结局可能是——客户得到的是 N 个互不相通的智能孤岛，而不是一个统一的企业数字孪生。

## SAP BDC 的位置：不是赢面最大，但是不能输

原文给 SAP BDC 的定位是这样的：和 Salesforce Data Cloud、Microsoft Fabric IQ、Palantir Ontology 一样，它会在自己理解最深的领域（运营和业务流程数据）里建一个"领域专属的智能层"。它不是要替代客户全部的分析栈，而是覆盖在上面、跟它同步、缓存关键部分，让 Agent 在 SAP 的业务上下文里跑得稳。

这种定位有它的好处：起步不需要等"完美的全企业数字孪生"，可以现在就交付价值。Sapphire 2026 上 SAP 反复强调"BDC 把数据、流程、治理统一进 AI 的上下文"，背后正是这个逻辑。

但风险也很明显。原文用了一个词：fragmented estates——破碎的资产。客户面对的是 SAP 的智能层、Salesforce 的智能层、Snowflake 的智能层、Microsoft 的智能层，每一层都说自己是"AI 的入口"，每一层都希望 Agent 优先在自己这里跑。

这件事对 SAP CX 客户尤其重要。Commerce Cloud 跑出去的订单数据、Engagement Cloud 推出去的营销事件、Sales/Service Cloud V2 收到的客户互动——它们和 S/4 的库存、ERP 的财务、Databricks 上的非结构化语料、Snowflake 上的第三方数据，到底应该在哪一层"汇合"？SAP 的答案是 BDC。但 Snowflake 和 Databricks 不会把这块地拱手让出。

## 对中国出海企业意味着什么

这件事看起来很美国，但对中国出海做 SAP CX 的企业有几个直接的判断点。

- **选 Agent 平台时不要只看"哪家模型最强"**。模型只是引擎，Agent 真正能不能跑得动，看的是它接的"业务上下文"完不完整。一个跨境电商如果把订单系统、商品主数据、客户标签都在 SAP 里，那让 Joule + BDC 跑订单异常处理 Agent，比让一个外部模型直接接 Snowflake 跑要实在。
- **"领域智能层"叠加问题，会变成中国出海企业的真问题**。中国制造企业海外分销网络中常见的场景是：S/4 在欧洲、Commerce Cloud 在北美、Salesforce 是收购来的子公司在用、Snowflake 在跑跨地区分析。如果不提前规划好哪一层是"主智能层"，Agent 上线之后会出现严重的语义不一致——同一个客户在四个系统里被理解成四种东西。
- **SAP BDC 的零拷贝（Zero-Copy）是真的有用还是营销话术**，今年下半年会见分晓。SiliconANGLE 这篇文章的态度比较克制——他们说 BDC 和 Salesforce Data Cloud 都是"领域专属"的智能层，言下之意是：每一家都只是部分数字孪生，整合的活儿还得客户自己干。这个判断对做跨境电商的中国卖家在做选型时是个清醒剂。

## 接下来值得盯的几个信号

Snowflake Summit 和 Databricks Data + AI Summit 都在 6 月——它们会怎么定义自己和 SAP BDC 的边界，比 Sapphire 上的官方话术更值得看。如果它们继续往业务规则、ontology、Process Definition 这些层面推，那 SAP 在中间层的护城河就会被持续侵蚀。

另一个值得看的是 SAP 自己的动作。Sapphire 2026 上 SAP 已经把 Knowledge Graph、Joule Capability 运行时、200+ Agent 都摆上台。但原文最后留了一句意味深长的话：决定胜负的不是发了多少 Agent，而是哪个平台能把"业务真正怎么跑"建模得最深、最全。

这场仗的有意思之处在于：没有谁能独赢。客户最终拿到的，大概率是一个"领域智能层 + 跨域协议"的混合架构。SAP 守得住自己的运营+流程那块地就够了，但守住的前提是真的把 BDC 和 Joule 跑成一个客户用得起、用得稳的产品，而不是一堆 PPT 上的层。

从 Sapphire 之后这两个月看，SAP 的紧迫感是明显的——BDC、Knowledge Graph、Joule Studio、Agent Hub 全在加速。但隔壁 Snowflake 和 Databricks 也没闲着。这场战争的输赢，可能不在 2026 年看出来。

参考来源：https://siliconangle.com/2026/05/30/personal-agents-light-fuse-snowflake-databricks-move-ai-stack/
