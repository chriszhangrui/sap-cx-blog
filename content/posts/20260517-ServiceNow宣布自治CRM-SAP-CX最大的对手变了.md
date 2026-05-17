---
title: "ServiceNow宣布自治CRM，SAP CX最大的对手变了"
date: 2026-05-17T14:00:00+08:00
draft: false
tags: ["SAP CX", "ServiceNow", "Agentic AI", "CRM", "AI治理"]
description: "ServiceNow在Knowledge 2026上发布Autonomous CRM和AI Control Tower，要做所有AI Agent的交警。当治理变成竞争壁垒，SAP CX的护城河还在吗？"
source_url: "https://diginomica.com/servicenow-knowledge-2026-whole-darn-thing"
---

今天看到一条消息值得聊聊。

ServiceNow在上周的Knowledge 2026大会上，正式发布了一个叫**Autonomous CRM**的东西。注意用词——不是"AI-powered CRM"，不是"Smart CRM"，是Autonomous。自治。

说白了就是：ServiceNow不再满足于做IT工单那点事了，它要直接进来抢CRM的饭碗，而且一上来就打"AI Agent治理"这张牌。

这事为什么跟我们SAP CX有关？往下看。

## 治理，才是真正的战场

ServiceNow这次发布了一个叫**Control Tower**的东西——一个集中式的AI Agent管控平面。它能追踪企业里所有Agent的行为，不管这些Agent是谁家做的。

具体来说，三个能力：

- **Kill Switch**：监测到Agent异常行为时自动停用，等人工审查后再恢复
- **ROI Dashboard**：实时看每个Agent消耗了多少Token、带来了多少业务价值
- **Shadow AI监控**：检测未授权的Agent试图访问企业数据，直接禁用

还有一个叫MCP Registry的东西——一个私有的、经过审核的Agent连接目录。意思是AI Agent想调用外部服务？先在这登记、过审，才允许连接。

diginomica的分析师Rebecca Wettemann说了一句很到位的话：ServiceNow把自己定位成了"所有AI Agent的管控层"——包括竞争对手平台上造出来的Agent。

## 为什么ServiceNow现在敢进CRM

这个逻辑其实很简单。

过去两年，所有企业都在疯狂试验AI：买工具、起pilot、做Agent、在各种软件上叠AI。CEO和董事会逼着上进度。结果呢？大量的实验，极少的生产部署。

人们真正怕的是两件事：一是Agent失控——删数据、泄露信息、同意了不该同意的商业条款；二是账单失控——Agent半夜跑起来烧Token，第二天早上一看花了一大笔，什么也没干成。

ServiceNow赌的就是这个恐惧心理。它的策略是：你不一定要用我的AI Agent，但所有Agent都得从我这过——我是安检门。

当管控平台的角色立住了，再推自己的Autonomous CRM、HR AI Specialist、IT AI Specialist……替换掉各个垂直领域的点解决方案，就顺理成章了。

## SAP应该紧张吗

坦白讲，有点。

SAP在Sapphire上也推了**AI Agent Hub**，逻辑类似——集中管理、发现、治理所有Agent。SAP Knowledge Graph提供业务语义，Domain Model提供上下文。方向是对的。

但有一个关键差异：SAP的治理故事目前主要服务于自家的Agent生态。而ServiceNow一上来就说"我管所有人的Agent，包括你SAP的"。

这两个定位，差别很大。

Wettemann的判断是：谁先拿下治理层，谁就能决定哪些Agent干什么活、哪些厂商拿钱。因为治理平台有天然的信息优势——它知道哪个Agent表现好、哪个在烧钱、哪个有安全风险。推荐自己的Agent替代表现差的竞品，只是时间问题。

## 几个值得注意的细节

ServiceNow把自己的Build Agent嵌入了Cursor、Claude Code、GitHub Copilot——让开发者不用学ServiceNow就能在ServiceNow平台上构建。这招狠。降低了开发者迁移到ServiceNow生态的门槛。

另外，ServiceNow还把Sales and Order Management直接改名叫"Sales CRM"，把Customer Service Management改名叫"Service CRM"。连名字都不藏了——就是来打CRM这场仗的。

再加上之前收的Armis（安全）和Veza（身份治理），安全+治理+CRM+AI Agent的组合拳已经成型。

## 我的判断是

AI Agent时代的竞争格局跟SaaS时代不一样了。以前是功能比较——谁的Campaign管理更好、谁的Lead Scoring更准。现在变成了：谁是Agent的调度中心、谁定义Agent的行为边界。

ServiceNow从治理切入CRM，是一步很聪明的棋。因为企业买CRM，本质上不是买功能，是买对客户关系的掌控力。而在Agent时代，掌控力=治理能力。

SAP的反击窗口还在。SAP Business AI Platform + Knowledge Graph + Agent Hub的组合有深厚的业务语义优势——这是ServiceNow短期补不上的。但SAP需要快速回答一个问题：你的治理，是只管自家的Agent，还是敢管所有人的？

如果答案是前者，那治理层这个高地，可能就让ServiceNow先占了。

参考来源：https://diginomica.com/servicenow-knowledge-2026-whole-darn-thing
