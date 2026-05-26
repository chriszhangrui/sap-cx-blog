---
title: "SAP API政策风波过去了吗？CTO亲自下场把话说清了"
date: 2026-05-26T18:00:00+08:00
draft: false
tags: ["SAP", "API", "Agentic AI", "Joule", "MCP", "Organizational Memory"]
description: "4月的API风波刚消停，CTO Herzig接受Diginomica独家专访，把三件被混在一起的事一一拆开。但比API更值得关注的，是他临时加的一段——组织记忆。"
source_url: "https://diginomica.com/sap-sapphire-2026-sap-cto-philipp-herzig-saps-api-policy-changes-and-why-organizational-memory"
---

4月23日开始，圈子里突然到处转API政策的截图。

"SAP要把外部Agent关在门外了。"
"以后访问自己的数据要交过路费？"

领英上一堆AI网红蹭着这个话题，越炒越凶。Sapphire Orlando之后，Diginomica的Jon Reed把SAP CTO Philipp Herzig堵在最后一天散场前，做了一次on-the-record的对谈。这是目前为止信息密度最高的官方解释。

这件事对中国出海企业的IT负责人意味着什么？我把这次访谈里几个关键点拎出来，再加点自己的判断。

## 三件被混在一起的事，被Herzig拆开了

原本一份政策更新被解读成了"SAP要锁API、收过路费、把Agent挡在门外"。Herzig的回答其实很冷静，把它拆成了三件事。

- **第一件，Fair Use 限速。** 这事SuccessFactors和Ariba做了几十年了，不新鲜。SAP拿全部客户工作负载的99百分位作为限额。已经签合同的客户，不会被砍。
- **第二件，Agent不是被关在门外。** Herzig的原话是"It makes no sense to block Cloud Code or Codex or Replit"。只要走A2A协议进得来，就放进来。挡的是什么？是来路不明、不亮身份、专门来嗅探数据的"坏Agent"。这种东西连客户自己都说不清在干嘛。
- **第三件，ODP-RFC其实是私有API。** 这是个被忽视很久的事实——它从设计之初就是给SAP系统之间互相通信用的，根本不是给外部调的。但客户和合作伙伴用了多年。审计日志里全是"SAP做了这个、SAP做了那个"，根本看不出谁是谁。这一次SAP要把它掰回去，不让它继续被当成事实API。

## 用户组的反应：从警惕到接受

DSAG（德语用户组）和ASUG（美国用户组）一开始都是冷脸的。SAP拉他们去Walldorf坐下来谈，把更新版FAQ吐出来。Reed这次在Orlando跟DSAG开了一次off-the-record的会，结论是DSAG接受了现在这版FAQ，但保留监督权——SAP后面要敢碰"基础数据访问收费"或者"限制客户用第三方分析自己的数据"，DSAG立刻就翻脸。

UKISUG主席Conor Riordan的判断更直接："just genuinely poor communication"。他说SAP在Walldorf的口径很清楚——SAP永远不会再因为客户用自己的数据而向客户收费。这个表态把多年前的Indirect Usage阴影摁了下去。

## 这件事对出海企业意味着什么

如果你的公司在用SAP系统支撑跨境业务，正在或准备接Agent类工具，那这次的政策变化要看三个层面。

**第一层，存量合同基本不影响。** Herzig明确讲了，已经签好的合同SAP不会去砍。但前提是你用的是公开的、规范的API。如果你过去几年偷懷用ODP-RFC做实时数据导出——这条路6月之后就要走Note 3255746里面的合规替代路径了。

**第二层，AI Agent要走"Golden Path"。** SAP给了一份AI Golden Path文档，本质上是说："你想接Agent可以，但要走我们认可的架构。"对中国出海企业来说，这个意味着——你之前找的那个号称用MCP给SAP做了集成的合作伙伴，得让他证明这套MCP架构是不是在Golden Path上。Reed访谈里就提到一个反例：某合作伙伴的MCP方案用了BTP Cloud Foundry，没走SAP的MCP Gateway，但因为认证方式合规，被Herzig认定是OK的。这件事说明不是只有SAP官方一条路，但路你得走对。

**第三层，组织记忆这事比API争议更重要。** 这是Herzig在访谈里临时加的一段，被Reed放在文章中段——SAP在Sapphire发布了"Organizational Memory"，要把员工脑子里的"部落知识"和决策痕迹捕获下来，跟Reltio收来的结构化数据合并，喂给Agent。这才是真正的护城河。Steve Jobs的一句话被Herzig引用："今天这么干，是因为昨天就这么干。"如果你只是把数据喂给Agent，Agent最多帮你做80%精度的查询。如果连"为什么这么决定"的推理链路都喂进去，Agent才能真正接班。

## MCP是真正的下一个考点

Reed在文章末尾把话说得很重——Emerging standards like MCP, moving at speed, put the most pressure on SAP's policies. MCP正在以SAP政策跟不上的速度演进。一个合作伙伴跟Reed吐槽：现在SAP的MCP Gateway会让Agent发起太多tool call，结果反而拖累了准确率。这种细颗粒度的架构问题，DSAG这种用户组很难替你打仗，你得自己懂。

Reed还引用了Mario Defilipe的一句尖锐评论：SAP给大多数客户和合作伙伴的MCP，并不是那个能访问到真正富上下文的那个MCP。换句话说，存在一个"标配MCP"和"内部MCP"的隔离。这件事如果不被澄清，是接下来半年合作伙伴生态最大的隐患。

## 出海企业现在该做什么

具体一点的建议——

- 把你目前所有从SAP系统抽数据的接口列一张表，标注每个接口走的是哪类API（公开REST、OData、ODP-RFC、CDS View）。如果还有用ODP-RFC做实时同步的，6月之前要拿出迁移路径。
- 找你正在用的SAP CX相关Agent方案商，让他给你出一份"我的方案在AI Golden Path上的位置"说明。如果他给不出来，这就是个红灯。
- 关注Organizational Memory的后续路线图。这玩意儿短期看不出价值，但一年后回头看，会决定你的Agent是80%精度还是95%精度。
- MCP相关的项目暂时别All In。Herzig自己都说政策还在跟标准赛跑，半年内一定会有架构变化。先做POC，先小步走。

## 最后

Reed这篇访谈最有价值的一段，其实不在API那里，是Herzig谈Organizational Memory的那段。结构化数据的护城河SAP已经修了一道（BDC + Reltio），现在他们要修第二道——把员工脑子里的非结构化决策痕迹也收进去。如果这第二道修起来了，那SAP在企业AI这个赛道上的位置就稳了。

如果没修起来，那200个Agent最后还是只能跑80%的活。

这事Reed说会continue to engage further with SAP，我也会持续关注。

参考来源：https://diginomica.com/sap-sapphire-2026-sap-cto-philipp-herzig-saps-api-policy-changes-and-why-organizational-memory
