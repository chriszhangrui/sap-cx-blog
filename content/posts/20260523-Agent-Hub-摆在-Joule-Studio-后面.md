---
title: "Agent Hub 摆在 Joule Studio 后面"
date: 2026-05-23T13:10:00+08:00
draft: false
tags: ["SAP CX", "AI", "技术深度", "Joule", "Agent Hub", "Signavio", "LeanIX"]
description: "Joule Studio 是表象，AI Agent Hub 也是表象，真正的护城河是流程图加架构图——SAP 这次的 Agent 治理走了一条不太一样的路。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/best-practices-for-ai-agent-lifecycle-management-with-joule-studio-and-the/ba-p/14393770"
---

SAP 在 Sapphire 2026 上密集发了一堆 AI 公告，最容易被忽略的反而不是 Joule Studio 本身——而是它身后那个 SAP AI Agent Hub。

很多人第一反应是：又一个目录？做项目这两年装过的 Agent 注册中心、模型市场、提示词仓库已经不下五个，每一个起点都说要"统一治理"，落地两个月就变成又一个孤岛。这次有什么不同。

把 Joule Studio 和 AI Agent Hub 放在一张图里看，会发现 SAP 这次解决的不是"在哪里管 Agent"，而是一个比这个更前置的问题——**Agent 在被构建出来的那一刻，是不是已经知道自己要在哪个流程上跑、不能碰哪些系统、要被谁审计**。

## 为什么 Agent 治理大多失败：自下而上

先讲企业 AI Agent 落地最常见的一个坑。绝大多数客户的第一批 Agent 都是某个业务部门拍脑袋自建的：销售部门做了一个查报价单的 bot，客服部门做了一个总结工单的 assistant，财务做了一个发票分类器。每个都跑得不错。

问题在第二年来。Agent 多了之后没人知道：

- 全公司一共部署了多少个 Agent，谁在维护
- 这些 Agent 调用了哪些后端 API、有没有越权
- 用了哪些 LLM、是否符合数据合规要求
- 实际效果到底改善了哪个流程的哪个 KPI

SAP 把这个状态叫 fragmentation——不是技术问题，是治理结构问题。Agent 自下而上长出来，没有流程上下文，没有架构对齐，没有衡量手段。

这正是 Joule Studio 和 AI Agent Hub 这次组合要打的位置。

## 真正的抓力：把流程图与架构图喂给 Agent

先看整体架构：

![Joule Studio × AI Agent Hub 架构图](http://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLVmkPT1xbVKqibM4GOUiclNjD5Pp8eEEpBaICaFVJM3nBqtURyDGRgd6bBe4Z6UiaV2MvE7x52kwrJjRIY93pt9uFIn3qf3BS8eicw/0?from=appmsg)

Joule Studio 是 build-time，AI Agent Hub 是 runtime governance。两者之间的关键不是工具链顺序，而是**有两条上下文输入贯穿了 build 与 run**：

- **SAP Signavio 提供流程上下文**——告诉 Agent 这个企业里到底有哪些瓶颈、哪些 KPI 需要改善、流程里出错最多的环节是哪几步
- **SAP LeanIX 提供架构上下文**——告诉 Agent 公司有哪些应用、哪些接口、哪些数据流，以及哪些是 clean core 不能动的

这两条线的意义在于：当架构师或者业务用户在 Joule Studio 里用自然语言写下"做一个能帮销售判断订单交付风险的 Agent"时，Joule Studio 不是直接喂给 LLM 让它发挥——而是先去 Signavio 拉取订单交付流程的当前 KPI 和异常点，再去 LeanIX 查这个流程依赖哪些系统、哪些接口可以暴露、哪些字段属于敏感数据。

这之后再生成 PRD、技术规范、Agent 逻辑、工作流，最后部署到 Joule Studio 自带的 runtime。SAP 给这个流程起了一个名字叫 intent-based development，本质上是**把架构治理前置到了 Agent 生成的第一秒**。

SAP 自己的口径里举了一个数字——某客户的端到端 Agent 从想法到部署 10 到 15 分钟，原本要 3 到 4 天的人工开发与协调。这个数字本身可能有点宣传味道，但更值得注意的是它隐含的一件事：**原来那 3 到 4 天里，绝大多数时间不是写代码，是协调架构合规和找业务上下文**。Joule Studio 把这部分压缩掉了，因为上下文不再靠人去问。

## SAP AI Agent Hub：Agent 的"工商登记处"

Agent 部署完之后进入 SAP AI Agent Hub 这一层。这里是几个能力：

- 跨 SAP 与非 SAP 环境的 Agent 发现——也就是说不光自己 Joule Studio 建的，第三方建的 Agent 只要注册都能纳入
- 中央的 Agent / LLM / MCP Server Registry，把"谁在用什么模型、调用什么工具"变成可查询的元数据
- 把 Agent 链接到具体业务能力、流程、应用、数据上——这一步是和 LeanIX 共用一份架构 graph 来做的
- Agent 调用链的执行追踪与可观测性，类似分布式追踪但是面向 Agent 行为
- Agent 与 MCP server 的风险与合规属性核验

关于 MCP 的部分值得单独说一句。MCP（Model Context Protocol）现在是 Agent 接外部工具的事实标准，但它的安全模型其实很弱——任何 MCP server 都可以注册、暴露任意工具。SAP AI Agent Hub 把 MCP server 也纳入了 Registry，给每个 server 打风险标签，这意味着企业内部使用的 MCP 工具列表是被治理的，不是想接就接。这一点对做 SI 项目的人来说比较关键，过去给客户接第三方 MCP server 走不走 IT 审批是个灰色地带。

把它类比成一个东西的话——AI Agent Hub 之于 Agent，类似于 API 管理平台之于 API。十年前每家公司的 API 都散在各团队，后来 Apigee 和 Kong 这类东西把 API 收编成可治理资源。Agent 现在走到了同一个十字路口。

## 反馈回路里最容易被忽略的一段：Agent Mining

Plan、Build、Discover、Govern 这四步如果是一条直线，那这套架构和市面上其他 Agent 平台差别不大。SAP 这次画的是反馈回路——关键在最后一段：**Agent 的运行行为被转成事件流，喂回 Signavio 做 process mining**。

这步叫 agent mining。意思是把 Agent 的执行日志当成业务流程事件来分析，能拿到这几个东西：

- 每个 Agent 的实际 cycle time 改善幅度
- Agent 行为是不是在偏离预期（behavioral drift）
- Agent 的动作和端到端流程之间的因果关系
- 哪些 Agent 应该被优化、哪些应该被下线

这件事单独看可能觉得是花活，但结合 SAP 拥有 Signavio 这个事实就比较实在——**SAP 是少数几家手里同时握着流程挖掘平台、架构图谱平台、AI 平台和业务系统数据的厂商**。Salesforce 的 Agentforce 想做这条反馈回路还得自己拼一套流程挖掘出来；Microsoft 走的路线是 Copilot Studio 加 Power Platform，但流程那一层是空的。

## 代码视角：Agent 注册的形态

从架构角度看，Joule Studio 和 AI Agent Hub 之间的注册关系大致是这样——一个 Agent 部署元数据看起来类似（这是基于 SAP 文档的概念示意，具体字段以官方为准）：

```yaml
# Joule Studio agent definition (concept)
agent:
  name: order-delivery-risk-agent
  intent: "评估销售订单交付风险并推送给责任人"
  context:
    process_ref: signavio://process/order-to-cash/delivery-risk
    architecture_ref: leanix://capability/order-fulfillment
  llm: claude-sonnet-4
  tools:
    - mcp://sap-s4hana/sales-orders
    - mcp://sap-bdc/delivery-history
  governance:
    risk_class: medium
    data_sensitivity: customer-pii
    owner_team: sales-ops
```

这里关键的不是 YAML 长什么样，而是**每个 Agent 都被强制带上 process_ref 和 architecture_ref**——也就是说一个 Agent 没法脱离它要跑在的流程和架构存在。这是这套设计和那些"Agent 即工具"的轻量平台最大的区别。

## 这个架构放弃了什么

没有架构是只解决问题不付代价的。这套组合的代价也很明确：

- **对 Signavio 和 LeanIX 形成事实绑定**。没买 Signavio 的客户用不到 process context，没用 LeanIX 的客户拿不到 architecture context，那 Joule Studio 的 intent 生成质量就大幅退化，剩下的就是一个普通的 low-code Agent builder
- **流程数据的口径必须先治理好**。Signavio 里的流程图如果是几年前画的、和现在的 S/4 实际跑的流程对不上，喂给 Agent 的 context 就是垃圾
- **对企业架构治理成熟度要求高**。LeanIX 里架构没维护好，"clean core 不能碰"这件事对 Agent 来说就只是一句空话

## 谁该现在动手，谁应该再等等

出海企业和跨国制造里，已经在用 S/4HANA Cloud + Signavio 做端到端流程梳理的客户，现在是最适合切进去的——这套架构本质上是在你已有的流程图谱上叠 Agent 层，前期投入早就摊掉了。Sales Cloud V2 和 Service Cloud V2 项目里如果客户提 AI agent 需求，第一步应该是查一下他们 Signavio 流程模型的覆盖度，没覆盖的部分先补流程，再谈 Agent。

在华外企的总部对接场景里，如果总部已经在 LeanIX 做架构治理，那中国分公司接 Joule Studio 的整合阻力会比想象中小——架构上下文是从总部继承下来的。

不建议现在动手的情况：业务流程还在每季度大改、Signavio 流程图覆盖度不足三成、LeanIX 应用清单从来没维护过。这些场景下硬上 Joule Studio + AI Agent Hub，最后会变成"用最贵的工具堆出最普通的 chatbot"。

还有一类陷阱要警惕——把 AI Agent Hub 当成一个独立的"Agent 集中营"来卖给客户。这个产品脱离 Joule Studio 单独存在意义有限，单买它的客户大概率最后只用了 Registry 的 5%，治理那部分根本起不来。Agent 治理是建立在 Agent 是怎么生成的之上的，build-time 和 runtime 必须一起谈。

## 写在后面

SAP 这次在 Agent 这件事上选的路其实挺克制——没有去抢 LangChain 那种通用 Agent 框架的位置，没有去做又一个 Copilot 类对话入口，而是把自己最值钱的资产（流程挖掘平台 Signavio、架构图谱平台 LeanIX、业务系统数据）拼成 Agent 的 context source。

Joule Studio 是表象，真正的护城河是流程图加架构图。Agent Hub 是治理工具，但能治理什么取决于这两张图有多准。

Agent 不会缺，缺的是知道企业在哪里、流程在哪里、边界在哪里的 Agent。这条逻辑这两年大概率会越来越清楚。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/best-practices-for-ai-agent-lifecycle-management-with-joule-studio-and-the/ba-p/14393770
