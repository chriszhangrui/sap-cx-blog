---
title: "Salesforce 自己出来打补丁，这扇门 SAP CX 没开"
date: 2026-05-31T00:15:00+08:00
draft: false
tags: ["SAP CX", "Salesforce", "Joule", "MCP", "Headless"]
description: "Salesforce 的 Headless 360 上月才发，CMO 这周就出来澄清说客户都听错了。这场风波背后藏着一个所有 CX 平台都要回答的问题——AI Agent 时代，到底要开门还是关门。"
source_url: "https://diginomica.com/have-salesforce-customers-been-heading-down-wrong-path-their-idea-headless-360-cmo-patrick-stokes"
---

大家都觉得 Salesforce 上个月那波 Headless 360 是个划时代的开放姿态——把 CRM 的"头"砍下来，让客户随便接 Slack、Claude、ChatGPT。结果上周 Salesforce 自己开完季度财报，CMO Patrick Stokes 在分析师电话里出来澄清了一句很尴尬的话：客户其实理解错了，我们没有要把应用层去掉。

这事的有意思之处不在于 Salesforce 自己打补丁，而在于它把这两年企业 CRM 平台最关键的那个分歧，第一次摆到了台面上。把这场风波放到 SAP CX 这边来看，结论可能跟很多人想的相反——Salesforce 其实不是要追上 SAP，是想"修正"自己之前用力过猛的那一脚。

## 「Headless」三个字到底惹出了什么麻烦

Stokes 在电话会上原话是这样的——"问题大概出在『Headless』这个名字本身吧，它确实容易让人理解成把头砍掉了。"Benioff 当时在台上还有点不解："为什么大家觉得 Headless 就意味着不再有应用、不再有 Salesforce App 了？"答案其实就藏在他自己发的那条爆火推特里。技术圈过去一提 Headless，第一反应都是 commercetools、contentful 这种纯 API 优先的电商或 CMS。Salesforce 借这个词，想说的是另一回事：把 Sales、Service、Commerce、Marketing 的能力通过 MCP server 暴露出来，你既可以继续用我的 App，也可以从 Slack、Claude、ChatGPT 任何地方调用。

这两个意思中间隔着一道很大的鸿沟。前者是放弃头部 UI 这层产品价值，后者是给已经付了钱的客户多开几个调用入口。Salesforce 显然是后者，但客户听成了前者。

![Patrick Stokes](https://diginomica.com/sites/default/files/images/2026-05/Screenshot%202026-05-28%20at%2011.58.17.png)

## CRO 拿出来举证的两个客户

Salesforce CRO Miguel Milano 在同一场电话里举了两个例子来证明这条路是对的。一个是欧洲人力公司 Adecco，他们已经在用 Salesforce 跑大量招聘 Agent，现在正往 Voice 走。Headless 360 一发，Adecco 立刻打电话过来："等等，是不是说我们在 Agentforce 之外、用别家 AI Lab 搭的那些 Agent，也可以接进 Salesforce 的数据了？" Salesforce 的回答是："对，就是为这个做的。"

另一个例子更耐人寻味——Anthropic。Anthropic 自己是 Sales Cloud 和 Slack 的大用户，Q1 它的 Salesforce 用量同比涨了 5 倍，理由是它的销售团队不再"打开 Sales Cloud 这个 App"，而是从 Slack、从 Coworker、从其他工具里调用 Sales Cloud 的能力。换句话说，AI 公司自己在用脚投票：我不要打开你的界面，我要从我的工具里捅一根 API 进去把你的数据捞出来。

这两个例子放一起看，Salesforce 真正想要的，其实是把 CRM 应用从"客户每天打开的那个网页"变成"散落在企业各种工具底层的一组 MCP 端点"。这是一个很务实的战略转向，但它把过去十年 Salesforce 自己反复强调的"Customer 360 统一界面"叙事给反着推了一遍。所以客户听岔了不奇怪——Benioff 自己讲的故事本来就是矛盾的。

## SAP CX 这条路是怎么走的

如果你现在让一个评估两家平台的中国出海企业 IT 负责人画图对比，会发现一件挺反直觉的事：在外部 Agent 接入这件事上，SAP 几个月前还公开过保守姿态——上次 SAP 把 Joule Agent Hub 开出来时，外界注意到 SAP 设了准入机制：要"上户口"才能进来调用，还跟 NVIDIA 一起搞 Agent execution 的安全规范。Forrester 在 Sapphire 之后专门写过一篇博客指出这事，标题大意是"SAP 想当 AI 的数据管家，但钥匙锁在自己口袋里"。

这就形成了一个现在两家最直接的对照：Salesforce 这边大开 MCP 大门，反过来又怕客户把"开门"理解成"把房子拆了"；SAP 这边把外部 Agent 拦在门外，但内部把 Joule Studio、Knowledge Graph、Joule Capability 焊得越来越紧。一个用开放姿态找回来路，一个用封闭姿态稳住根基。两边都在赌，赌的还是同一个东西——AI Agent 时代谁来当那个"运行时入口"。

站在做出海生意的中国企业角度，这件事意义其实挺具体的。如果你的 CX 平台选 Salesforce，未来你的销售用 Slack、客服用 ChatGPT、Marketing 用一个第三方 Agent 编排器都能自己接进来——但这意味着你的合规、安全、审计责任要自己扛。你买的不再是"一个统一应用"，而是一组能被任意调用的能力包。Stokes 这次澄清想强调的是 Salesforce 的应用还在，可后半句 Milano 已经说穿了：Anthropic 现在已经不打开 Salesforce App 了。

SAP CX 走的逻辑刚好相反。Sapphire 上发那 50 多个 CX AI Agent，全都是被 Joule 编排、按业务流程串起来的；OMF、CLM、Sales Cloud V2、Service Cloud V2 这些组件之间通过事件总线连接，外面想接进来要按 SAP 的规矩走 MCP 注册、加治理、加安全栈。这套打法的代价是灵活度低一些，回报是企业级合规和审计可追溯。这一点对那些要在欧盟 AI Act、对数据出境有要求的中国出海客户来说，分量不轻。

## 客户真正要回答的不是"开不开门"

这场所谓 Headless 风波让我想到一个其实更前置的问题：客户买 CRM 平台，到底买的是"应用"还是"数据 + 能力"？如果是前者，Salesforce 这次澄清没有错，Headless 360 只是多开几扇窗；如果是后者，那 Anthropic 的 5 倍用量增长其实是一个隐喻——应用本身正在变成"看不见的中间件"，谁先把自己变成 MCP 端点，谁就活下来。

SAP CX 现在押的不是这个二选一，是另一条第三条路：应用必须存在，因为应用承载行业流程；但应用要能被 Joule 当成 Agent 来调用。Sales Cloud V2 的 Application UI 嵌入、Service Cloud V2 的 Agent Inbox、Engagement Cloud 把商品目录改成 API 对象，每一步都是同一个方向——既不去掉头，也不放任何外部 Agent 自由进出。这是一种比较慢、比较重的做法，但对那些不想后年再来一次"被自己 CMO 出来澄清"的企业来说，可能是更稳的一条线。

Salesforce 这次的小小补丁不会动摇大局，但它无意中给所有做企业 CRM 的厂商讲了一个真相：用一个新词去包装一个旧战略，客户是会自己脑补的。脑补的方向往往比公司原本想说的更激进。Benioff 在电话里那句"为什么大家觉得 Headless 就意味着没有应用了"——也许真正的答案是：因为客户已经准备好那一天到来了，只是 Salesforce 自己还没。

参考来源：https://diginomica.com/have-salesforce-customers-been-heading-down-wrong-path-their-idea-headless-360-cmo-patrick-stokes
