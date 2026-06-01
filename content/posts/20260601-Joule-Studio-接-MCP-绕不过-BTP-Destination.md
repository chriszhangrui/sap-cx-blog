---
title: "Joule Studio 接 MCP，绕不过 BTP Destination"
date: 2026-06-01T17:05:00+08:00
draft: false
tags: ["SAP CX", "AI", "Joule", "MCP", "BTP", "技术深度"]
description: "外面挂 MCP Server，里面 Joule Agent 想用，中间必须经过 Destination Service 这一道关——而且每一项配置都有一个失败点"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/part-3-workflow-n8n-integration-with-joule-studio/ba-p/14392294"
---

做过 Joule Studio Agent 项目的都遇到过同一个时刻——配 MCP Server 时找不到「外面那台」。明明 n8n 那边 MCP endpoint 起好了，token 也拿到了，回到 Joule Studio 的 MCP Server 选择器里却空空如也。

这不是 bug，是设计。Joule Studio 不直接连任何外部服务，所有出站调用都必须经过 BTP Destination Service。这一脚必经之路，决定了 Joule 在企业里能不能跑得稳——但也制造了几乎所有人第一次都会踩到的几个坑。

最近 SAP Community 有一篇 n8n + Joule Studio 集成的实战记录，把这条链路的每一段都摆出来了。值得拆开看看。

## 一、为什么是 Destination Service

把 MCP Server 挂在 BTP Destination 后面而不是让 Joule Studio 直连，这个选择本身就值得讨论。直接连有直接连的好处：少一跳延迟，配置更短。但 SAP 没这么做。

理由有三条，按企业落地的优先级排：

- **凭证集中**。Bearer token、OAuth client、mTLS 证书全部托管在 Destination Service 里，Agent 自己看不到也拿不到。换 token 不需要改 Agent，换 endpoint 不需要重新部署。
- **子账户隔离**。Destination 是子账户作用域，多租户 SaaS 场景下每个客户子账户独立维护自己的 MCP 端点，不会串。
- **统一审计**。所有出站流量过 Connectivity Service 一次，日志、限流、IP 白名单都能在 BTP 这一层统一加。

代价是：Destination 配置项里塞进了 MCP 专属的 metadata。这就是接下来那两条 Additional Properties 的由来。

## 二、那两条 Additional Properties 才是关键

Destination 本身的字段——Name、Type、URL、Authentication——任何对接过 BTP 的人都熟。但 MCP 这条链路的核心信息不在这些字段里，而在 Additional Properties 里：

```
sap-joule-studio-mcp-server = true
URL.headers.Authorization = <n8n bearer token>
```

第一条 `sap-joule-studio-mcp-server: true` 是 Joule Studio 识别这条 Destination 的开关。少了这一条，Joule Studio 的 MCP Server 选择器会跳过这条记录——子账户里几十条 Destination，它只显示打了这个标的。

第二条把 token 注入到出站请求的 Authorization 头里。这里有个反直觉的点：Authentication 字段必须选 `NoAuthentication`，不能选 OAuth2ClientCredentials 或者 BasicAuthentication。token 不走标准 auth 通道，而是走 header 注入。

为什么？因为 MCP 是个新协议，BTP Destination 的标准 auth flow 不认它的握手方式。Header 注入是最直白的兜底——但你必须放弃 Destination 的 token 自动刷新能力。这就是设计取舍：开了一个口子让新协议跑起来，代价是凭证生命周期管理你自己背。

## 三、n8n 那边发生了什么

n8n 自己内置了 MCP Server——这件事如果不动手做一遍，光看官方介绍是看不出来的。它的逻辑是：所有带 Chat Trigger 节点的已发布 workflow，会被自动登记成 MCP 工具，对外暴露一个统一 endpoint。

路径是固定的：

```
https://<your-kyma-domain>/mcp-server/http/
```

末尾那个斜杠不能丢。n8n 的路由严格匹配，少一个字符会直接 404，但前端不会给你一个明确的错——只会显示 tool list 为空，让你以为是别的环节坏了。这是这条链路最隐蔽的失败点，没有之一。

Joule Studio 侧需要把这个路径填到 MCP Server 配置的 Advanced Options → Path 字段里，而不是直接拼到 Destination 的 URL 上。Destination URL 只放到 Kyma 域名为止，路径在 Agent 配置层才补齐。这种分层有意而为之：同一个 n8n 实例可以暴露多套 MCP endpoint，每个 Agent 挂自己那一套。

## 四、Workflow 没 Publish，全链路白搭

整条链路最末端的失败点反而最反常识——n8n 那边 workflow 必须点 Publish。

没发布的 workflow，Chat Trigger 不激活，n8n 的 MCP Server 不会把它列进工具表。Joule Studio 这边连得上、token 对、path 对，拉到的 tool list 还是空的。然后你开始怀疑 Destination 配置、怀疑 token、怀疑 BTP 子账户权限，绕一圈才发现是 n8n 编辑器右上角那个按钮没点。

这个失败模式的根本原因是 MCP 的 discovery 阶段——Joule Studio 连过去不是直接调工具，而是先调一次 list_tools。n8n 把「未发布的 workflow」从这个列表里过滤掉了。整套机制是合理的，但端到端排错的时候，你需要知道这一层过滤存在。

## 五、Joule Agent 这一头的几个细节

Agent 创建本身没什么悬念，三段式配置：Identity（Name + Description）、Expertise（Persona + Instructions）、Model + MCP Server。值得提的是 Instructions 字段——它不是简单提示词，而是会影响 Agent 决定要不要调工具、调哪一个工具的关键。

一个常见错误是把 Instructions 写得太具体——「先查 ticket，再查 KB，最后查 employee skill」——结果 Agent 失去了组合工具的灵活度。MCP 的整个设计前提就是让 LLM 自己决定调用顺序，把流程硬编码进 prompt 等于把 Agent 退化回脚本。

正确的写法是定义角色和边界：

```
Role: IT Helpdesk Assistant
Domain: Internal IT support — tickets, KB, skills lookup
Boundaries: 不要回答 IT 范围之外的问题；不要编造 ticket 编号；用户身份未知时优先问
Tone: 简短、直接、可操作
```

## 六、什么样的项目可以用，什么时候别碰

跨境/出海企业的 IT/客服/销售场景里，这种「Agent + MCP + 现有自动化平台」的组合很有想象空间。比起在 ABAP 里写 BAdI 或者在 BTP 上从零搭一套 CAP 服务，把已有 n8n / Make / Zapier 工作流直接挂成 MCP 工具的成本要低一个量级。

适合上的场景：

- 已经在跑 n8n 或类似低代码平台，工作流稳定、有维护人，只缺一个对话入口
- 跨多系统的轻量编排（OData 拉数 + 邮件 + Slack 通知这种），用 Agent 比写硬编码脚本灵活
- 内部工具类场景——HR 助手、IT 助手、Onboarding 引导——用户量可控，幻觉成本可承受

不要碰的：

- 写库类操作。MCP 工具是 Agent 自主决定调用的，没有人工审批关卡。涉及订单创建、金额变更、用户权限调整这种，写一道事务确认是底线
- 依赖严格 SLA 的实时业务。Joule Studio → Destination → n8n → 后端 OData 这条链路有四跳，p99 延迟难以做承诺
- 数据驻留敏感的场景。n8n 跑在 Kyma 上没问题，但跨境企业要确认 Kyma 实例所在区域是否符合数据保护要求——这一层 Joule Studio 自己看不见

## 七、几个落地建议

做实施前准备这几件事：

- Destination 起名加前缀，比如 `mcp-prod-helpdesk`、`mcp-dev-helpdesk`。Joule Studio 选择器按字典序排，前缀让多环境管理省心
- n8n 的 token 写进 BTP 的 Credential Store 而不是直接贴进 Additional Properties。生产环境必须做
- MCP 工具描述要在 n8n workflow 那边写清楚——Agent 选工具靠的就是这段描述。糟糕的描述等于让 LLM 瞎猜
- 先用 Joule Studio 的 built-in test 跑通，再开放给业务用户。这一步 90% 的问题都能在测试聊天里暴露出来

---

MCP 协议本身简单——它就是个让 LLM 发现并调用外部工具的标准。但 SAP 把它装进 Joule Studio 的方式，体现了企业级 AI 平台的一贯思路：协议层不是问题，治理层才是。Destination Service 那一脚必经之路，看起来是一道阻力，实际上是企业 IT 唯一能拿到的控制权。

Joule 的 Pro-Code 路径开始走通了。下一关，是把工具调用的可观测性、审批流、配额管理这些治理面也补齐——这条路还长。

---

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/part-3-workflow-n8n-integration-with-joule-studio/ba-p/14392294
