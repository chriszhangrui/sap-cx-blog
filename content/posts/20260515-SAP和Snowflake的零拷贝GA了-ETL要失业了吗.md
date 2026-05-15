---
title: "SAP和Snowflake的零拷贝GA了，ETL要失业了吗"
date: 2026-05-15T12:00:00+08:00
draft: false
tags: ["SAP CX", "SAP Business Data Cloud", "Snowflake", "数据集成", "零拷贝"]
description: "SAP和Snowflake的零拷贝集成正式GA，双向数据共享不需要复制、不需要ETL。这对企业数据架构意味着什么？对已经投了几百万搞数据集成的客户意味着什么？"
source_url: "https://www.snowflake.com/en/blog/sap-snowflake-zero-copy-integration-enterprise-ai/"
---

**一个数字：零。**数据不搬家、不复制、不落地，SAP系统里的库存表和Snowflake里的天气预测模型之间，中间那层ETL管道直接蒸发了。5月11日，SAP和Snowflake在Sapphire大会上宣布，双方的零拷贝集成方案正式GA（General Availability）。

这不是又一个"达成战略合作"的公关稿。去年11月宣布合作的时候，大家还在猜这东西到底能不能跑起来。现在它正式进入生产环境了。

![SAP and Snowflake Integration](https://www.snowflake.com/adobe/dynamicmedia/deliver/dm-aid--e6a46fde-9d44-4743-9338-d10425d2c4a1/snowflake-sap.png?preferwebp=true&quality=85)

**先说它到底是什么。**这次GA的产品有两个：一个叫SAP Snowflake（Solution Extension），给没有Snowflake的SAP客户用，相当于SAP帮你卖了一套完整的Snowflake AI数据云；另一个叫SAP Business Data Cloud Connect for Snowflake，给已经在用Snowflake的客户，做双向零拷贝打通。

双向零拷贝——这四个字是关键。不是单向导出一份快照，而是两边都能看到对方的实时数据，数据留在原地，治理策略统一管控。用Snowflake产品负责人Christian Kleinerman的话说：**企业正在从碎片化的AI实现，走向深度植根于业务流程和数据的AI。**

打个比方：以前SAP和外部数据平台之间的关系，像两栋楼之间用卡车运文件。零拷贝等于直接打通了两栋楼之间的走廊——文件不用搬，人走过去就能看。

**这对实际业务意味着什么？**

Snowflake在博客里举了三个场景，说实话这三个场景对我们CX领域的人来说都很熟：

- **预测性供应链**：SAP里的实时库存数据 + Snowflake里的天气预测和市场信号 = 从被动补货变成主动预判。不用等缺货了再反应。

- **财务场景推演**：财务分析师直接在混合了销售、营销、运营数据的环境里跑what-if分析，不需要先等数据工程师花两周搬数据。

- **客户360视图**：SAP里的订单和购买历史 + 外部的客户情绪分析 = 预测流失、个性化推荐、触发自动化动作。这个对做CX的人来说太直接了。

尤其是第三个。以前要拼出一个真正的Customer 360，你得把SAP ERP的订单数据导到数据湖，把CRM的交互记录导过去，再把社交媒体的情绪数据也导过去，然后在上面做模型。光是数据搬运这一步就要几个月。**现在这一步被跳过了。**

**但有个前提你得想清楚。**

目前这个集成只在AWS商业区域可用。Azure和GCP的支持计划在2026年下半年。所以如果你的Snowflake跑在Azure上——还得等。

另一个现实是：零拷贝不等于零成本。数据不搬了，但数据建模的工作还在——你得确保SAP那边的语义模型（semantic model）是清晰的，Snowflake那边才能正确理解字段含义。**管道没了，翻译官还得留。**

**从更大的棋局来看。**

SAP最近一个月收了Dremio（数据湖仓），收了Reltio（主数据管理），和AWS做了零拷贝，现在Snowflake的零拷贝也GA了。这些动作拼在一起看，SAP Business Data Cloud（BDC）的战略意图非常清晰：**SAP要成为企业数据的重力中心，不管你的数据湖是Snowflake、Databricks还是AWS，你的业务数据都以SAP为锚点。**

这就像苹果的iCloud策略——你的照片可以在任何设备上看，但同步的中枢是Apple的服务器。SAP想做的是同一件事，只不过同步的是企业的订单、库存、财务和客户数据。

**对我们做CX售前的人来说，一个具体的行动建议：**

下次客户问"我的客户数据分散在SAP、Snowflake和各种SaaS里，怎么办"的时候，答案不再是"我们来做一个数据集成项目"。答案变成了"你不需要搬数据，你需要的是一层零拷贝的语义连接层"。这是一个完全不同的conversation——预算小得多，见效快得多，但对数据治理的要求高得多。

ETL不会真的失业。但它的角色正在从"每天搬一次数据"变成"只在必须物理落地的场景下才搬"。零拷贝把数据集成从一个工程问题，变成了一个治理问题。

这个变化，比我们想象的来得快。

参考来源：https://www.snowflake.com/en/blog/sap-snowflake-zero-copy-integration-enterprise-ai/
