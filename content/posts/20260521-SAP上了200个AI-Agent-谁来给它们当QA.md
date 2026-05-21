---
title: "SAP上了200个AI Agent，谁来给它们当QA"
date: 2026-05-21T12:10:00+08:00
draft: false
tags: ["SAP CX", "SAP Business AI", "Joule Studio", "Tricentis", "Solution Extension", "Cloud ALM"]
description: "Tricentis 把 Agentic AI Testing 焊进了 SAP Solution Extension。这不是又一个测试工具新闻，是自治企业叙事里第一颗真正落地的螺丝。"
source_url: "https://www.tricentis.com/news/tricentis-releases-agentic-ai-testing-sap-business-transformation"
---

所有人都在欢呼 SAP 在 Sapphire 发布了 50+ Joule Assistant、200+ 专项 Agent，铺天盖地的稿件都在讲"自治企业落地了"。但有件事没什么人提：这些 AI Agent 上线之前，谁来测试它们。Tricentis 上周丢出来的那个 Agentic AI Testing for SAP 公告，看起来像一条不起眼的合作伙伴新闻，其实是这一轮"自治企业"叙事里第一颗真正落地的螺丝。

不是夸张。SAP 现在的整套 Agent 架构是这么个结构——业务流程在 Signavio 里建模，Agent 在 Joule Studio 里编排，运行时由 Cloud ALM 调度。三层都是新东西，每一层都还在快速迭代。客户买了，就要把这三层焊到自己的 SAP 环境里跑业务。问题来了：原本一个 SAP CX 项目，UAT 阶段就能让甲方业务部门的人从早上吵到晚上——一个报价审批流程、一个订单履约逻辑、一个会员积分发放规则，每个细节都要测。现在你告诉我，这些流程不再是配置出来的硬规则，而是 AI Agent 自主决策跑出来的，那 UAT 怎么做？

## Solution Extension 这条路，SAP 走了快三十年

先说 Tricentis 这次发布的具体东西叫什么——SAP Enterprise Continuous Testing by Tricentis，AI 辅助自动化测试用例生成。这个名字很 SAP——前面挂着"SAP"开头，意味着它走的是 Solution Extension 这条路。SAP Solution Extension 是 SAP 跟伙伴产品深度绑定的一种合作模式，伙伴的产品挂上 SAP 品牌，由 SAP 销售、SAP 计费、走 SAP 的合同。这个机制 SAP 用了快三十年，从早年的 OpenText 到 Vistex 到 Tricentis，一直是 SAP 把生态里成熟产品"招安"进自己工具链的核心抓法。

这次 Tricentis 把它的 Tosca 系列里的 agentic test automation 能力嵌进了这个 SAP 标版里，跟原来的差别在哪？关键一句：测试用例的消耗走 SAP AI Units 计费。SAP AI Units 就是 SAP 给所有 Joule 和 Business AI 能力定的统一计量单位，谁调用 AI 谁烧 Unit。Tricentis 这次直接把测试用例生成接进了这套计量体系，意味着客户用 SAP ECT 跑 AI 测试，账单是从 SAP 那一侧出的。

> 这条线很关键。它意味着 Tricentis 不再只是"SAP 客户在用的第三方工具"，而是 SAP 自己定义的 Agent 工具链里负责质量层的那个固定位置。位置一旦焊死，就很难被换掉。

Karl Fahrbach（SAP 首席合作伙伴官）在公告里那句话其实点得很到位：这次合作扩展的是"AI 如何应用到关键质量保障流程"。换句话说，SAP 自己内部不打算造测试工具，把这块工程难度极高、又极其碎的活，交给一个有二十年专业积累的伙伴来扛。

## "测试 AI Agent" 比"测试软件"难得多

这事的难度被普遍低估了。传统软件测试的逻辑很简单——给定输入，期望输出，跑完看结果对不对。但 AI Agent 不是这样的。同样一句"帮客户排个 30 万订单的优先级"，今天 Agent 可能给出 A 方案，明天可能给出 B 方案，两种都"看起来对"——那到底哪个是 bug？

这是 SAP CX 项目里一个具体到要命的场景。比如 Sales Cloud V2 上线一个报价审批 Agent，老的流程是配置出来的——价格折扣超过 X% 走 Y 审批人，是死规则，UAT 跑几个边界用例就清楚了。换成 Agent 之后，它会综合客户历史合同、当前 pipeline、信用额度、季度目标做出判断。你测试什么？测试它每次都输出同一个结果？那 Agent 还要 Agent 干嘛。测试它"输出合理"？合理由谁定义？

Tricentis 这次给出的解法值得拆开看一下。一个是用自然语言生成端到端测试用例——这本身就要求测试工具能理解业务流程，而不是机械地点击按钮。第二个是"自愈测试"，意思是当 Agent 行为漂移、UI 变化、流程小调整时，测试脚本能自己改，不需要人工维护。第三个是把 Signavio 里的业务流程模型作为测试基准——Signavio 里有什么流程，测试就照那个流程跑，把 Agent 的实际行为跟流程模型对齐。

这个第三点是真正的杀招。它把 SAP 内部的两个产品线焊到了一起——Signavio 是流程，ECT 是测试，中间是 Cloud ALM 的变更影响分析。也就是说，从今往后想在 SAP 里做 AI Agent 的全生命周期管理，这条 Signavio→Joule Studio→Cloud ALM→ECT 的路径就是 SAP 钦定的范式。其他第三方测试工具想插进来，都要面对这条已经焊好的路径。

## 中国出海企业，要在哪几个动作上提前看

说回到具体场景。我接触过的中国出海客户里，做欧洲、东南亚、北美市场的不少都用 SAP CX，Commerce Cloud 跑 B2B/B2C 站点，Sales Cloud 管全球销售线索，Service Cloud 接售后工单。这两年明显能感觉到他们在做一件事——把"业务流程数字化"从 KPI 升到了"AI Agent 化"。但很多甲方 IT 负责人对一个问题一头雾水：项目上线后怎么验证 Agent 跑得对。

这里有几条建议是从 Tricentis 这次发布里能直接读出来的。

- **Signavio 不再是可选项**。以前不少出海客户在 SAP CX 项目里把 Signavio 当成"那个画流程图的工具"，要不要上看预算。现在不一样了——如果你的 SAP CX 落地路径里要用 AI Agent，没有 Signavio 里跑出来的流程基线，测试和治理都做不了。
- **SAP AI Units 的预算要单独算**。原来项目预算里 license 和实施费分得清清楚楚，现在多出来一项叫 AI Units 消耗——这是按调用量计费的运营成本，跟订阅费不在一个口袋里。Agent 越多用、测试越频繁，这一项越大。出海项目跨多个时区跑，自动化回归测试一晚上烧掉的 AI Units 不是小数。
- **Cloud ALM 的角色要重新定义**。过去做 SAP 项目，Cloud ALM 主要管运维监控。Agent 时代它是变更影响分析的中枢，决定了一次 Agent 行为调整会不会影响下游 50 个流程。出海项目最怕的就是欧洲业务调一次规则把美洲那边的 KPI 报表搞乱——这种事现在要靠 Cloud ALM 提前看出来。

还有个比较隐性的影响。SI 合作伙伴的报价模型可能要变。原来 UAT 阶段是按人天报的，业务顾问坐在客户现场，看着 QA 把测试用例一个一个跑。现在 Tricentis ECT 自动生成测试、自愈、自动跑，那这部分人天怎么算？是不是要从"人天"切到"测试覆盖率达标 + AI Units 消耗"的 outcome-based 模型？这个问题 SI 圈子里讨论过几轮了，但 Tricentis 这次发布给出了一个具体的工具锚点——之前没工具，这种讨论永远只能停留在概念。现在有了。

把这事拉回主线。SAP Sapphire 2026 上 200 个 Agent 一字排开，那个画面很爽。但 Agent 越多，Agent 之间互相影响、Agent 跟人工流程边界、Agent 行为漂移这些问题就越多。Tricentis 这次发布解决的不是"再多一个测试工具"，而是给 SAP 的 Agent 大军装上了 QA 这个角色。一个看起来不起眼的伙伴公告，背后其实是 SAP 自治企业叙事里最关键的兜底机制之一。

所以那个原始问题——"200 个 AI Agent 同时上岗，谁来当 QA"——SAP 已经把答案外包给了 Tricentis。问题是，作为客户，你愿不愿意把质量这条线一并交出去。这是接下来一两年每个 SAP CX 项目都要回答的题。

参考来源：https://www.tricentis.com/news/tricentis-releases-agentic-ai-testing-sap-business-transformation
