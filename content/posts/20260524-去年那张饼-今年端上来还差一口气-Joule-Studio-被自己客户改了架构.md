---
title: "去年那张饼,今年端上来还差一口气 — Joule Studio 被自己客户改了架构"
date: 2026-05-24T10:00:00+08:00
draft: false
tags: ["SAP", "Joule Studio", "AI Agent", "SAP CX", "Knowledge Graph", "LangGraph", "Sapphire 2026"]
description: "SAP 自己 CPO 承认 Joule Studio 第一版客户采用率远低于预期,2.0 接入 LangGraph、AutoGen,GA 时间排到 2026 Q3。低代码红利在 Agent 时代失效了。"
source_url: "https://www.infoworld.com/article/4170661/saps-ai-promises-last-year-most-are-still-rolling-out-2.html"
---

## 去年那张饼,今年端上来还差一口气 — Joule Studio 被自己客户改了架构

企业软件这行有个老规矩:Sapphire 上画的饼,十有八九要拖到下一届才能真上桌。2025 年那一届,SAP 把 Knowledge Graph、Joule Studio、AI Agent Hub 三件套一起摆出来,承诺年底 GA。一年过去,Madrid 站上的复盘是另一个故事 — Joule Studio 的客户采用率"远低于我们的预期"。

这话不是分析师说的,是 SAP 自己 Business Suite 的 CPO Manoj Swaminathan 在媒体吹风会上原话讲的。InfoWorld 这两天把这段对话翻了出来,顺手把 SAP 首席 AI 官 Jonathan von Rüden 的解释也挖了出来。两个人讲的是同一件事:**去年那版 Joule Studio 走低代码路线,客户碰到稍微复杂一点的 Agent,工具就不够用了。**

### 问题不在功能,在路线选错了

von Rüden 的原话很直白:"我们走了 low-code 路线。能给扩展点、能挂工具,但你没法碰内核。"

这话翻译成中文就是 — Joule Studio 第一版,本质上是个"封装好的玩具"。SAP 想让业务侧的人也能拖拖拽拽搞 Agent,但真正要落地的客户带着大目标进来,一上来要的就不是拖拽:他们要清晰的审批节点,要硬规则,要能切进 Agent 主流程做控制。第一版统统给不了。

更要命的是 Agent Hub。这玩意儿去年 Sapphire 当成"统一编排中枢"来卖的,今年的官方说法是"正在做大规模重构,2.0 在路上"。一个产品 GA 不到一年就要重写,这事儿在 SAP 历史上不算新鲜,但放在 Agent 这个被 Salesforce、ServiceNow、Microsoft 三面夹击的赛道,时间窗口被压得很紧。

只有 Knowledge Graph 这一块,SAP 是真把它跑出了原本的范围。最初它是给 Joule Skill 当上下文用的,现在直接喂给 Agent,让 Agent"自己想清楚怎么动态调用东西"。这条线反而比预期走得更前。

### 重写的核心:把 LangGraph、AutoGen 接进来

2.0 版 Joule Studio 的改动,可以拆成三层来看。

- **第一层是开放性。**原来你只能用 SAP 自己的脚手架搭 Agent,2.0 直接接入 LangGraph 和 AutoGen。这是 Anthropic 和微软那边带火的两个开源 Agent 框架,SAP 之前死活不松口,现在认了。
- **第二层是 pro-code 入口。**允许开发者把自定义 Agent 接到自己的 GitHub。这等于把整个研发流程从 SAP 控制台搬出来,放回开发者熟悉的 IDE + Git 工作流里。
- **第三层是工作流原语。**审批节点、子 Agent、规则网关 — 这些去年没有的东西,2.0 全部内建。von Rüden 说"现在全焊在一起了"。

外面还有一个动作叫 Joule Desktop,这周一起发的。卖点是"个人用户不走 IT 也能搭自动化"。这一手很值得拆开看 — SAP 在赌一件事:**自上而下的 IT 集中部署速度太慢,不如让一线员工自己先跑起来。**

### 为什么 Joule Studio 第一版没跑起来

抛开市场叙事,这事儿还有一层更现实的解释 — **企业 AI 工具的低代码红利,在 Agent 时代失效了。**

低代码工具好用的前提,是用户的场景可以被一组预设组件穷尽。表单、流程、审批、CRUD,这些东西二十年下来已经被抽象得很清楚,业务人员拖拖拽拽能搭出 80% 的应用。但 Agent 不一样。Agent 的核心价值是处理"非结构化、模糊、多步推理"的任务,这恰恰是低代码组件最难封装的部分。

你给一个 Agent 配三个工具,它能跑;给它配 30 个工具加一组业务规则,低代码界面立刻变成噩梦 — 不是拖不动,是逻辑根本表达不清。SAP 的客户在试用过程中已经把这件事摸透了,所以 von Rüden 才会承认"客户带着大计划来,但需要硬规则和审批节点,我们的工具给不了"。

Joule Studio 第一版的失败,本质上不是 SAP 没做好,是整个行业对"业务人员搭 Agent"这件事过于乐观。Salesforce 的 Agentforce 1.0 同期遇到的问题几乎一模一样 — 第一版面向业务侧,第二版被迫加 pro-code 入口。

### 对 SAP CX 体系意味着什么

聊到这里,真正要问的问题是:这事儿对 CX 这条产品线影响有多大?

Sales Cloud V2 和 Service Cloud V2 是直接受益者。它们的 Agent 能力此前依赖 Joule Studio 第一版来做扩展和定制 — 比如客户想让 Service Agent 在处理工单时挂上自己 ERP 的库存查询规则,第一版能做但做得很别扭。pro-code + LangGraph 接入之后,这一层定制的天花板会被推得更高。

Engagement Cloud(原 Emarsys)和 CDP 受影响相对小。这两个产品的核心价值在数据和触达频次的精细化,Agent 编排不是它们的主战场,但当 SAP 把 Knowledge Graph 直接接给 Agent 的时候,营销侧的 Agent 调取多源数据会变快 — 这是个隐性收益。

Commerce Cloud 这边要看场景。如果客户要做的是 AI 选品助理或 Agentic Commerce 这种需要多步推理的场景,Joule Studio 2.0 的开放性会直接决定项目能不能落。这也是为什么 SAP CX AI Toolkit 最近一直在强调"和 Joule 体系对齐"。

但 Joule Studio 2.0 自己的 GA 时间是 2026 年 Q3。也就是说,客户现在能用的,还是那个被自己人吐槽过的第一版,或者得等三个月。

### 这一年最值得记住的一件事

SAP 这次媒体吹风的措辞很克制 — "minimal adoption"、"limited capabilities"、"big plans 但工具跟不上"。这种自我承认在 SAP 过去几年很少见。背后的原因不复杂:Sapphire 这种大会画饼太快,客户那边的 ROI 拷问太紧,**承认问题 + 给出 2.0 路线图,反而比硬撑去年的版本更有说服力。**

Ericsson、Mercado Libre、西门子被点名"已经在生产环境跑 Joule Agent",这三家是 SAP 用来证明"Joule 不是 demo"的招牌客户。但点名的同时也暴露了另一件事 — Sapphire 2025 上承诺的"今年底所有客户都能用上 Joule Studio",一年过去,真正跑起来的还是这几家头部。

下一个 Sapphire 在 2027 年 5 月。如果到那时候 Joule Studio 2.0 的客户名单还停在两位数,SAP 这盘 AI 棋的节奏就要重排了。

---

> 本文基于 InfoWorld 2026/05/22 报道二次创作,加入了对 SAP CX 产品线的影响判断。
