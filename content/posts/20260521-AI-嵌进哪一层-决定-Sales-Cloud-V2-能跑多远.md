---
title: "AI 嵌进哪一层，决定 Sales Cloud V2 能跑多远"
date: 2026-05-21T17:10:00+08:00
draft: false
tags: ["SAP CX", "Sales", "技术深度"]
description: "2026 Q1 给 Sales Cloud V2 砸进来一批 AI 特性。AI 没被独立做成新产品，而是嵌进业务流；移动端 OCR 让给 Apple Intelligence。这两个选择比新增功能更值得拆。"
source_url: "https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/sap-sales-cloud-q1-2026-innovation-overview/ba-p/14377174"
---

SAP 在 Q1 2026 给 Sales Cloud V2 砸进来一批 AI 特性。先不看营销词，把这批特性按"它放在系统的哪一层"摊开来看，更能讲清楚 SAP 这次想做什么。

最有意思的两点：第一，AI 没有被独立做成新产品，而是嵌进现有的销售业务流（邮件、商机、移动端）；第二，移动端 OCR 让出给 Apple Intelligence，端侧推理替代了云端模型。这两个选择，比新增了几个功能本身更值得拆。

## 从一个工程问题切入：销售流程里的 AI 该放哪里

B2B 销售场景里，邮件接单、商机推进、续约管理这几个动作，本质上都是结构化数据的"识别—推断—写入"。过去 SAP 的处理是把这些环节交给销售代表手工完成——读完邮件，登录 Sales Cloud，建商机、加产品、调价格、发报价单。

2026 Q1 这批新功能，本质上是把这条人工动线的几个关键节点，替换成嵌入式 AI 推理。问题在于：AI 应该放在用户界面层（让销售代表点一下"AI 帮我填"），还是放在业务对象层（系统自己决定何时触发模型）？SAP 这次的选择是混合——前端触发，但模型紧贴业务对象绑定，不走 Joule Studio 那种独立 Agent 路径。

![Sales Cloud V2 · Q1 2026 AI 嵌入层级](http://mmbiz.qpic.cn/mmbiz_png/lWqJzSMIBLW2DEuRxkjLqceSrYPPlujYGkoae1PQKaBaF6hOicvS79sT3vsdSJnPsButU2qs7pwX0AVbluhJmUiaR2bD2XgrjwEtyibOWghbQs/0?from=appmsg)

## Email-to-Order：从 Outlook 插件回看 V2 的扩展路径

这个特性官方描述很短：销售代表在 Outlook 里收到客户的订货邮件，插件自动识别 SKU 和数量，一键生成订单到 Sales Cloud V2，再走 ERP 落地。

值得拆解的不是"AI 能识别 SKU"——这是 NLP 老活了——而是这个功能的部署位置。Outlook 插件本身是 Office 365 应用商店里那种 Web Add-in，它必须能调用 Sales Cloud V2 的 API。这意味着：

- 调用方需要拿到租户级的 OAuth 2.0 token，走 Sales Cloud V2 的 OData v4 接口；
- SKU 识别走的是 SAP 托管的 GenAI Hub（不是租户里的 Joule，否则推理延迟扛不住邮件场景）；
- 写入路径要经过价格主数据匹配——这部分如果接 S/4HANA，就要用 Group System 配置和 SOAP/IDoc 通道兜底；
- 订单生成不是直接跨系统，而是先在 Sales Cloud V2 里创建报价/订单业务对象，再走异步集成往 S/4 同步。

这条路径的关键判断：SAP 没有把 AI 做成"独立的中间件去拦截邮件再写多个系统"，而是让 Sales Cloud V2 继续作为单一控制点。所有 AI 输出都先回流到 V2 的业务对象，再由原本的集成层往下游走。这条设计的代价是 ERP 同步还是异步——但好处是销售流程里"AI 改了什么"完全可追溯。

## Predictive Close Date：嵌入式 ML 与 Joule Agent 的分工

这个功能让系统基于历史商机数据预测真实的成交日期，而不是销售代表主观填的那个日期。乍一看是 ML，但放进 SAP 现在的 AI 体系里看，它属于"Embedded AI"——和 Joule 这种代理式 AI 是两条不同的产品线。

两者的工程区别：

```
Embedded AI（Predictive Close Date）
  - 训练数据：租户内历史 Opportunity + Activity
  - 推理触发：业务对象保存事件
  - 输出绑定：Opportunity 字段（predicted_close_date）
  - 用户感知：列表/管道视图里多一列预测值

Joule（Sales 域 Agent）
  - 训练数据：跨租户 SAP 知识 + 跨域客户数据
  - 推理触发：用户对话或手动召唤
  - 输出绑定：自然语言回复 + 可选的业务动作
  - 用户感知：Joule 入口或独立 Studio
```

Predictive Close Date 不是 Joule 在背后跑——它是 Sales Cloud V2 自带的 ML 服务，模型生命周期、特征存储都在产品内部。这种"贴着业务对象长"的 AI，对实施团队的好处是：上线只需要打开开关、做一次基线训练，不需要去 BTP 上配 GenAI Hub、不需要写 Agent。坏处是定制空间小，模型逻辑黑盒。

做 V2 项目时的一个判断点：先用 Embedded AI 拿现成效果，等业务方提出"我要让模型解释为什么这条商机被压了三个月"这种需求时，再考虑接入 Joule Studio 自建 Agent。两者顺序倒过来会很痛苦。

## Subscription Renewal：续约对象不是新表，而是新的关系视图

订阅续约商机是 V2 这次新增的对象类型。它解决的是一个老问题——客户成功（CS）和销售（Sales）双方各管各的数据，订阅到期前没人统一看视野。

从数据建模角度，续约对象的关键不是新增了什么字段，而是它关联了三个对象：原始订单、客户健康度、活动记录。这种"关联式商机"的实现方式直接决定了报表能不能跑得动——如果只是在 Opportunity 表上加个 type=renewal 的标记，报表跑大客户视图时会很慢；如果是独立物理对象 + 关联键，跨域分析才好做。

这部分往下接 BDC（Business Data Cloud）的数据产品模型很自然——续约对象本身就是个跨域的数据消费场景，BDC 提供 Sales 和 CS 数据的统一语义层正好对得上。出海企业如果在用 Subscription Billing + Sales Cloud V2 双引擎，续约对象就是这两条数据线在 V2 里的汇合点。

## 移动端把 OCR 让给 Apple Intelligence

名片扫描这个功能听上去普通，但 SAP 这次的实现方式是个工程信号——iOS 端调用的是 Apple Intelligence 的端侧 OCR，不走云端模型。

这个选择的逻辑链：名片扫描场景对延迟敏感（销售代表在展会上当场扫一张就要弹出"这是哪家公司、要不要建联系人"），云端模型从拍照到出结果至少 1-2 秒，端侧 OCR 控制在 300ms 内。Apple Intelligence 在 iPhone 15 Pro 以上机型本来就有 Vision Framework + Live Text，调用成本接近零。

SAP 把这一步交给苹果，自己只接收结构化结果（姓名、职位、公司）后写入 Lead 对象。这是个克制的架构选择——不是所有 AI 能力都得自己做。Android 端短期内不会有同等体验，因为 Google 的 ML Kit 和 Apple Intelligence 的 OCR 精度差距还在。

## Daily Catch Up：移动端的"今日要事"屏

销售代表打开 App 第一屏不是商机列表，而是当天的关键活动、待跟进、AI 摘要的下一步动作。这个交互模式参考过 Salesforce Mobile 的 Today 视图，但 SAP 这次往里塞了 AI 摘要——把跨对象（活动、邮件、会议、商机更新）的近 24 小时事件做归并，按客户维度合一。

从架构看，这是个新的查询入口，不是新数据。它需要在租户内有一个"近期事件聚合"服务，模型按客户优先级排序输出 top-N。实施时的坑：聚合查询如果直接打主表会拖性能，得有读写分离的视图层。

## 什么样的项目可以现在上 V2 这批 AI

出海企业、有海外业务的跨境制造、在华外资销售总部，这三类客户做 Sales Cloud V2 项目时，可以把 AI 特性分成两档考虑：

- **开箱即用档**：Predictive Close Date、Subscription Renewal、Daily Catch Up。这些功能 Q1 已经 GA 或在路线图上，不需要额外的 BTP 资源，开关打开就能用。
- **需要 BTP 配套档**：Email-to-Order（依赖 Outlook 插件 + Office 365 域信任）、Joule Studio 里自己写的 Sales Agent。这些要先拉一套 BTP 子账户，做好 Cloud Foundry 的扩展环境。

什么时候不要碰：纯内贸、没有海外业务的中国本土企业。Sales Cloud V2 的数据中心目前不在国内，监管和延迟都过不去。Commerce Cloud 是 SAP CX 里唯一在国内有数据中心的产品，其他几条线都只对有海外业务的企业开放。

**踩坑预警**：Predictive Close Date 的模型基线需要至少 6-12 个月的历史商机数据才有意义；冷启动租户跑出来的预测会很糟。建议项目上线前 6 个月就把销售代表的商机录入习惯抓好——AI 模型从来不能挽救脏数据。

## 写在最后

这一轮 Sales Cloud V2 的更新，把 AI 嵌入业务对象层，把端侧能力让给设备厂商，把代理式 AI 留给 Joule Studio。三条路径分得清楚的产品体系，比一锅炖的"全能 AI"更值得做项目时仔细看。

下一个真正值得跟踪的信号：当 V2 把 BDC 数据产品作为 AI 训练特征源接进来，跨域销售模型才会显出威力。在那之前，这些 Q1 特性是"系统在替销售代表干以前他自己干的活"，再往后才是"系统帮销售代表看到他自己看不到的事"。

参考来源：https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/sap-sales-cloud-q1-2026-innovation-overview/ba-p/14377174
