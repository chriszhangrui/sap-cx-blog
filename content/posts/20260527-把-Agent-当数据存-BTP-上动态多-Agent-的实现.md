---
title: "把 Agent 当数据存：BTP 上动态多 Agent 的实现"
date: 2026-05-27T00:10:00+08:00
draft: false
tags: ["SAP CX", "AI", "技术深度", "BTP", "Joule", "MCP", "Pydantic AI"]
description: "Pydantic AI + MCP + XSUAA 在 BTP 上落出来的多 agent 应用，干的是把 agent 定义从代码搬进数据库这件事——配置、热加载、JWT 透传，每一步都不容易。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-members/agentic-ai-on-btp-dynamic-multi-agent-on-demand-with-pydantic-ai/ba-p/14388032"
---

做过 SAP BTP 上 LLM 集成的同学，多半遇到过这样一种递进。

第一步是写一个 agent，把若干个 OData 服务封成 MCP 工具列表丢进去——能跑，但工具一多 LLM 就开始挑错的调；第二步是改成多 agent，一个 orchestrator 在上面，下面每个领域一个 specialist 各自接自己的 MCP server——准确率上来了，但 specialist 列表写死在代码里，新增一个领域要改 repo、重新部署、重启容器。

第三步呢？SAP Community 里有个叫 Lewis Maitre Smith 的工程师，前段时间发了一个 demo 项目 `btp-dynamic-multiagent-app`，把这一步做完了：agent 定义从一段 Python 代码，挪进了数据库的一行记录。新增、禁用、热加载都不用重启。这件事看起来像个工程小巧思，但只要你做过把企业 API 暴露给 LLM 的项目，就会知道它解决的是一个相当具体的麻烦。

## 问题不在多 agent，在 agent 的生命周期

如果你只是给一个团队、一两个 API 做 agent，把 specialist 写死在 code 里没什么问题。但 BTP 这种大平台的真实场景，是要把组织里几十甚至上百条 API 暴露给 LLM 用——每条 API 都有自己的 owner、自己的发布节奏、自己的权限边界。

这时候有两件事顶到你面前。

**一个是规模**。每发布一个新 API，难道都要等 agent 平台团队改代码、过 PR、走 CI、重新部署？这不是技术问题，是组织瓶颈。理想状态是 API 团队自己注册一个 agent，像在数据平台上注册一张新表那样。

**另一个是治理**。如果 agent 配置是代码，它就只能进 Git——但 agent 谁能创建、谁能停用、谁能 reload 编排器？基于代码的权限控制粒度太粗，要么人人能 push，要么集中到一个团队。把 agent 当成数据，问题就反过来了：能给数据加 RBAC 的所有手段都能用上，行级权限、审计、版本、diff、export/import 全是免费的。

这个项目的核心判断就一句话：**agent definition is data, not code**。

## 五层结构，每一层都把"什么是 agent"再松一点

![btp-dynamic-multiagent-app 的五层结构](https://community.sap.com/t5/image/serverpage/image-id/405457i5E85B316648B1385)

架构本身不复杂，但有几个细节值得拎出来看。

**L1 / Approuter**：BTP 上的标准做法，用 approuter 终结 XSUAA 认证。重点是它把 JWT 透出来交给 FastAPI——这一步决定了后面的身份能不能一路传到 MCP server。

**L2 / FastAPI**：两条路由 `/chat` 和 `/admin` 在同一个 app 里，但 admin 路由额外要求一个 admin scope。也就是说，普通聊天用户和能改 agent 的用户走同一套 JWT 校验，区别只是 role collection 不同。BTP 原生 RBAC 就这么直接接上了。

**L3 / Registry**：这是整个设计的发动机。它做三件事：从 DB 读出当前所有"已启用"的 agent 行；用每行配置实例化一个 Pydantic AI agent；把这些 agent 拼成一个 orchestrator 实例，里面带好 delegation tools 指向每个 specialist。Reload 按钮触发的就是这一整套重建。

**L4 / Specialists**：每个 specialist 在 DB 里是一行——name、description、instructions（system prompt）、MCP URL、enabled flag。这五个字段就是 agent 的全部"代码"。

**L5 / MCP servers**：每条 API 域自己有一个 MCP server。Specialist 调用的时候，contextvar 里的 JWT 被取出来当 bearer token 传过去——下游 MCP server 用 caller 的实际权限去鉴权，不是 service account。

## 真正难的地方是 reload 不能停服

agent 当数据存这件事，写起来不难。难的是 reload 一个正在跑请求的编排器。

原文里作者只用了一句话带过："the trick was to keep the chat ASGI wrapper thin and always resolve the current orchestrator lazily, so swapping the instance is atomic from the request's point of view"。

翻译过来：chat 入口的 ASGI 是个薄包装，每个请求进来时才去问 registry 当前的 orchestrator 是哪个。这意味着——

- 正在处理的请求，引用的是它进来那一刻的 orchestrator 实例
- Reload 时 registry 会构造一个全新的 orchestrator 对象，然后原子替换引用
- 新请求开始用新实例，旧请求继续用旧实例直到结束

零停机，零状态污染。Python 的引用计数和 GIL 在这里帮了大忙——同样的设计在 Go 或 Java 里要自己处理 atomic pointer。这个细节解释了为什么 agent 平台多半选 Python：不是因为 ML 生态，是因为运行时模型恰好契合。

## admin UI 加 agent 的全部代价

看一下从用户视角怎么加一个新 agent。

```
POST /admin/agents
{
  "name": "sales-order-agent",
  "description": "Query and analyze sales orders",
  "instructions": "You are a specialist on S/4 sales orders. ...",
  "mcp_url": "https://so-mcp.cfapps.eu10.hana.ondemand.com",
  "enabled": true
}

POST /admin/agents/reload
→ Registry rebuilds orchestrator
→ Next /chat request sees the new specialist
```

就这么多。三步：填表、保存、reload。整个过程不重启容器、不重新部署、不通知任何人。

再加上 export/import 的 JSON snapshot，agent 配置就拿到了 Git 该有的能力——版本、diff、code review、跨环境 seed。

## 对企业 AI 平台的真正含义

如果你的工作是给一个企业搭"AI Data Enabler"——把内部 API 系统化暴露给 LLM 调用——这个设计是可以直接照抄的。

反例是什么样？是一个巨大的、中心化的 MCP server，里面挂着全公司的 200 个工具。LLM 看到这个工具列表就晕了，准确率断崖式下降；治理上 200 个工具的权限要么放开要么收死，没有中间状态；运维上这一台 MCP 是组织里所有 API 团队的瓶颈。

动态多 agent 把这个反例的每一条都翻过来——

- 每个 API 域有自己的 MCP server，由自己的团队拥有
- 每个 MCP server 在平台里被注册成一个 agent，由 owner 自己负责
- Orchestrator 看到的是一组小而稳定的 specialist 列表，每个 specialist 看到的是自己 MCP server 那一小堆工具
- 身份从用户一路透传到底层 API，每个调用都用真实权限授权

暴露的 API 数量可以增长到任意规模，单个组件的复杂度都不变。这才是企业级 agent 平台该有的形状。

## 什么场景不要碰

这种动态多 agent 架构虽然漂亮，但确实有它不适合的地方。

**一是 specialist 之间需要协同的场景**。Pydantic AI 这种 orchestrator + delegation 模型，本质上还是一个主 agent 把任务派给子 agent 的"层级结构"。如果你要的是 specialist 之间互相对话、协商、迭代——那你需要的是 LangGraph 或 AutoGen 那种"图结构"的多 agent 框架，不是这个。

**二是 agent 数量很少且变动很慢的场景**。如果你确定就 3-5 个 specialist，未来一年也不会变，那把它们写死在代码里更简单——不需要 admin UI、不需要 PostgreSQL、不需要 reload 机制。"agent as data" 的优势只在 agent 多、变动频繁时才显现。

**三是冷启动场景**。registry 第一次起来需要从 `agents.seed.json` bootstrap，否则 DB 是空的、orchestrator 没有 specialist。如果你的环境频繁起停（比如 dev 环境每晚销毁），种子文件维护就成了新的负担。

## 落地几个硬约束

看似轻巧的一个 demo，要真上生产还有几道坎。

**PostgreSQL 必选**。本地开发用 SQLite 没问题，但 BTP CF 上必须用 PostgreSQL service。这意味着 MTA 里要加 `postgresql-db` standard plan，单这一项每月成本就要算进去。如果你的 BTP 子账号用的是 free tier，没有 standard plan，就跑不起来。

**Role collection 要显式分配**。第一次部署后必须手动把"Pydantic Agent Administrator" role collection 分给该有权限的用户，否则连 admin UI 都进不去。这个步骤经常被忘记，部署完发现谁都没法加 agent。

**每个 MCP server 都得是独立的 BTP 应用**。这个架构假定你已经有一堆 MCP server 在跑——每个对应一条 API 域。MCP server 自己怎么部署、怎么鉴权、怎么注册到 SAP 的 destination service，是另一整套问题。这篇文章只解决"已经有 MCP server 之后，怎么动态编排它们"。

## "Agent as data" 不是一个 trick

看完这个项目你会发现，它最大的贡献其实不是技术——是定位。

过去一年 SAP 自己也在 Joule Studio、Agent Hub 这些产品里做 agent 平台，方向同样是把 agent 当成可配置的实体，而不是写死的 prompt。Joule Studio 走 low-code 路线，给业务侧用；Agent Hub 是 lifecycle 管理的中枢；SAP 自己的官方答案，和这个 community 项目的方向高度一致。

这种设计共识形成的速度比想象中快——它告诉我们一件事：企业 agent 平台未来不会是"每家公司养十个 prompt engineer"那种形态。它更像数据平台的演进——agent 会被像数据资产那样登记、版本化、授权、审计、监控。今天写一个动态 agent 框架的工程师，做的事跟十年前写第一代 data catalog 的人没什么区别。

如果你正在为出海企业、跨国制造商、跨境电商这些场景做 BTP 上的 AI 集成——尤其是那种"我有一堆 OData 服务，想让大模型能调"的需求——这个 demo 项目的代码值得 clone 下来跑一遍。它不是产品级的成品，但作为参考架构，它把"动态 agent 平台"该有的几道关节都画清楚了。

---

参考来源：https://community.sap.com/t5/technology-blog-posts-by-members/agentic-ai-on-btp-dynamic-multi-agent-on-demand-with-pydantic-ai/ba-p/14388032

参考代码仓库：https://github.com/lemaiwo/btp-dynamic-multiagent-app
