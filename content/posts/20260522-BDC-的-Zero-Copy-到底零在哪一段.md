---
title: "BDC 的 Zero-Copy 到底零在哪一段"
date: 2026-05-22T10:00:00+08:00
draft: false
tags: ["SAP CX", "BDC", "技术深度"]
description: "Delta Sharing 的零拷贝承诺只覆盖消费端，生产端仍要落 Parquet。CMDP 的两条路径：1 份还是 2 份物理副本，看你的合并逻辑写在哪。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/customer-managed-data-products-in-sap-bdc-what-really-happens-when-your/ba-p/14394675"
---

做 SAP BDC 的项目，营销话术里最容易让人误会的就是那句"zero copy"。一线工程师真把数据接进来才发现：从 S/4HANA 到 Datasphere，再到 HDLF Delta Lake，每一步都有物理动作，到底哪里"零"了？

有个真实场景：一家用 S/4HANA Private Cloud Edition 的中型企业，财务团队花了几周在 Datasphere 里建好了一个总账科目主数据 VIEW——SKA1 chart-of-accounts 表和 SKAT 文本表 join 上，过滤英文，投影出报表需要的列，业务清清爽爽。订阅 BDC 之后，自然想把这个 VIEW 直接发布成 Customer-Managed Data Product（客户管理的数据产品，下文简称 CMDP），共享给企业自己的 Databricks 工作区。

结果文档里写得清楚：HANA 关系型 VIEW 不能直接通过 Delta Sharing 共享。VIEW 必须先物化成 HANA local table，再通过 Transformation Flow 导出成 HDLF（HANA Data Lake Files）里的 Delta Lake 文件，CMDP 才能基于这个文件发布。两份物理副本——一份在 HANA 里，一份在对象存储里。

这是 SAP 在偷换概念吗？不是。但要把这件事讲清楚，需要比 marketing 材料更精确的语言。

## Delta Sharing 的零拷贝承诺，到底承诺了什么

Delta Sharing 是一个开放协议（Databricks 主导，已经厂商中立），用来把 Delta Lake 格式的数据从 provider 受控地共享给 consumer，consumer 不需要导入或复制任何字节。Delta Lake 格式的本质就是两样东西：Parquet 列式数据文件 + 一份 transaction log（一组 JSON 文件，记录每次变更），统一存放在云对象存储里——AWS S3、Azure ADLS Gen2 或 GCS，看你跑在哪个 hyperscaler 上。

关键来了：**Delta Sharing 的零拷贝保证，只覆盖 consumer boundary（消费端边界）**。Databricks 读一张 Delta-shared 表的协议级流程是这样的：

- Databricks 用 mTLS（双向 TLS，两边互相亮证书）+ OIDC（OpenID Connect 做基于令牌的授权）认证到 SAP BDC 的 Delta Sharing server
- Server 验证授权策略后，签发一组带时效的 pre-signed URL，直接指向 HDLF 对象存储里那批 Parquet 文件
- Databricks 凭 URL 通过 HTTPS 直接读 Parquet——SAP 的中间件不在这条读路径上
- URL 过期，下次查询重新签
- Databricks 端不留任何持久化副本

这个零拷贝是真的，不是包装。但 Delta Sharing 协议从来没承诺过——也不可能承诺——消除 producer side（生产端）的准备步骤。它只能共享"已经在对象存储里、已经是 Delta Lake 格式"的数据。如果你的数据躺在关系表里、视图里、CSV 里，就必须先转换成 Parquet + Delta Log。这不是数据复制，这是协议要求的格式转换。AWS、Azure、GCP、Databricks 原生、Snowflake 全是这个规矩。SAP 不是例外，SAP 是标准实现。

## BDC 里两个不同的 HDLF，先别搞混

BDC 里有两个独立的 HDLF 对象存储实例，搞混会导致整套架构理解全错位。

**Foundation Services HDLF（FOS）** 由 SAP 完全托管，存放 SAP-Managed Data Products——S/4HANA 财务、采购、销售、HR 那些开箱即用的数据产品。在 BDC Cockpit 里激活一个 Intelligent Application，SAP 会无声地完成抽取、复制、转换、发布。客户不能直接访问这一层。

**Datasphere HDLF Object Store** 则是客户自管，跑在客户自己的 Datasphere tenant 里。CMDP 全部住在这一层，配置、装载、转换、治理都是客户的事。底层对象存储技术和 FOS 一样，差别在所有权和职责。

> 一个硬约束：**CMDP 只能从 Datasphere HDLF 对象存储发布，不能从 HANA 关系空间直接发布。** 这条约束决定了下面两种架构的形态。

## 两条路径：合并逻辑放在哪，决定了几份副本

用上面那个总账科目主数据的例子（SKA1 ⋈ SKAT，过滤 EN，投影分析列），把场景拆成两种路径：

![Architecture](https://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLXMX1W25eARdd3XKRHalXmKOJWeqBiarvJoFlq4mbThQZWczv8NK4hicxBGzAGSUCniaf7FDg52BLBhBaCfQ1yBnpEZ5mkLcbqa98/0?from=appmsg)

## Option A：上游合并，1 份物理副本

核心思路：把 join 逻辑推到 S/4HANA 层，写成 CDS view。数据离开 S/4HANA 时已经是合并好的、business-ready 的扁平结果，直接落 HDLF File Space 当 Delta Lake local table。HANA 关系空间全程不参与。

PCE（Private Cloud Edition）下，ABAP 团队建一个支持抽取和 CDC 的自定义 CDS view（比如 `ZI_GL_ACCOUNT_MASTER`），用注解打开抽取能力：

```abap
@analytics.dataextraction.enabled: true
@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
define view entity ZI_GL_ACCOUNT_MASTER as
  select from ska1
  inner join skat on skat.saknr = ska1.saknr
                  and skat.ktopl = ska1.ktopl
  where skat.spras = 'E'
  { ... }
```

这个 view 会出现在 Datasphere Replication Flow 的 `CDS_EXTRACTION` container 里。Datasphere 到本地 S/4HANA 的连通走 SAP Cloud Connector——一个 outbound-only 的轻量隧道，不需要在客户防火墙开 inbound 规则。Public Cloud Edition（CE）下用 Fiori Launchpad 里的 Custom CDS View 磁贴建（`YY1_` 前缀），不用 ABAP 开发权限，连通走 ABAP SQL Service 的 communication arrangement，连 Cloud Connector 都省了。

整条数据流：

- S/4HANA 层：CDS view 完成 join、过滤、投影
- Replication Flow（Datasphere）：从 CDS_EXTRACTION 读出，目标是 HDLF File Space，数据先进 inbound buffer
- Merge Task：定时把 inbound buffer 合并进 HDLF local table——产出 Parquet 和 Delta transaction log，**这是 SAP 端唯一一份物理副本**
- Transformation Flow（可选）：基于 Spark 跑 silver layer 逻辑（过滤无效账户、币种补全、语言回退），写到 Silver HDLF local table
- CMDP 发布：建 Data Provider Profile（带 Formations 可见性），CMDP 指向 HDLF local table 作为 Product Artifact，ORD（Open Resource Discovery）描述符发布到 BDC catalog
- Delta Sharing → 企业自有 Databricks：通过 BDC Connect（2025 年 10 月已 GA），mTLS + OIDC 鉴权，签发 URL，Databricks 直读 HDLF

什么时候选 A：从零开始的新 CMDP 项目，ABAP 团队能配合改 view；高频高吞吐数据集，省掉 HANA 内存占用对成本有意义；任何想把生产端做到最瘦的场景。

## Option B：在 Datasphere VIEW 里合并，2 份物理副本

核心思路：join 逻辑写在 Datasphere 的 HANA 关系空间里，做成 VIEW。VIEW 必须先物化成 HANA local table，Transformation Flow 才能读，再产出 HDLF 的 Delta Lake 文件。SAP 端两份物理副本：HANA 关系层一份 + HDLF 对象存储一份。

大多数已经在 Datasphere 上跑了一段时间的客户，业务逻辑已经沉淀在 HANA VIEW 里，这是默认路径。两份副本的技术原因其实很直白：

- Datasphere 的 HANA VIEW 是虚拟对象，本质是一段保存的 SQL 查询定义，不含数据
- Transformation Flow 需要物理源才能读——VIEW 必须先物化
- Replication Flow 设计上只接外部源（S/4HANA、其他 ABAP 源、云数据库），不能把 Datasphere 自己的 VIEW 或 local table 当源

这个限制不是 SAP 独有。Oracle、SQL Server、Snowflake 里 view 都是虚拟对象，导出前要物化是关系数据库的通用模式。

什么时候选 B：合并逻辑已经写在 Datasphere VIEW 里，且业务上没必要重写到上游；低到中等体量的参考主数据（总账科目、成本中心、利润中心），物化表的 HANA 内存开销可以忽略；VIEW 里有复杂逻辑（多表 join、CASE 表达式、聚合），重写成 CDS view 或 Spark Transformation Flow 工作量太大不划算。

## BDC Connect 这条管道，到底怎么搭

企业自有 Databricks（SAP 叫 brownfield 场景，区别于 BDC 自带的 SAP Databricks 捆绑版）接进来，三步握手：

1. Databricks 工作区管理员在 Data Ingestion → SAP Business Data Cloud 磁贴里生成连接标识，发给 SAP BDC 管理员
2. BDC 管理员用这个标识在 BDC 里建一个 Third Party Connection，生成邀请链接
3. Databricks 管理员用邀请链接完成连接——确认消息出现"Connection to SAP Business Data Cloud was successful"，BDC 就以授权 Delta Sharing provider 身份注册到 Databricks Unity Catalog

每次 Databricks 查共享表的实际读路径：mTLS + OIDC 认证 → server 校验授权策略（哪个数据产品、哪个接收方、哪些列或分区放行）→ 签发指向 Parquet 的时效 URL → Databricks 走 HTTPS 直读 HDLF（S3/ADLS Gen2/GCS），传输全程加密。**SAP 的中间件不在这条读路径上。** 这就是消费端零拷贝的运行时形态。

这里有个值得记下来的判断：消费端的体验完全独立于生产端是 Option A 还是 Option B。Databricks 看到的就是一份共享表，背后是 1 份还是 2 份副本，对它而言无差别。

## 反向流：Databricks 算出来的结果回写 BDC，会覆盖原 CMDP 吗

这个问题几乎每个客户架构评审都会问。答案明确：**永远是新建一个 Derived Data Product（派生数据产品），原 CMDP 不会被修改、覆盖或版本化。**

数据科学家在 Databricks notebook 里用 BDC Connect SDK 三个调用，把 ML 模型产出的富集数据集发回去：

```python
bdc_connect_client.create_or_update_share()      # 注册 share + ORD 元数据
bdc_connect_client.create_or_update_share_csn()  # 用 CSN（Core Schema Notation）声明数据 schema
bdc_connect_client.publish_data_product()        # 把数据产品发到 BDC catalog
```

新派生产品在 BDC Cockpit Catalog 和 Marketplace 里以独立条目出现，SAP Analytics Cloud 报表、Datasphere 模型、其他数据产品都能消费它。原始的 GL Account master CMDP 原封不动，照样服务其他下游。原产品和派生产品在 catalog 里平级共存，各自的血缘、治理策略、更新节奏都独立。这种 composability 是 SAP 数据产品策略的根基，也是预览中的 Data Product Studio（2026 年逐步成熟）要在低代码层面规模化的方向。

## 这事对项目落地意味着什么

跨国企业 IT 在评估 BDC 时，最容易踩的坑是把 zero-copy 当成"哪一段都不复制"的承诺去和老板对齐。一旦实施进去发现 HDLF 里有一份 Parquet、HANA 里还可能多一份物化表，预期就崩了。

更准确的说法是这样的：**BDC 在消费端通过 Delta Sharing 协议提供真正的零拷贝；生产端落 Delta Lake 是协议要求的格式准备，不是数据冗余。** 选 A 还是选 B 不是对错之争，是设计权衡——合并逻辑写在哪、S/4HANA 是 PCE 还是 CE、Datasphere 已有投资有多深，这些决定走哪条路。

给跨境运营和有海外业务的中国企业几个实用判断：

- 项目还没启动、ABAP 团队配合度高，无脑选 A——后期运维一份副本比两份省事
- 已经在 Datasphere 里沉淀了大量 HANA VIEW 业务逻辑，老老实实走 B，别为了"显得干净"重写一遍，不划算
- 在和 Databricks 团队对架构时，把消费端零拷贝和生产端格式转换分开讲清楚——这是技术评审能不能通过的关键
- 反向流一定按"派生数据产品"做，别搞回写覆盖那一套——治理边界一旦糊掉，后面 lineage 就没法看
- 如果你的合规框架对数据落地物理位置敏感（比如出海企业要满足 GDPR 或本地化要求），HDLF 跑在哪个 hyperscaler 区域、对象存储具体在哪个 region，这个决策要在签合同前定下来，不是上线后再调

BDC 的故事还在演化。Data Product Studio 走出 preview、SAP-Managed Data Products 覆盖到更多 line of business、和 SAP Databricks 捆绑版的能力分工进一步收敛——2026 年这条产品线还会有大动作。但今天就要做技术决策的项目，先把 zero-copy 这件事讲清楚，比什么都重要。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/customer-managed-data-products-in-sap-bdc-what-really-happens-when-your/ba-p/14394675
