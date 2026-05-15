---
title: "AI Agent要上生产了，但谁来当安全员"
date: 2026-05-14T18:45:00+08:00
draft: false
tags: ["SAP CX", "SAP Business AI", "NVIDIA", "Agentic AI", "AI治理", "OpenShell"]
description: "所有人都在讨论AI Agent能干什么，但真正决定它能否进入生产系统的，是另一个问题：谁来管它。SAP和NVIDIA给出了一个答案。"
source_url: "https://blogs.nvidia.com/blog/sap-specialized-agents/"
---

大家都在聊AI Agent能做什么——自动跑采购审批、主动给客户发邮件、实时分析供应链异常。但有个问题很少被拿出来正面讨论：**如果一个Agent能动你的系统记录、能跨应用边界操作、而且不需要人类每一步都审批，你真的敢让它进生产环境吗？**

诚实地讲，大部分企业的答案是"不敢"。不是因为Agent不够聪明，而是因为没有人能告诉CIO：如果这个Agent做了一件不该做的事，我怎么知道它做了、怎么拦住它、怎么追溯？传统的chatbot时代那套权限控制——用户登录、角色分配、API调用限流——在Agent面前根本不够用。因为Agent不是在"回答问题"，它是在"执行动作"。执行动作的风险模型和生成回答的风险模型，完全不是一回事。

![SAP NVIDIA AI Agents](https://blogs.nvidia.com/wp-content/uploads/2026/05/logo-lockup-blog-SAP-1920x1080-1-1280x720.jpg)

**SAP和NVIDIA在Sapphire上给出了一个具体的技术方案。** SAP把NVIDIA的OpenShell——一个开源的安全运行时——嵌入了SAP Business AI Platform。这不是简单的"集成"，SAP的工程团队在和NVIDIA一起联合开发OpenShell的代码库，把企业真实需要的东西往里面塞：运行时加固、策略建模、企业身份集成、审计钩子。

OpenShell做的事情其实很具体：给每个Agent一个隔离的沙箱环境，文件系统层面和网络层面都有策略强制执行，Agent逻辑出了问题的时候有基础设施级别的隔离兜底。用大白话说就是——Agent可以自主行动，但它被关在一个透明的笼子里，笼子的规则是声明式的YAML文件，随时可以热更新。

但光有"能不能安全执行"还不够。这里有个非常精妙的分层设计值得说一下：

- **OpenShell回答的是：** "这个动作能安全地执行吗？"——这是基础设施层的安全。
- **Joule Studio运行时回答的是：** "这个动作应该被执行吗？"——这是业务层的治理。

这两层之间的区别，才是enterprise-grade真正的含义。一个Agent可能技术上有权限访问某个API，但从业务逻辑上它不应该在这个时间点、这个上下文里去调用。前者是infrastructure safety，后者是business governance。把这两层分开但又协同，是这个方案最值得关注的地方。

![Autonomous Enterprise](https://news.sap.com/wp-content/blogs.dir/1/files/2026/05/11/307298_GettyImages-2226298689_medium_uncropped_F-e1778589176742-540x310.jpg)

**黄仁勋把AI比作五层蛋糕：** 能源、芯片、基础设施、模型、应用。应用层在最上面，是AI创造经济价值的地方。SAP坐在应用层的核心位置——全球企业的财务、采购、供应链、制造流程都跑在SAP上。这就是为什么SAP的参与对企业级Agent落地如此关键：Agent要理解的不只是语言，是角色、流程、权限、数据边界。这些东西都在SAP的系统里。

还有一个细节值得注意：NVIDIA本身就是SAP的客户，它的财务、供应链和物流都跑在SAP上。所以这次合作不是两个厂商坐在实验室里假想企业需求——他们自己就是自己方案的用户。这种"吃自己的狗粮"的设定，让我对这个合作的落地可信度多了几分信心。

对于在SAP生态里做事的人来说，这件事的意义可能比又一个Agent功能发布要大得多。**Agent的能力上限大家都在追，但Agent能不能上生产，取决于安全和治理的下限在哪里。** SAP选择和NVIDIA一起把这个下限做成开源的、标准化的基础设施，而不是锁在自己的闭源平台里，这步棋下得挺有意思。

另外一个对SAP客户很实际的利好：NVIDIA的NemoClaw——一个构建和部署自主Agent的参考蓝图——会直接内置在Joule Studio里。这意味着开发团队不用从零搭建安全脚手架，有一条结构化的路径从初始构建走到可信的生产部署。

说到底，AI Agent能创造价值的前提是企业能信任它。而对大多数组织来说，最核心的数据就在SAP里——那是跑着整个业务的系统记录。在这些系统上放一个自主运行的Agent，不是一个可以随便试试的决定。SAP和NVIDIA这次合作的本质，就是在试图回答一个底层问题：怎么让Agent既能干活，又不失控。

参考来源：https://blogs.nvidia.com/blog/sap-specialized-agents/
https://news.sap.com/2026/05/secure-ai-agents-how-sap-and-nvidia-co-define-enterprise-grade-agent-execution/
