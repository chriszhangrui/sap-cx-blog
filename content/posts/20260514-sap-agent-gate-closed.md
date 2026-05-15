---
title: "SAP把Agent大门关上了，Salesforce和ServiceNow却开着"
date: 2026-05-14T19:05:00+08:00
draft: false
tags: ["SAP CX", "Agentic AI", "API政策", "Salesforce", "ServiceNow"]
description: "SAP通过API政策禁止外部AI Agent调用接口，一切必须走Joule。而Salesforce和ServiceNow选择完全开放。谁的路线对？"
source_url: "https://www.techzine.eu/blogs/applications/141379/sap-blocks-external-ai-agents-salesforce-and-servicenow-dont/"
---

今天看到一条消息值得聊聊。

Sapphire 2026上，SAP把"Autonomous Enterprise"的口号喊得震天响。但在热闹的keynote背后，有一个安静的动作比任何新产品发布都更值得注意——SAP在API政策v4/2026的2.2.2条款里，白纸黑字写了一句话：**禁止外部AI Agent独立调度或执行SAP API调用。**

说白了就是：你的AI Agent想跟SAP系统打交道？必须走Joule。没有第二条路。

## 对面在做什么

同样是过去几周，Salesforce发布了Headless 360——所有数据、工作流、动作都通过API、MCP工具或CLI直接调用。超过60个MCP工具可以从Claude Code、Cursor或任何兼容运行时直接触发。不需要经过任何专属AI接口。

ServiceNow在Knowledge 2026上放出了Action Fabric：20年积累的工作流、审批链、业务规则，全部对外部AI Agent开放，通过REST API和MCP接入。同样不需要额外的LLM推理层。

![SAP autonomous enterprise](https://www.techzine.eu/wp-content/uploads/2026/05/sap-better-question-768x439.jpg)

两家的逻辑都是一样的：平台当执行层，AI选择权留给客户。

## SAP的"强制绕路"架构

SAP的CTO在Sapphire的Q&A环节说了一句很有意思的话：API是"过时的技术"，A2A（Agent-to-Agent）和MCP才是"现代的"。但实际上，SAP只支持外部Agent通过A2A协议与Joule通信。直接的API调用？法律层面已经堵死了。

结果就是：外部Agent每次调用都必须经历**双重推理**——你的Agent推理一次，Joule再推理一次。双倍延迟，双倍成本。这不是你可以选择绕开的设计，而是架构层面的强制要求。

SAP确实也在Q2推出了Integration Suite MCP Gateway。但Thomas Saueressig（Chief Customer Officer）在采访中说得很明白：这个网关只提供数据通道，不带Knowledge Graph和业务上下文。想要智能层？走A2A，走Joule。

说白了就是：MCP Gateway是个收费入口，卖的是你以前用直连API就能拿到的东西。

## 确定性问题

这里面还有一个技术上的硬伤。

Salesforce和ServiceNow的做法是：对于复杂的多步骤流程（比如信用卡争议处理），你可以定义一个确定性工作流，外部Agent直接触发它，每次执行结果一致。不需要LLM参与。

SAP的做法是：所有请求都得过Joule。Joule用Knowledge Graph理解你的意图，然后决定执行什么步骤。问题是——一旦有LLM推理参与，执行路径就不再是100%确定的。同一个请求，两次走可能走出不同的路。

![SAP Autonomous Suite](https://www.techzine.eu/wp-content/uploads/2026/05/autonomous-enterprise-suite-sap-450x279.jpg)

Saueressig自己也承认：对于确定性的规则流程，不应该用Agent，应该用标准自动化API。但问题是——如果触发动作的那个外部Agent本身就被政策禁止调API了呢？

## 数据说了什么

DSAG（SAP德语区用户组）2026年投资调查显示：

- **3%** 的SAP客户在生产环境使用Joule
- **77%** 的AI活跃SAP企业在用Microsoft Copilot

SAP现在把直连路径掐断了，逼那97%没在用Joule的客户必须经过Joule。这看起来不像是"让客户自愿选择Joule"的策略，更像是"让客户没法选别的"。

Saueressig反驳说这个调查样本不到100家不具代表性，同时表示68%的Fortune 500用了SAP AI。但"SAP AI"涵盖过去十年所有AI功能，不等于Joule。那1亿欧元的合作伙伴基金说明了真实的采纳压力。

## 似曾相识的剧本

2017年，Diageo因为Salesforce用户间接访问了SAP数据，被判赔5400万英镑。AB InBev也被SAP追讨过6亿美元。SAP后来在2018年推出了Digital Access模型作为妥协。

2026年，同样的模式在AI Agent上重演。

## SAP合作伙伴的私下态度

Techzine采访了参加Sapphire的SAP合作伙伴，他们用了"lock-in""walled garden""firewall"来形容这个策略。他们不愿具名——毕竟是在SAP的场子上。但他们的判断是：长期来看这个策略不可持续，因为行业大方向是相反的。

## 我的判断是

SAP的Knowledge Graph确实有独一无二的价值——50年的业务流程数据，财务、供应链、HR、采购深度互联，这个护城河是Salesforce和ServiceNow学不来的。如果Joule真的能提供不可替代的业务上下文，客户会自己来的。

但问题是，SAP选择了强制而不是吸引。如果你的产品真的好，为什么需要用法律条款把竞争者挡在门外？

对于我们做SAP CX的人来说，这件事的影响很现实：如果客户的技术栈里已经有了微软或Google的AI平台，他们想让自己的Agent直接操作SAP系统（比如自动创建订单、查询库存、触发服务工单），现在法律层面是不允许的。你必须解释为什么他们需要多过一层Joule，并且为此付费。

这个"解释"的难度，可能比任何技术集成都大。

参考来源：https://www.techzine.eu/blogs/applications/141379/sap-blocks-external-ai-agents-salesforce-and-servicenow-dont/
