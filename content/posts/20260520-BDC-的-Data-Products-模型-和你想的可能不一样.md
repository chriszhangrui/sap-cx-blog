---
title: "BDC 的 Data Products 模型，和你想的可能不一样"
date: 2026-05-20T14:05:00+08:00
draft: false
tags: ["SAP CX", "BDC", "技术深度", "Business Data Cloud", "Data Products"]
description: "Zero-copy、对象存储、ORD 元数据——SAP Business Data Cloud 把数据当产品操作的方式，正在改写企业数据平台的玩法。"
source_url: "https://sovanta.com/en/sap-business-data-cloud-how-data-products-are-used/"
---

做过 BW 迁移或者 Datasphere 项目的同行，应该都遇到过同一种纠结：业务侧问"这份销售数据能不能给营销云那边用"，开发侧的回答永远是"我建个视图给你"。十年下来，整个 SAP 生态里堆了无数个"看起来差不多但谁也不敢复用"的视图、抽数表、副本——直到 BDC（SAP Business Data Cloud）把"Data Product"作为一等公民端出来。

这事的本质，不是 SAP 又造了个新名词。是它打算把"数据"这个最不愿意被产品化的东西，强行按软件产品的方式来管理：版本、生命周期、契约、所有权、可发现性，一个都不少。本文从 sovanta 在 4 月底发的一篇技术博客切入，把 BDC 里 Data Product 的运行机制、三种创建路径、以及那些 PPT 上看不到的工程前置条件梳理一遍。

## 一、Data Product 不是"数据集"，是四层结构的封装

原文给了一个很值得抄下来的定义：一个 Data Product 是按"产品原则"管理的、可复用的、定义清晰的数据对象。这话听着虚，落到工程上其实是四层强绑定的结构：

- **Data**：核心业务信息本身，比如销售订单、财务凭证
- **Code**：抽取、转换、清洗、供给的逻辑
- **Infrastructure**：底层的存储和算力
- **Metadata & Governance**：描述、血缘、所有权、质量、访问控制

关键在最后一层。一个 CDS view 加上权限就能跑，但它不是 Data Product——因为它没有显式声明的 owner、没有版本、没有 lineage、没有可被第三方系统按契约消费的元数据。原文这句说得很到位：**operational tables, CDS views, or API outputs only become production-grade data products with a platform character through deliberate curation, harmonization, governance, and lifecycle management**。

翻译过来：表、视图、接口本身不是产品，必须经过策展、治理、版本管理这一道才算。这个区分对中大型客户特别重要——出海做全球化运营的企业，最怕的不是没有数据，是几十个国家的子公司各自维护着名字一样但口径完全不同的"销售数据"，最后没人敢做集团级看板。

## 二、整体架构：Zero-copy + Object Store + Foundation Services

![BDC Data Products 架构](http://mmbiz.qpic.cn/mmbiz_png/lWqJzSMIBLVh0kZgWvpoWv6bAg4UDYDvZONHiczzkUoSnVmCJYfA9Y0M7ULtBOHEn4Eic3SUHV7zCwITvzhMv2VFcUxApBefElf8IkFhBVjtA/0?from=appmsg)

四条架构原则按重要性排：

- **Zero-copy 消费**：不再产生物理副本。下游通过 API、事件或 Delta Sharing 直接读对象存储里的同一份数据。HANA Cloud、SAP Databricks、Enterprise Databricks、SAC 都按这条路径接入
- **对象存储底座**：放弃了"什么都塞进 HANA 内存"的旧路线，改用对象存储承载海量读密集场景。这是 BDC 能跟 Databricks 平起平坐谈联邦的物理基础
- **Foundation Services**：抽取、协调、增量同步这些脏活由平台统一接管，应用层不用自己写 Pipeline
- **ORD 元数据**（Open Resource Discovery）：每个 Data Product 自描述，包括字段结构、业务语义、权限、版本、依赖

> 一句话概括 zero-copy 的意义：以前一份订单数据要被复制到 BW、复制到数据湖、复制到 CDP、复制到 Marketing 工具，每复制一次就多一个口径分歧的来源；现在它只在对象存储里存一份，谁要用谁按契约读。

## 三、三种创建路径，决定了你现在能做什么

**路径 ①：BW Data Product Generator**——面向 BW/4HANA 存量客户，把 ADSO、CompositeProvider、MultiProvider 直接转 Data Product。**Q2 2026 起取消 Data Sharing Cockpit 中转**。

**路径 ②：Data Sharing Cockpit（Datasphere 内）**——当前最灵活的路径。S3/GCS/Azure Data Lake/Confluent 都支持，可挂 Data Marketplace。

**路径 ③：Data Product Studio**——MVP 时间窗 **Q2/Q3 2026**。最值得关注的是 Interface Data Products——预定义目标结构的接口型产品，把"数据契约"做成第一公民。

## 四、对外 Delta Sharing：和 Databricks 联邦的技术细节

跨平台 Delta Share 不是简单的连接配置，需要三样东西配齐才能工作：

```
ORD metadata    # Open Resource Discovery 格式的标题、描述、目录条目
CSN schema      # 把表结构翻译成 SAP 内部数据模型
Publish step    # 激活到 catalog / marketplace 可见
```

技术上的 Delta Share 只解决了"传输"，业务侧能不能用、能不能被发现、能不能被治理，靠的是 ORD + CSN + Publish 这三步。

## 五、那个最少被提的硬条件

> Prerequisite: Creating and operating Data Products requires an SAP Datasphere configuration with Object Store (minimum 128 GB memory).

要在 BDC 里玩 Data Product，得有一个启用了对象存储的 Datasphere 实例，且最小 128 GB 内存。这个数字看起来不大，但乘以多区域部署、加上后面要塞 Intelligent Application 模型——**容量规划的起步线就在这里**。

## 六、跟竞品的差异

对比 Snowflake Native Apps、Databricks 自身的 Unity Catalog + Delta Share，关键差异是 ORD 元数据和 CSN schema 这两层 SAP 语义。Databricks 给的是技术契约（schema + 权限），SAP 给的是**业务契约**——这份数据的业务对象是什么、跟 S/4 哪些表有 lineage、哪个 Process Owner 负责。

放弃了什么呢？放弃了"我自己什么都做"。BDC 把 AI 工作负载推给 SAP Databricks，把分析推给 SAC，自己只做"治理底座 + 数据产品交付"。

## 七、什么样的项目适合用，什么时候不要碰

**适合上**：

- 出海企业有 BW + Datasphere + 一堆非 SAP 数据源，想把"集团统一口径"真正落下来
- 跨国制造或跨境电商需要把 S/4 的订单、库存数据 zero-copy 给到 SAP Databricks 跑预测分析
- 在华外资总部 IT 团队需要把中国子公司数据在合规边界内跟全球分析平台拉通
- 营销云这条线（Engagement Cloud / Customer Data Platform）需要 S/4 的成单、退货、客户主数据做激活信号源

**不要碰**：

- 没有 BW 或 Datasphere 存量、纯轻量分析场景——大炮打蚊子
- 数据治理组织没建好、Process Owner 不清晰
- Data Product Studio 还没 GA 的窗口期，关键路径上需要"接口型 Data Product"的项目要谨慎

## 八、踩坑警示

- **对象存储起步线 128 GB 内存**：环境申请阶段就要算清楚
- **ORD + CSN + Publish 三件套**：跨平台 Delta Share 不是配个连接就完事
- **BW Data Product Generator 在 Q2 2026 之前还要走 Data Sharing Cockpit 中转**
- **"Data Product"和"被产品化的数据"不是一回事**：缺四层结构里任何一层都跑不起 zero-copy

## 结尾

BDC 这套 Data Product 模型，本质上是把数据网格（Data Mesh）那套理念用 SAP 的方式做了一遍：去中心化拥有权 + 平台化治理底座 + 强制元数据契约。它没有完全照搬 Zhamak Dehghani 那套，去掉了一些过于激进的部分（比如完全联邦的架构），保留了最核心的一条——**数据是产品，不是数据库表的别名**。

出海做全球化的企业，第一个月做的事都很像——先把全球的销售数据"拉通"。拉通这件事如果只在技术层面做，三年后必然走到老路上：又一堆视图、又一堆抽数副本、又一次推倒重来。把它当 Data Product 来运营、配上 Owner 和 Lifecycle，才有可能逃出这个循环。

当然，这一切的前提是组织侧愿意承担"业务侧对数据负责"的成本。技术架构再先进也救不了一个把数据当 IT 产物的组织。

参考来源：https://sovanta.com/en/sap-business-data-cloud-how-data-products-are-used/
