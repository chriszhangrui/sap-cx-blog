---
title: "SAP承认了：Joule最大的瓶颈不在AI这边"
date: 2026-05-21T15:10:00+08:00
draft: false
tags: ["SAP CX", "Joule", "AI Readiness", "Clean Core", "Sapphire 2026"]
description: "Sapphire 上 SAP 把 Joule 推到第三阶段，叫 Joule Work——但同时也承认：客户准备度才是真正的瓶颈。Clean core、流程标准化、数据治理这些老地基，决定了你能不能赶上这班 AI 车。"
source_url: "https://erp.today/sap-joule-enterprise-execution-ai-readiness/"
---

SAP 已经把 Joule 推到第三个阶段了。

第一阶段是聊天框，回答几个问题；第二阶段是技能库，预制 2500 个 skills 让你点点点；第三阶段叫 Joule Work——Klein 的人在 Sapphire 现场给它的定位是"软件即结果"，不是软件即服务。

这话听着挺虚。但 Tarsilla Moura 在 ERP Today 上写的现场观察里，藏着一句让人停下来的话——SAP 自己人也承认：这个新阶段最大的问题不是 AI 能不能做到，是客户那边到底准备好没有。

![SAP Sapphire Opening Keynote](https://erp.today/wp-content/uploads/2026/05/SAP-Opening-Keynote-2026-1024x768.jpeg)

## 2500 个 skills 不够用，所以呢？

SAP 首席 AI 官 Jonathan von Rueden 在演讲里讲了一段挺直白的客户原话：

> "2500 个 skills 听起来很多，但根本覆盖不了我要干的活。我想直接跟我的 SAP 系统说话，跟所有数据打交道，让它帮我做完。"

这就是 Joule Work 出现的理由。它不再是预先写死的脚本集合，而是一个动态工作台——你说想要什么结果，它调用 SAP Knowledge Graph（SAP 自己说里面有 2 亿条事实和三元组），自己推理、生成、执行。

配套的还有 Joule Spaces，这玩意儿更激进——根据你当前的任务，临时生成一个工作界面。von Rueden 强调说，这不是那种用完就扔的 HTML，而是"可复现、安全、企业级的 UI"，能分享、能协作。

翻译过来：用户界面这个概念，SAP 准备从"固定应用屏幕"重构成"按意图临时生成的空间"。

## Joule 不只是一个聊天窗口了

现场公布的访问入口让人有点惊讶：

- **Joule Voice**：开车路上对着手机说"看一下那个销售订单的状态"，或者"帮我提交个请假单"
- **Joule Desktop**：本地装一个客户端，能连 SAP 后端、能读你的日历、能调用本地沙箱跑代码
- **Joule Studio**：放进同一个壳里，让有权限的用户自己搭 Agent

von Rueden 现场演示的一个场景是：从 CRM 拉客户数据→生成客户拜访简报→做成 PowerPoint→出一份费用分析→生成 PDF→挂上邮件发出去。这一整串动作，用户在 Joule Desktop 里说一句话就完成。

他说了句关键的话："员工现在期待 Joule 也有那种消费级应用的体验。"

这话本身不新鲜——所有 to B 软件都喊了十年。但这次不一样的地方在于，SAP 把 SAP 后端连接、Agent 协同、桌面权限控制焊在了同一个层里。这是它跟 Copilot、Gemini Enterprise 这些通用助手最大的区别。

![SAP Sapphire 2026 Opening Keynote](https://erp.today/wp-content/uploads/2026/05/SAP-Sapphire-2026-Opening-Keynote-1024x768.jpeg)

## 真正的瓶颈不在 AI 这边

愿景讲完，文章后半段切到 Sapphire 上更值得做项目的人琢磨的部分。

SAP 美洲区首席营收官 Jan Gilg 提到一个新组织叫 Customer Value Group。这个组的存在本身就说明问题——客户在跟 SAP 打交道时能感受到"接缝"：售前、售后、咨询、支持是断的。Gilg 自己的原话是"客户经常跟我反映，跟 SAP 做生意太累了"。

Sapphire 上 SAP 还公布了一个数据：超过 90% 的项目是由 SI 合作伙伴交付的。换句话说，SAP 自己造再多 AI 能力，能不能在客户那边落下来，最终取决于它的生态。

SAP 客户参与与采纳负责人 Thomas Pfiester 讲的一个对比挺有意思：

> "两年前，客户问的是要不要用 AI。今天他们说：我们已经跟技术对齐了，知道这东西能干嘛。问题是怎么从 PoC 走到真正在生产里跑、走到规模化使用。"

这个跃迁卡在哪？Pfiester 列了一串，没一个让人意外：

- 变更管理
- 用户接受度
- 对新工具的恐惧
- 流程标准化
- 数据质量
- SAP 系统和非 SAP 系统的集成

他原话："如果你要让 Agent 跑一个跨大流程的编排框架，这个流程必须是标准化的。数据是大问题，加上各种系统之间的集成点。"

## Clean Core 的故事被重写了

这里有个细节值得拎出来。

两年前 SAP 推 clean core 的时候，客户的反弹很大——"就为了升级快一点，我要把所有客制化都拆掉？这账算得过来吗？"

Pfiester 说，今天回头看，那些当时咬牙做了 clean core 的客户，现在反而在 AI 这一波里最从容。原因很简单：

- 流程是标准化的
- 数据是统一的
- 集成是规范的
- 技术债是清掉的

Joule Work 那种"动态生成工作台、Agent 跨系统跑流程"的能力，前提就是这四样东西到位。fragmented 的 SAP 环境跑不动这套东西——这不是技术能力问题，是地基问题。

所以 SAP 现在的话术变了：clean core 不是为了升级方便，是为了让你赶得上 AI 这班车。

## 这件事对中国出海企业意味着什么

中国制造往海外铺渠道、跨境电商做品牌全球化、在华跨国公司由总部牵头统一 Customer Experience 平台——这几类项目在过去两年里都遇到过类似的问题：技术选型早就定了，但落地节奏跟不上。

在 Sales Cloud V2、Service Cloud V2、SAP Engagement Cloud (Emarsys) 这些产品落地的项目里，一个反复出现的现象是：客户买了 V2，最后用得最多的是 V2 里那些跟原来 V1 行为一致的功能。新的 AI 能力——销售云的会议总结、服务云的 Agent Console、营销云的 AI Send Time Optimization——很多客户买了一年都没真正接进自己的业务流程。

原因往往不是产品不行，是上面 Pfiester 说的那几条：流程没标准化（每个区域销售自己一套打法）、数据没统一（CRM、ERP、电商各跑各的）、变更管理没做透。

如果 Joule Work 这一代 AI 真的要把"软件即结果"做成现实——动态生成工作台、跨系统编排 Agent——那对项目落地的要求只会比之前更苛刻，不会更松。

## 几个值得盯的信号

**Customer Value Group 的实际动作。** SAP 这次承认自己内部有"接缝"，这是过去几年没听过的话。如果这个组真的能让售前、售后、咨询、支持串起来，对接客户的体验会有实质变化。但组织变革的事，三年内能落地多少要打个问号。

**SI 合作伙伴的转向。** 90% 项目靠 SI 交付，意味着 Joule 落地的关键不在 SAP 总部的产品发布会，而在 SI 团队懂不懂 Agent 怎么部署、Knowledge Graph 怎么调、clean core 怎么从客制化里剥出来。生态的能力差距会被快速放大。

**从 PoC 到生产的转化率。** Pfiester 说的"客户都知道技术能干嘛了"——这话其实把一个事说穿了：所有人都做 PoC，但很多 PoC 永远停在 PoC 阶段。下半年值得观察的是 SAP 公开多少"PoC 转生产"的客户案例，而且是真上量的那种，不是 demo 跑通就发新闻稿的。

SAP 把 Joule 推成"企业新界面"是个很大的押注。但这次它学聪明了一点——同时也承认了：如果客户那边的地基没打好，这个界面再炫也没人能用上。

参考来源：https://erp.today/sap-joule-enterprise-execution-ai-readiness/
