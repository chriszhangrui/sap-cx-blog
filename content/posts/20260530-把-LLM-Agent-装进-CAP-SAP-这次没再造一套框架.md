---
title: "把 LLM Agent 装进 CAP，SAP 这次没再造一套框架"
date: 2026-05-30T16:10:00+08:00
draft: false
tags: ["SAP CX", "AI", "技术深度", "CAP", "Joule", "LLM Agent"]
description: "OrchestrationClient + langchain + zod，agent 直接拿 CAP service 当 tool。一个 SAP 自家的 agent 实现样本，价值不在代码量，而在它绕开了什么。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/building-an-llm-agent-using-cap/ba-p/14404223"
---

做过 SAP 项目的都知道一个事：每次想在 BTP 上跑一段稍微复杂点的业务逻辑，第一道关都不是写代码，而是搞清楚怎么把权限、租户、事务、审计这些"production qualities"接上。换成 AI Agent，问题更尖锐——agent 想读一个订单、写一条预约，能不能复用 CAP service 上已经写好的那套规则？还是要在 agent 框架里再写一遍？

SAP 上周放出来一篇博客，作者是 SAP 内部 advisor Johannes Vogt，标题朴素得不像 SAP 风格——《Building an LLM Agent using CAP》。看完之后第一反应是：这次没再造一个 framework，而是把 langchain 直接拉进了 CAP。这个选择本身就值得说道。

## 为什么不再做一个 SAP Agent 框架

过去两年 SAP 在 AI 这一层堆了不少东西：AI Core、Generative AI Hub、Joule Studio、Joule Capability、Agent Hub。每一个都有自己的位置，但写过代码的都知道，真正落到一个 CAP 应用里时，开发者面对的问题不是"用不用 Joule"，而是"我手里这个 service，怎么让它跑起 agent loop"。

博客里给出的答案直白得有点意外：用 langchain 的 createAgent，模型走 sap-ai-sdk 的 OrchestrationClient，工具就是 CAP service 上的方法。完事。没有自定义 SDK、没有专属注解、没有"必须托管在 Joule Studio 里"这种约束。

> "An LLM, some tools and a loop meet in a bar." —— 作者引用的这句段子定义，恰恰是这篇博客的设计哲学。

换个角度看，这是 SAP 做出的一个克制选择：agent 的本质就是"模型 + 工具 + 循环"，社区里 langchain 已经把这套循环做得很成熟，没必要再发明一遍。SAP 要做的事是把"工具"这一层和企业级服务层接好——这正好是 CAP 擅长的。

## 整条调用路径长什么样

原文给了一个 travel agent 的完整 demo——三个工具：搜行程、订行程、查我的预约。把这条路径画成一张图，关键是看清楚 agent 决策、工具调用、数据访问之间的边界。

![架构图：用户一句话 → Agent 决策 → CAP service tool → CQL 安全访问数据](http://mmbiz.qpic.cn/mmbiz_jpg/lWqJzSMIBLWgjB8YLaUibdMkkzibIcUSD0LsxHWVKRmkUL97bSiav4bVHTG0b0LCZbCkVklhmOxJCUBPb3MKcz9v4BiabQzr4yl11PN7yhtHzfM/0?from=appmsg)

几个值得停下来看的地方：

- **OrchestrationClient 是模型抽象**。代码里写 `name: 'mistralai--mistral-small'`，换成 `gpt-5` 或 `anthropic--claude-4.5-haiku` 只改一行字符串。AI Core 的 Generative AI Hub 在背后做路由，这意味着 agent 代码不会被某个具体模型绑死——这点对企业架构师很重要，模型迭代节奏远快于业务系统。
- **tool 是 langchain 原生的概念，不是 SAP 自创的**。一个 tool 包含函数、zod schema、name、description。Agent 看 description 决定调谁，跟 OpenAI function calling 的写法没区别。
- **middleware 是 langchain 提供的钩子**。原文用 `dynamicSystemPromptMiddleware` 注入"今天日期"——因为模型停留在训练数据的时间点（作者实验里是 2024 年 7 月），不告诉它今天是哪天，agent 就会推荐去年的行程。这是个会让人在 demo 现场翻车的细节。

## Tool 等于 CAP service method：这个等价关系藏着关键判断

原文里最关键的一段代码，不是 agent 怎么写，而是 tool 怎么调 service。先看 read tool：

```javascript
const tripSearch = tool(
  async ({ begin, end }) => {
    const { Trips } = this.entities
    const { SELECT } = cds.ql
    const q = SELECT.from(Trips)
      .where`beginDate >= ${begin} and endDate <= ${end}`
    // 走 service 不直连 DB —— 自动复用权限
    const availableTrips = await this.run(q)
    return JSON.stringify(availableTrips)
  },
  {
    name: 'tripSearch',
    description: 'Find trips by date range',
    schema: z.object({
      begin: z.iso.date().describe('begin date, YYYY-MM-DD'),
      end:   z.iso.date().describe('end date, YYYY-MM-DD')
    })
  }
)
```

两个细节决定了这套写法是不是企业级：

第一，**where 用的是 tagged template string**，``where`beginDate >= ${begin}` ``。这不是字符串拼接，cds.ql 会把变量当参数化绑定处理。Agent 是不可信输入源——它生成的日期可能是合法格式但语义有问题，也可能被恶意 prompt injection 引导生成攻击 payload。tagged template 把 SQL 注入这一层在编译阶段就堵上了。原文专门提了一句，"还要把 agent 的输入当 untrusted data 处理"——这是十年项目经验的人才会写的话。

第二，**this.run(q) 是在 service 层执行**，不是直接打 DB。意味着 Trips 这个实体上配的 `$user` 过滤、`@restrict` 注解、before/on/after handler 全部生效。Agent 看到的数据集，跟当前用户登录后通过 OData/REST 直接访问能看到的完全一致。

再看 write tool——订行程：

```javascript
const bookTrip = tool(
  async ({ trip }) => {
    // 通过 service 事件机制触发 action
    const result = await this.send({
      event: 'book',
      entity: 'Trip',
      data: {},
      params: [trip]
    })
    return JSON.stringify(result)
  },
  {
    name: 'bookTrip',
    description: 'Book a single trip',
    schema: z.object({
      trip: z.object({
        ID: z.string().describe('ID of the trip')
      })
    })
  }
)
```

action `book` 已经在 cds 文件里定义好，里面写真正的业务逻辑（库存校验、支付、确认）。Tool 通过 `this.send()` 把 agent 的请求当成一次普通 service 调用送进去——所有 ACL、租户隔离、事件订阅、审计日志，全都按 CAP 原本的规则跑。

> 这个等价关系（tool ≡ service method）是这篇博客真正的架构主张：你不需要为 agent 单独建一套数据访问层，CAP 已经是。

## Agent 跑飞了怎么办：原文给了一个真实的翻车样本

原文里有一段比代码更值得读。作者把三个 tool（search/book/myBookings）一起给 agent，让它"帮我订个夏天的海滩行程，别问我，直接订"。结果日志是这样的：

```
[tool:start] tripSearch { begin: '2026-06-01', end: '2026-08-31' }
[tool:end]   tripSearch [{"ID":2001,"description":"7-Night Maldives ..."}]
[tool:start] bookTrip { trip: { ID: '2100' } }
[tool:start] bookTrip { trip: { ID: '2001' } }
[tool:end]   bookTrip "trip with ID 2100 booked"
[tool:end]   bookTrip "trip with ID 2001 booked"
```

Agent 一口气订了两个行程，其中一个 ID（2100）压根不在搜索结果里——典型的 LLM 幻觉。

作者没有把这段藏起来当事故，反而专门写一句话："agents 是自主的，所以一定要做 guardrails"。落地建议是：critical action 上加 confirmation step，high-impact 决策走人工 review。技术上能用的"刹车"包括 system prompt、tool description 微调、middleware 在 wrapToolCall 里拦截可疑参数。

这个翻车样本对实施方的价值，远大于一个 happy path demo。它告诉你：agentic 系统上生产之前，必须先回答一个问题——哪些 action 允许 agent 直接执行，哪些必须走二次确认。这件事 CAP 没法替你回答，得 case by case 设计。

## 和 Joule Capability、A2A 是什么关系

看完很多人会问：那 Joule Studio 和 Joule Capability 不就没意义了？不是这个逻辑。三层关系大致是：

- **CAP + langchain agent**——开发者在自己的 BTP 应用里写的 agent，独立运行，独立暴露 API，权限和数据归属全在自己的 CAP service。
- **Joule Capability**——把这个 agent 注册到 Joule 这个 super-agent 入口，让终端用户在 Joule 对话框里就能调用它。这一层主要是用户体验和 orchestration。
- **Agent2Agent (A2A) 协议**——多个 agent 之间互相调用的协议（OpenAI 提出，SAP 跟进）。原文最后留了个伏笔："tool calls 本身就是 agent 调 CAP service 的一种协议"——言下之意，A2A 也只是另一种调用协议而已。

这套分层很务实。开发者先用 CAP 把 agent 跑起来，能用、能测、有 guardrails，再决定要不要 plugin 到 Joule、要不要开 A2A。不是先选框架再写代码，而是先把代码写对再考虑接入哪里。

## 什么样的项目可以照这套写

结合做过的几个出海企业 CX 项目看，下面几类场景值得直接套用这个架构：

- **跨境电商客户的预订/查询助手**——一个跨境品牌的 Service Cloud V2 项目，需要给海外客服一个 agent，能查订单状态、能在边界条件下触发售后流程。tool 直接接 OMF 的 OData，权限沿用 Service Cloud 的 user context。
- **跨国制造企业的销售助手**——给一线销售一个 agent，输入"上周华南区前 10 大客户的拜访记录"，agent 把 Sales Cloud V2 的查询接口当 tool 调，返回结构化结果。CAP 这一层做权限隔离，sales rep 看不到不属于自己 territory 的数据。
- **会员/积分类自助查询（Engagement Cloud + CLM）**——会员小程序背后挂一个 agent，可以在合规允许的范围内查积分、查会员等级，但所有写操作（兑换、降级）走 confirmation 流程。

反过来，下面几种情况建议先按住手：

- **纯内贸的国内品牌**——AI Core 在国内没有数据中心，企业数据出境是合规硬约束，这条路径走不通。在华外资企业、有海外业务的出海品牌、跨国供应链可以走。
- **已经在 ABAP / S/4 私有云上做的客制化**——这套 CAP+AI Core 的写法是 cloud-native 范式，跟 ABAP 那套世界观差异很大。混合环境下做 agent，更适合走 BTP side-by-side + iFlow 桥接的方式。
- **对响应时间要求 sub-second 的场景**——agent 至少一次 LLM 调用，加上 tool execution，端到端通常 2~5 秒起。客服热线、实时风控这种场景，agent 只能放在异步链路里。

## 几个会被低估的实施细节

从原文里挑几个真正动手时容易踩的坑：

- **项目必须设成 ES module**——sap-ai-sdk 用 ESM，CAP 默认 CommonJS，`npm pkg set type=module` 这一步落了的话，import 直接报错。
- **cds bind 必须打在测试和运行命令前面**——`cds bind --exec -- node test/agent.js` 才能把 service key 的环境变量注入。本地用 `cds w --profile hybrid` 才能连真实的 AI Core。
- **system prompt 必须显式注入"今天是几号"**——否则 agent 默认用训练数据里的日期，给客户演示"明天的航班"会变成 2024 年的结果。原文用 `dynamicSystemPromptMiddleware` 把当前日期作为系统提示词每次刷新。
- **所有 tool 输入当 untrusted data 处理**——zod schema 只校验技术格式，不校验业务语义。一个合法日期可能是恶意构造的查询；一个合法字符串可能是 prompt injection 的载荷。CAP 这边用 cds.ql tagged template 防 SQL 注入，agent 这边在 middleware 里加输入审查。
- **tool 调 service 用 this.send，不要直接 INSERT 表**——博客里 book action 实现里有一行 `await INSERT.into(Bookings)` 看起来像在绕开 service 层，实际上那是在 service 内部的 action handler 里，已经在权限上下文里了。Agent 那边的 tool 必须走 `this.send({ event: 'book' })`，不能图省事直接 INSERT。

## 一个判断：这套写法会变成 SAP 上 agent 的事实标准吗

短期内，是的。理由有三个：

一是 SAP 自己内部的 advisor 出来写这种 step-by-step 教程，方向已经表态了。二是它没引入新概念——CAP 开发者写 service handler 本来就是这样，只是多了一个"把 service method 注册成 tool"的薄包装。三是 langchain 在社区里已经形成事实生态，SAP 顺着用，比自己造一套门槛低得多。

要保留的疑问是：当 agent 数量上去之后（一个客户可能在 BTP 上跑十几个 agent），治理、监控、版本管理、成本归因这些事情，CAP 这一层能不能撑住，还是必须靠 Joule Studio 提供的 lifecycle 管理。这个问题的答案，可能要等 Q3、Q4 的几篇博客才能看清。

但至少眼下这一篇说明了一件事——SAP 在 AI 这条线上，开始放下"全栈自营"的姿态，愿意把社区做得最好的那部分接进来用。这个姿态本身比代码更值得看。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/building-an-llm-agent-using-cap/ba-p/14404223
