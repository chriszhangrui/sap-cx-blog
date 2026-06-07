# JDK 21 倒计时之外，SAP 把 Commerce Cloud 更新做成了产品

SAP 在 6 月初悄悄上线了一份新的 Commerce Cloud 更新指南，没什么大新闻气，但里面藏着一件被很多客户长期忽略的事——SAP 已经把 Commerce 的更新治理本身做成了产品。

这份指南的全名是 *Getting Up to Speed with SAP Commerce Cloud Updates*，由 SAP Readiness@Scale 团队发出，挂在 SAP Learning 上免费开放（要 S-user 和 Enterprise Support 订阅）。配套还出了一份 Sales Cloud 和 Service Cloud 的版本，结构一致。

## 从大版本到月度发布，门槛抬到哪里去了

Commerce Cloud 从 2211 版本开始已经切到 Continuous Innovation 模型——每月发版，里面是安全补丁、bug fix 和小功能增量。客户拿到的不再是隔一两年来一次的大版本升级，而是每个月都要过一道阀门。

这套打法听起来熟悉，因为 Salesforce 早就这么干。差别在于 SAP Commerce Cloud 是一个上下文比纯 SaaS 重得多的系统：客户大量自定义 Java 代码、自建 storefront、深度集成 ERP 和支付。每月一推，意味着客户的 IT 团队也要每月做一次回归。这是一个隐性成本，过去常被低估。

## JDK 21 这道闸门，时间已经写在墙上

指南把 Framework Update 拎出来单独讲了一节，重点是从 JDK 17 迁到 JDK 21 的路径。背后跟着的不只是 Java 版本号——Spring 6.x、OAuth 实现、Drools 10 全部要跟着动。

**2026/08 是 JDK 21 切换的截止时点，之后只发 21 构建。**

这意味着所有还在跑 JDK 17 的客户，今年三季度之前必须搞定测试和上线。Spring 6 的非兼容变更、Drools 10 的规则引擎升级、OAuth 流程的差异，每一项都可能踩坑。出海企业里那些把 Commerce Cloud 跑了三五年的客户，扩展代码越多，迁移测试周期就越长。

## 指南拆出的四件事，真正值得看的是后两件

指南把 Commerce Cloud 的更新治理拆成四个动作：理解月度发版机制、掌握 Framework Update 路径、用好 Release Navigator、参与 Continuous Influence Program。

- 前两件是基本功——任何在 SAP 平台上跑生产的团队都该会
- 第三件是 Release Navigator——SAP 把发布信息从分散的邮件、Help 站点收拢到一个中央台，给运维和架构提供可订阅的变更流
- 第四件最反常识：Continuous Influence Program 让客户能投票、提需求、报名 Beta，把路线图从 SAP 单方面推变成双向通道

在 Commerce Cloud 的世界里，SI 和客户技术团队过去很少把"影响路线图"当成一项治理动作。指南把它写进四个标准动作里，本身就是一个信号——SAP 想让大客户的反馈成为发版优先级排序的输入，而不是被 PM 选择性听取。

## Sales Cloud 和 Service Cloud 同步发了一份指南，节奏也开始一致

指南页面最后挂了一条——Sales Cloud V2 和 Service Cloud V2 也有同款的 e-learning。这不是凑数。Sales / Service Cloud V2 的发版周期向来比 Commerce Cloud 更激进（季度发版 + AI 能力大量增量上线），客户做 V2 落地时更需要看清"什么时候推、什么时候验证、什么时候能用"。

把三朵云的更新治理用同一套教材规整起来，意味着 SAP 的 CX 产品组合在运维语言层面正在收敛。这对中国出海企业里同时跑 Commerce + Sales/Service 的客户来说，是一个正面信号——不用再为每朵云写一套不同的运维 SOP。

## 几个隐性影响

在 Commerce Cloud 项目里，"什么时候升级"基本都是临时决定——出问题了再排，发了 hotfix 再补。Continuous Innovation 模型把这个节奏从被动变主动，但前提是客户的 IT 团队必须把回归测试做成常态化产线，而不是项目制运动战。这件事很多甲方还没准备好。

第二件事——JDK 21 截止线压在 8 月，对那些在 SI 合作伙伴手里跑了多年扩展代码的客户尤其紧。出海电商里有大量 storefront 自建客户，扩展点深、依赖第三方库多。如果 SI 的资源不到位，三季度集中迁移会撞车。

第三件事可能更隐蔽——SAP 把 Continuous Influence Program 摆进核心运营动作里，等于在告诉大客户：你的反馈是被收的，但前提是你要按指南那套机制反馈。这是一个治理姿态。Commerce Cloud 里那些做得好的中国出海客户，最近一两年在 Beta 项目和 idea 投票里其实已经能见到名字。

SAP 这次没有发新功能、没有发新产品。它发的是一份"怎么用产品"的指南。但这件事本身值得记一笔——在一个所有人都在喊 AI Agent 的季度，把基础工程治理重新摆上桌面，本身就是一种克制。

> 参考来源：https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/getting-up-to-speed-with-sap-commerce-cloud-updates/ba-p/14406755
