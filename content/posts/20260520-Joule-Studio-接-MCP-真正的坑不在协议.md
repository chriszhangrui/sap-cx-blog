---
title: "Joule Studio 接 MCP，真正的坑不在协议"
date: 2026-05-20T15:00:00+08:00
draft: false
tags: ["SAP CX", "AI", "技术深度", "Joule", "MCP", "BTP"]
description: "从 SAP Community 一篇 BTP 上的 MCP Server 实施手记复盘：Destination 怎么分层、认证模型为什么要退到 thin orchestration、tools/list 通了为什么不算通。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-members/sap-joule-studio-mcp-setup-guide/ba-p/14384568"
---

## 从一个看似简单的目标说起

Joule Studio 增加 MCP（Model Context Protocol）支持之后，纸面上的事情看起来很短：把工具通过 MCP Server 暴露出来，在 Joule Studio 里挂上这台 Server，Agent 就能通过标准接口调用后端能力。

真做下去会发现，从"服务起来了"到"Agent 真的在用这些工具"，中间不是配一个 endpoint 那么轻。最近读到一篇 SAP Community 的实施手记，作者把多套后端的结构化数据和文档引用通过自建的 MCP Server 暴露给 Joule，跑在 BTP Cloud Foundry 上，复盘了从 Destination、XSUAA 到 MCP 生命周期的完整踩坑路径。值得拿出来逐层拆。

## 为什么没有一上来就写 MCP Server

这一步是整个实施里最有意思的判断。作者最初的本能是直接搭脚手架冲 MCP，但他停了一下，先用 Express 写了个轻量 wrapper，开几条临时 `/test` 路由，把后端调用本身验证通了，再把这个 wrapper 改造成真正的 MCP Server，最终 endpoint 落到 `/mcp`。

这种"先证明数据通路再叠协议"的方式，省了大量后期定位时间。事后他自己总结，很多看起来像 MCP 协议出错的现象，其实根因在 Destination 配置或者认证模式上。如果一开始就把 MCP 协议处理叠在没验证过的后端调用上，故障域会立刻交叉，根本分不清是 transport 还是 backend 的问题。

## Destination 模型：薄，再薄一点

最终跑通的拓扑里有一个核心选择——MCP Server 自己不存 backend 的完整 URL，而是调用 named destination + 相对路径。整个 Cloud Foundry 应用内部一共三类 Destination：

- 一类指向 DMS / 文档相关后端
- 一类指向 processor 类业务服务（故障码、设备上下文、维修历史等）
- 单独一条 Joule-facing Destination，指向这台 MCP Server 自己，不指向 CAP

Joule-facing 这条 Destination 上有一条小但要命的纪律：URL 里不要拼 `/mcp`，`/mcp` 只配在 Joule Studio 的 advanced path setting 里。混在一起拼会让排错时很难判断请求到底到哪一层就断了。

## 认证模型：从"带 token 的 gateway"退到"thin orchestration"

最早作者把 MCP Server 设计成一个偏认证密集的 gateway——希望它接住 Joule 来的 bearer token，再透传给后端。几轮跑下来，问题集中在三个地方：Destination credential 过期、浏览器认证路径和技术后端调用混淆、approuter 行为和裸 Node.js 应用之间的差异。

转折点是把模型简化：Joule-facing Destination 直接指向裸 Node.js 应用，MCP Server 不再依赖任何 incoming bearer token，技术 OAuth 完全交给 backend destination 自己 client-credentials 流程处理。MCP Server 退化成一个"orchestration only"的薄层——拿到工具调用请求，照转给对应的 destination，把结果还回去。

这是企业里集成 Agent 框架时容易忽视的一条架构判断：在浏览器场景下 user session 透传是顺的，但 Joule 是技术身份发起调用，强行让 MCP Server 当 token 中转站会把它和 approuter / XSUAA 状态深度耦合。退一步把它做成无状态、不认 token 的 thin orchestration 层，整条链路立刻可推理。

## MCP 生命周期：tools/list 通了不等于通了

把 wrapper 升级成正式 MCP Server 时，作者实现了四个核心生命周期方法：

```
initialize
notifications/initialized
tools/list
tools/call
```

实施途中有一个不显眼但关键的发现：Joule 能 reach 到 endpoint，但流程一直卡住，原因是 `notifications/initialized` 这个握手通知没显式处理。补上之后，tool discovery 才真正开始工作。这是一个让"服务可达"和"服务符合 Joule 期望"分道扬镳的细节。

为了排除是不是 schema 风格导致 `tools/call` 推不动，作者还做了一个对照实验——单独用 Zod-based schema 重写一套（独立应用、独立 Destination、独立验证流程）。结果两套实现从 Joule 角度看完全一致：`initialize → notifications/initialized → tools/list` 都通，但 `tools/call` 推不到。结论非常重要——schema 风格不是这次的瓶颈，剩下的拦路问题在 Joule 端的 invocation path，而不在服务器注册侧。

## 从这次实施里能搬走什么

把这次踩坑往项目方法论上抽，几条经验直接可以套到下一个客户：

- 分层验证：先把 backend 调用单独跑通，再叠 MCP 生命周期，再验 tool discovery，最后才是 tool invocation。每一层独立诊断，根因定位才不会被交叉故障带偏
- 认证假设要早做减法：任何隐式依赖浏览器 session 的设计，一旦 MCP Server 成为真正的入口就会变得难以推理。technical OAuth 该归 backend destination 就归过去
- `cf services` / `cf env` / `cf logs` 是实施工具不是排查工具——Cloud Foundry 项目里这三条命令应该和 build 同等优先级
- tool discovery 通了不代表 tool invocation 通。卡在 `tools/list` 之后时，要意识到瓶颈可能已经不在自己的 MCP Server 上

## 什么样的项目可以现在动手

出海制造业、跨境零售、有海外多个业务系统的全球化运营场景，是这套模式当下最合适的落点——后端散在 BTP / S/4 / 第三方 SaaS 上，但希望员工通过 Joule 自然语言调度，而不是为每个场景做一个独立的 Fiori 入口。

不要现在就上的情况：后端服务还没有标准化的 OAuth 客户端凭据流、Destination 没沉淀好、连 BTP subaccount 的角色和 role-collection 都没理清楚。在这种状态下硬上 Joule + MCP，会把所有平台基础设施的债一次性还到 Agent 项目身上，看起来是 AI 项目失败，其实是平台地基没打好。

最后一条警示：MCP 现在还是一个活跃演进的协议，Joule Studio 的 invocation path 行为也在动。所以做 PoC 时务必把 MCP 协议版本、Joule 端的发布版本、SDK 版本三者锁住，不然过两个月再回头看自己的 demo，可能复现不出来。这不是 SAP 的问题，整个 MCP 生态都还在收敛。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-members/sap-joule-studio-mcp-setup-guide/ba-p/14384568
