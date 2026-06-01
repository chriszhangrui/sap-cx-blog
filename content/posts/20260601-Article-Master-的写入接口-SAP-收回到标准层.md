---
title: "Article Master 的写入接口，SAP 收回到标准层"
date: 2026-06-01T16:05:00+08:00
draft: false
tags: ["SAP CX", "S4Retail", "技术深度", "RAP", "Clean Core"]
description: "做零售项目的人都接过 BAPI_MATERIAL_SAVEDATA 那条老路，RAP + EML 的写法把商品主数据的接入点换了一层，背后是 Clean-Core 的硬约束。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-members/creating-material-master-data-in-sap-s-4hana-using-rap-and-eml/ba-p/14300852"
---

做过 SAP Retail 项目的人都熟悉一条路径：客户要批量导入商品主数据，方案文档掏出来，写的是 **BAPI_MATERIAL_SAVEDATA**。再不济换成 IDoc，外加几段封装。这条路在 ECC 时代用了二十年，到 S/4HANA Cloud 上还能跑，但只要项目沾上 Clean-Core 这个词，路就走不通了。

近期 SAP Community 上有一篇技术博客，把替代路径完整写了一遍：用 **RAP（RESTful ABAP Programming Model）** 加上 **EML（Entity Manipulation Language）**，挂在标准接口视图 I_Product 之上做一个 Unmanaged BO，把创建、扩展工厂、多语言描述这些动作全部放进 Behavior 里。代码量不见得少，但接入点彻底换了。

这篇文章值得拆开看的不是 ABAP 语法，而是它背后的几条架构判断——为什么必须换、换完之后什么变了、对零售场景意味着什么。

## BAPI 这条路，到底卡在哪

BAPI_MATERIAL_SAVEDATA 是 SAP 公开的标准 RFC，签名稳定、文档齐全，过去几乎是所有 Retail 集成的默认入口。但它有几个先天缺陷，在 S/4HANA Cloud 时代被放大了：

- **调用语义是过程式的**。一个 BAPI 调用做完以后，你需要自己 COMMIT WORK，自己处理错误回滚，自己做并发锁。它不是"业务对象"语义，是"远程过程"语义。
- **扩展点是 Z 字段加 BADI**。每加一个客制字段，要改 BAPI 接口、改 BADI 实现、再改外部调用方，三层耦合。这正是 Clean-Core 想消灭的那种依赖。
- **接口视图与之无关**。CDS 接口视图 I_Product 是新一代消费层，BAPI 走的还是旧的字段映射，两条路并行存在，意味着双倍的维护成本。

SAP 在 S/4HANA Cloud Public Edition 上已经把很多 BAPI 标记为"不推荐"。Public Cloud 客户连 SE37 都看不到，BAPI 根本不是选项；Private Cloud 还能用，但每次升级都得为这条老路写迁移说明。

## RAP + EML 是怎么把这件事重新组织的

原文给的方案，本质是一个四层调用链：

![Article Master 写入路径](https://community.sap.com/t5/image/serverpage/image-id/0)

> 注：四层结构为前端 / 消费层 → 自定义 RAP BO（Z 命名 · Unmanaged）→ 标准 RAP BO（I_ProductTP_2）→ 持久层（MARA / MAKT / MARC）。

关键在第二、三层。开发者自己定义的 RAP BO（Z 命名）不直接落表，它通过 EML 操作 SAP 标准的 **I_ProductTP_2**——这是 I_Product 对应的"事务版本"接口视图，由 SAP 拥有和维护。所有写入、关联子实体的级联创建（描述、工厂扩展），全部由 SAP 这一层兜底。

原文展示的核心 EML 写法：

```abap
METHOD create.
  DATA lt_create_product TYPE TABLE FOR CREATE I_ProductTP_2.
  DATA(ls_entity) = entities[ 1 ].

  lt_create_product = VALUE #( (
    %cid           = 'product1'
    Product        = ls_entity-Product
    ProductType    = ls_entity-ProductType
    BaseUnit       = ls_entity-BaseUnit
    IndustrySector = ls_entity-IndustrySector
    %control-Product        = if_abap_behv=>mk-on
    %control-ProductType    = if_abap_behv=>mk-on
    %control-BaseUnit       = if_abap_behv=>mk-on
    %control-IndustrySector = if_abap_behv=>mk-on ) ).

  MODIFY ENTITIES OF I_ProductTP_2 PRIVILEGED
    ENTITY Product CREATE FROM lt_create_product
    CREATE BY \_ProductDescription
      FIELDS ( Language ProductDescription )
      WITH VALUE #( ( %cid_ref = 'product1'
                      Product  = ls_entity-Product
                      %target  = VALUE #( (
                        %cid              = 'desc1'
                        Product           = ls_entity-Product
                        Language          = ls_entity-Language
                        ProductDescription= ls_entity-ProductDescription ) ) ) )
    MAPPED   DATA(ls_mapped)
    REPORTED DATA(ls_reported)
    FAILED   DATA(ls_failed).
ENDMETHOD.
```

几个细节值得注意。**%cid 和 %cid_ref** 是 RAP 的内部句柄，让你在一次 EML 调用里把"商品头"和"语言描述"作为关联实体一次性写入，不需要先 commit 一次再写描述。**CREATE BY \_ProductDescription** 这种语法直接走的是 CDS 视图上的 association，I_ProductTP_2 已经在它的 Behavior Definition 里把级联规则定义好了，开发者不需要自己处理顺序和外键。**PRIVILEGED** 关键字声明这次写入跳过常规权限检查——这在自定义业务对象代理标准对象时是必须的。

## 这套设计放弃了什么

它放弃的东西比想象中多。

- **批量吞吐**。BAPI 加 BAPI_TRANSACTION_COMMIT 配合 SUBMIT 模式，一次写几万条商品没问题。RAP 的事务边界更细，一次 MODIFY ENTITIES 写超过几千条，性能会急剧下降。原文这个例子是"管理界面级"的吞吐，不是"主数据迁移级"的。
- **Delete 能力**。原文里 delete 方法直接被关掉：APPEND failed + 报错"Delete functionality is not enabled"。这是有意为之——商品主数据在 S/4HANA Retail 里几乎不允许真删，只允许标记下架。RAP 让你显式地把这个业务约束写进 Behavior，比 BAPI 的"调了就删"更安全。
- **跨系统调用的简单性**。BAPI 暴露成 SOAP 或 RFC，跨系统直接调。RAP BO 必须额外发布成 OData V4 Service，再通过 Communication Arrangement 暴露给外部——多一层包装。

换来的是什么？是写入语义的所有权回到了 SAP 这一层。当 SAP 升级 I_ProductTP_2，自定义的 ZI_product_demo 不需要改；当 SAP 在 I_Product 上加新字段，自定义 BO 通过 metadata 扩展机制就能拿到，不需要改 BADI。这就是 Clean-Core 真正在卖的东西——升级路径的稳定性。

## 对零售场景，这件事意味着什么

S/4HANA Retail Cloud 在 SAP 2026 的产品组合调整里被划进 CX 范围，零售客户的集成压力从过去的"ERP 单点"变成"商品-促销-订单-门店"四向同步。商品主数据不再只是 ERP 内部对象，它要被 Commerce Cloud 拉去做 storefront 同步，被 Promotion Management for Retail 当作促销定价的输入，被 OMF 做订单寻源时的关键属性，被门店补货引擎当作分货依据。

这就让"商品主数据的写入接口"成了一个核心枢纽。如果还走 BAPI，每个下游系统要么自己适配字段映射，要么走一层 CPI iFlow 转换。如果走 RAP + OData V4，下游系统拿到的是同一份语义化的接口视图，**I_Product 既是读模型也是写模型的源头**，CPI 在大多数场景里只做协议转换，不再做语义翻译。

更重要的是，前面那条 PRIVILEGED 写入路径，让企业可以把"商品入库"这个动作做成一个有业务约束的服务。比如：从 PIM 同步过来的新品，必须经过买手审批才能写入工厂扩展；从供应商 EDI 进来的临时品，强制走特定的 Article Type。这些规则过去散落在 BADI、用户出口、外围中间件里，现在可以集中写在 RAP Behavior 的 validation 和 determination 里——同一处定义，多个入口共享。

## 什么时候用，什么时候别碰

这套写法适合的项目：S/4HANA Public Cloud 或 Private Cloud 上做客制化的零售项目；商品主数据的写入入口超过两个（人工 UI、PIM 同步、供应商上传等）；客户已经在做 Clean-Core 治理；下游需要 OData V4 接口而不是 SOAP。

不适合的场景：一次性的主数据迁移（用 Migration Cockpit 或 LTMC 更快）；ECC 上的项目（RAP 在 ECC 上有限支持，但接口视图未必齐全）；只需要内部 ABAP 调用、不暴露给外部的简单工具。

踩坑警示几个：第一，I_ProductTP_2 的字段不是 I_Product 的全集，有些字段（比如 Sales Org 相关）需要走 I_ProductSalesTP，要么扩展 BO，要么分两次写。第二，Unmanaged 模式下你自己负责锁，原文里的 `lock master` 是关键，掉了的话并发更新会丢数据。第三，late numbering 在批量场景下会拖慢响应，零售客户做"门店上新"这种秒级要求的场景，要评估是否切换成 early numbering 配合预留商品号段。

商品主数据的写入这件事，过去十几年没怎么变过，BAPI 一直在那里。这次变化的不是技术细节，是 SAP 把"写入语义的所有权"从客户代码收回到了标准接口。RAP 和 EML 是手段，Clean-Core 才是目的。

对正在做 S/4HANA Retail 项目的团队，这是一道选型题：今天投在 BAPI 上的客制代码，下一次升级时就是技术债。早一点切到 RAP，债就早一点清掉。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-members/creating-material-master-data-in-sap-s-4hana-using-rap-and-eml/ba-p/14300852
