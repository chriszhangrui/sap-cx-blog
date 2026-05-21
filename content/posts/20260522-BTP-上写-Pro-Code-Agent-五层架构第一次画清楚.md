---
title: "BTP 上写 Pro-Code Agent，五层架构第一次画清楚"
date: 2026-05-22T10:00:00+08:00
draft: false
tags: ["SAP CX", "BTP", "技术深度", "AI Agent", "CAP"]
description: "SAP Architecture Center 5 月初更新的参考架构，把 CAP、Cloud SDK for AI、HANA Vector、Destination、A2A 与 MCP 五层关系第一次梳理清楚。这篇拆给架构师看。"
source_url: "https://architecture.learning.sap.com/docs/ref-arch/ca1d2a3e/3"
---

SAP Architecture Center 在 5 月初更新了一份 Pro-Code AI Agents on SAP BTP 的参考架构。这份文档不长，但把过去两年大家一直在争的一件事讲透了——当客户决定不用 Joule Studio 这种低代码工具、要自己写 Agent 时，BTP 到底给你哪几层东西可用。

这件事的背景比看起来更尖锐。Joule Studio 解决了 80% 的 Agent 场景，但剩下那 20%——多步条件分支、跨系统状态机、要接遗留 ERP 或者自研硬件——一旦不能配置就解决，Pro-Code 这条路就必须走通。问题是过去几年，BTP 上做这件事的人各搞各的：有人用 Cloud Foundry 跑 Python，有人在 Kyma 里塞 LangGraph，模型调用满天飞。这次官方架构图把所有零件归位，五层结构第一次画得清清楚楚。

## 先看一眼整张图

下面这张是基于原文重绘的版本，按从上到下的调用顺序梳理：

![五层架构](http://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLVaYG9ZurK1e0eSiaJqbe1xqgvic3CPIPCZf4NySJ4CSRcb53S87ichMWnL8EmqrwxBGpuiaicElB1lbOENRlQ5PDRrQB2P9ibLRt4hI/0?from=appmsg)

五层从上到下分别是：开发与编排（Dev & Orchestration）、AI 服务（AI Services）、数据与知识（Data & Knowledge）、连接与集成（Connectivity）、Agent 接口（Agent I/F）。这个分层不是 PPT 上画着好看，是真的对应了一次完整请求会经过的物理路径。

## 最上层：CAP 不只是写 Service 的

第一层有两个核心组件——SAP Cloud SDK for AI 和 CAP（Cloud Application Programming Model，云应用编程模型）。前者管和大模型对话，后者管业务逻辑的骨架。

很多人对 CAP 的印象还停留在写一个 OData 服务接前端这种用法。这次架构里，CAP 被定位成 Agent 用例的编排框架——它的领域建模能力被拿来描述业务对象，service.cds 文件被拿来声明 Agent 暴露给外部的端点，built-in 的 connectivity 直接对接 Destination Service。一段最小骨架大概长这样：

```javascript
// schema.cds
namespace agent.support;
entity Tickets { key id: UUID; subject: String; resolution: String; }

// service.cds
service AgentService {
  action invoke(query: String) returns String;
}

// handler.js
const { OrchestrationClient } = require('@sap-ai-sdk/orchestration');
module.exports = (srv) => {
  srv.on('invoke', async (req) => {
    const client = new OrchestrationClient({ ... });
    return await client.chatCompletion({ messages: [...] });
  });
};
```

SDK 那一边支持 Python 和 TypeScript 两套，可以挂 LangGraph、AG2（原 AutoGen）、CrewAI、Smolagents 等几乎全套主流 Agent 框架。SAP 这次没有自己造一个 Agent 编排器，而是认了开源生态——这是个值得停下来想一下的决定。

## 第二层：Harmonized Orchestration 是真正的护城河

第二层叫 AI 服务层，包含三块：Generative AI Hub、Harmonized Orchestration Service、Foundation Models。三块里 Harmonized Orchestration 是最容易被忽略、但实际最关键的一个。

为什么？因为它做了一件 OpenAI 直连做不到的事——把 Prompt Registry（提示词注册表）、Grounding（基于企业数据的 RAG）、Data Masking（数据脱敏）、I/O Filtering（输入输出过滤）做成了一个统一 API 层。换句话说，开发者写一行 chatCompletion 调用，背后已经自动完成了"PII 字段脱敏 → 注入企业上下文 → 调底层模型 → 出站结果再过滤一遍"这一整套流水。

底下挂的 Foundation Model 也很现实——Azure OpenAI、AWS Bedrock、Google Vertex AI 加 SAP 自建模型并存。这意味着合同上你跟 SAP 签一份，模型层底下走的是 Azure 还是 Vertex 不影响业务代码。对真正要做企业治理的客户来说，这一层比模型选型本身重要得多。

## 第三层：Vector Engine 在 HANA Cloud 里，不是单卖的

数据与知识层有两个关键组件——HANA Cloud Vector Engine 和 SAP Knowledge Graph。前者是向量数据库，后者是企业知识图谱。

这里的设计选择很有意思：SAP 没有让客户去外面买 Pinecone 或 Weaviate，也没有单独包一个向量数据库产品出来卖，而是把 Vector Engine 直接嵌进 HANA Cloud。这意味着 RAG 场景下，业务数据和向量索引可以在同一个事务里更新，不存在双写一致性问题。Knowledge Graph 走得更远——它把 SAP 标准业务对象（订单、客户、合同）的关系编码成图结构，让 Agent 可以用图查询而不只是文本检索来推理。

代价是什么？这条路把客户绑在 HANA Cloud 上，没法换底层数据库。这件事在做架构选型时要明牌讲清楚。

## 第四层：Destination Service 是隐藏冠军

连接层有 Connectivity Service（连本地系统的 Cloud Connector 通道）和 Destination Service（云端目的地管理）。后者长期被低估。

Destination 的本质是一个有凭证托管能力的反向代理。所有对 S/4HANA、SuccessFactors、Concur、Customer Experience、Business Networks 的调用，都写成对一个 Destination 名字的引用——OAuth Token、证书、mTLS 配置都在 BTP 后台维护。开发代码里完全看不到 secret。

在 Agent 场景下这个设计有个隐含好处：当 Agent 决定要调一个工具时，它不需要知道目标系统的认证细节，只需要拿一个 destination 名。把"谁能调什么"的权限治理从代码层面挪到了平台层面。这是企业 IT 安全部门会喜欢的解法。

## 第五层：A2A 和 MCP，方向相反的两个协议

最后一层只有两个东西，但分别代表 Agent 对外的两种角色：

- **A2A（Agent2Agent Protocol）**：Pro-Code Agent 暴露一个 A2A 兼容的 server endpoint，Joule 或者其他外部系统可以把任务派发过来。在这个协议里，你的 Agent 是被调用方。
- **MCP（Model Context Protocol）**：Pro-Code Agent 作为 MCP 客户端，去发现和消费外部 MCP server 暴露的工具。在这个协议里，你的 Agent 是调用方。

一个朝外暴露能力，一个朝内吸收工具。两个协议都是开放标准，意味着同一个 Agent 既可以被 Joule 编排，又可以反过来调用第三方厂商的 MCP 服务——比如 Salesforce 那边如果暴露了 MCP，理论上你的 BTP Agent 直接就能调。

## 部署：Cloud Foundry 还是 Kyma

原文里这一段比较克制，只说部署到 Cloud Foundry 或 Kyma runtime。但选哪个不是无关紧要的。

- **Cloud Foundry**：buildpack 模型，cf push 一行命令上线，对单体 CAP 应用最友好。运维心智成本最低。
- **Kyma**：基于 Kubernetes，需要 Helm Chart 或 Kustomize，但能拿到 K8s 全套生态——HPA 自动扩缩、Istio 服务网格、Knative 事件驱动。Agent 工作负载有突发性，HPA 在这里价值很大。

CAP 自身已经支持 cds add kyma 一键生成 Helm 模板。如果团队已经在用 K8s，Kyma 是更好的长期选择。如果团队对容器化没经验，Cloud Foundry 可以先把业务跑通。

## 几个落地建议

**什么场景适合走 Pro-Code 这条路**——业务流程涉及多步条件分支或循环调用、需要接非标准外部系统（自研中台、行业云、特殊设备）、对 prompt 调优和模型选型有精细要求、要做企业级错误处理（重试、熔断、可观测性）。这些场景用 Joule Studio 拖不出来。

**什么时候不要碰**——如果场景能用配置解决，就不要写代码。Joule Studio 在标准业务对象上的开箱体验比从头写 LangGraph 好得多。Pro-Code 的成本不在第一次开发，而在三年后的维护——CAP 升级、SDK 升级、底层模型换代，都要团队有能力跟上。

**避坑提醒三条**：

- Vector Engine 绑 HANA Cloud，签合同前要确认 HANA Cloud 的容量规划，向量索引会显著吃内存。
- Generative AI Hub 调用底层模型有 token 计费，开发阶段最好开 prompt cache，否则联调成本会失控。
- A2A 协议还在演进中，Joule 那一端的兼容性目前以最新版为准，跨版本测试要排进迭代节奏。

## 写在最后

SAP 在 Pro-Code AI 这件事上走了一条务实路线——不重复造 Agent 框架的轮子，把开源生态接进来；不另卖向量库，把它塞进 HANA Cloud；不强推自家模型，让客户用 Azure 或 Vertex 的底模也行。这个做法和 Salesforce 的 Atlas 思路差异挺大——后者倾向于把整套 Agent 能力封死在自家平台。

对做出海的中国企业、跨境运营场景、在华外资总部这些客户来说，BTP 这套架构的好处是治理边界清晰：Destination 管凭证、Orchestration 管模型治理、A2A 管对外暴露。如果接下来要在 SAP 生态里做自己的 AI Agent，这五层架构图值得贴在工位上。

参考来源：https://architecture.learning.sap.com/docs/ref-arch/ca1d2a3e/3
