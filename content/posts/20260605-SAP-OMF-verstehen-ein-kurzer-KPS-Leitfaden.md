```markdown
# SAP OMF 不是另一个订单中心：拆单、编排、状态聚合，但不做退货

> 技术深度解读 · OMS 系列

一个真实问题：客户的 webshop 把单子打过来，里面 6 个 SKU，3 个走自营仓 S/4HANA，2 个走 dropship 给两家不同供应商，1 个货还在路上要等海外仓。这单进 SAP 之后，前端要看一个完整的订单状态，后端要拆成 4 票发给 4 个履约系统，结束之后再把每个分单的发货、签收、退款回写聚合给消费者。这个事，谁来做？

十年前的答案是 ERP 自己做，靠 ABAP 写一堆增强；五年前是 SAP CAR 里的几个模块拼一拼；现在 SAP 给的答案叫 **Order Management Foundation**（OMF）。这个名字第一次听到很容易误解——以为又是一套订单管理系统。其实不是，它是一层。

这篇说清楚三件事：OMF 不是什么、它解决了什么、以及一个真实项目跑下来踩到的坑。源材料是 KPS（一家德国 SAP SI）一个日均 6500 单的客户上线 OMF 之后写的复盘，叠加 SAP 官方文档对 OMF 与 OMSA、OSTA 三件套的定位说明。

## 先说 OMF 不是什么

这是理解 OMF 最重要的一步，因为 SAP 自己的命名很容易误导人。看到 "Order Management" 直觉会以为它是个完整的订单管理产品，类似 Salesforce OMS 或者 Oracle Order Management。**不是**。

- 它**不**做实时库存可用性查询——这是 OMSA（Order Management for Sourcing & Availability）的活
- 它**不**处理 POS 流水——这是 OSTA（Omnichannel Sales Transfer & Audit）的活
- 它**不**提供客服中心前台界面——前端要找其他 CX 工具
- 它当前版本**不**支持订单修改，无论是手工还是 API
- 它当前版本**不**处理退货

所以如果有人拿着 OMF 来对标 Manhattan Active Omni 或者 Salesforce OMS，不是同一个东西。OMF 是订单数据中枢这一**底座层**，上面要再叠业务能力。SAP 把这层从 CAR 里抽出来重做，是因为 CAR 的几个模块（POS-DTA、OAA）已经在新功能上停止迭代，新东西全部 cloud-native 重写到 BTP 上。

## 那 OMF 解决什么

三件事，对应三个核心能力。

**Order Splitting（订单拆分）**：webshop 来一张单，OMF 根据来源、商品类型、库存位置、供应商规则把它拆成 N 张子单。每张子单对应一个履约方——可能是 S/4HANA、可能是 dropship 供应商、可能是第三方仓。原始的"购物车单"在 OMF 里保留为父单，子单各走各的。

**Order Orchestration（订单编排）**：把拆出来的子单路由到对应的履约系统，并维持整个流程的状态机。这里的关键不是"发出去"——HTTP POST 谁都会写——而是处理拆单后的依赖、超时、重试、补偿。

**Status Aggregation（状态聚合）**：每个履约系统都会回写自己的处理状态——发货、揽收、签收、缺货、改期。OMF 把这些事件按父单视角聚合，对外提供一个 API：消费者在 webshop 里查"我的订单"，OMF 能告诉你 6 个 SKU 里 3 个在哪、2 个在哪、1 个在哪。

## "API first"——这个口号怎么落到配置上

SAP 这几年讲 clean core 讲了很多，OMF 是少数把这个原则贯彻得比较彻底的产品。最直接的体现：**OMF 没有 SPRO**。所有配置——拆单规则、路由规则、状态映射、Feature Toggle——都通过 API 配置。

这个设计的代价和好处都很明显。代价是顾问没有熟悉的事务码可点，要先学 OMF 的配置 schema、写 JSON、调 API。好处是配置本身天然 IaC，可以进 Git、做 Diff、做环境间同步——这件事在传统 SAP 项目里是技术债重灾区。

举例，在 OMF 里启用一个新拆单规则的典型动作：

```http
POST /omf/v1/configurations/order-split-rules
Authorization: Bearer <BTP-token>
Content-Type: application/json

{
  "ruleId": "DROPSHIP_BY_SUPPLIER",
  "condition": {
    "field": "item.fulfillmentSource",
    "operator": "eq",
    "value": "DROPSHIP"
  },
  "splitBy": "item.supplierId",
  "featureToggle": "OMF_DROPSHIP_SPLIT_V2",
  "active": false
}
```

注意最后的 `featureToggle` 字段。SAP 在 OMF 里大量使用 feature toggle 灰度新功能——一个新规则推到生产环境上之后，先 active=false，再用 toggle 按租户、按订单类型逐步开。这个工程实践对应到客户侧，就是上线节奏不再是"大版本切换"，而是"持续滚动"。

## 异步请求-响应：OMF 的核心交互模式

OMF 大部分对外接口都是**异步**的。提交一张订单进来，OMF 不会同步返回"已拆单完成"，而是返回一个 transactionId，客户端要么轮询，要么订阅 webhook。

这个设计是必须的，因为拆单 + 路由 + 履约系统调用是一个跨多个外部系统的操作，同步等下来 timeout 几乎是必然。但这要求接入端必须按异步方式重新设计。一个常见的现场坑是：客户的 webshop 团队默认所有 API 都是同步的，把 OMF 接口当成 sync 来调，跑通 demo 没问题，一上压就到处 504。

```text
# 错误示范：同步等结果
POST /omf/v3/orders        → 202 Accepted, transactionId
GET  /omf/v3/orders/{id}   → 404（还没创建完）
GET  /omf/v3/orders/{id}   → 404（还没创建完）
... 业务方判断接口"挂了"

# 正确做法：订阅事件
POST /omf/v3/orders                      → 202, transactionId
（订阅 BTP Event Mesh / webhook）
← 收到 OrderCreated 事件，含完整订单 ID
GET  /omf/v3/orders/{id}                 → 200, payload
```

## 一个真实项目踩到的几个坑

KPS 那个 6500 单/天的项目，写得很坦诚，几个点直接照搬出来：

- **OMF 自带的集成 Content 只支持 S/4HANA**，对老 SAP ERP（ECC）没有标准内容。如果客户后端还是 ECC，整套接入要自己搭 iFlow，工作量翻倍
- **OMF 的物料/订单数据模型是精简版**，不等于 S/4 的标准主数据结构。两边映射要单独设计
- **订单行项目数量有硬上限**。B2B 大客户一张采购单几百行的场景一上来就撞墙，需要直接找 SAP 提需求扩限制
- **无法关闭物料校验**。OMF 默认每条订单行都要做物料检查，找不到物料就拒单，目前没有开关
- **BTP 认证的边角案例**。接到 Google Cloud Functions 这类外部服务时，需要 workaround
- **云中断要规划**。SAP BTP 不是不会挂，重试、幂等、消息去重要在客户侧做好

## 什么时候用 OMF，什么时候不该用

**适合 OMF 的场景：**

- 多渠道订单收集 + 多履约方混合（自营 + dropship + 3PL）
- 订单量级足够大（KPS 项目稳定跑 6500/天，单量小性价比不高）
- 后端是 S/4HANA Cloud（标准集成内容现成）
- 已经在用或计划用 OMSA、OSTA

**不适合 OMF 的场景：**

- 业务流程要求大量订单修改（改地址、改数量、改 SKU）
- 退货是核心场景之一
- 期望开箱即用的客服前台界面
- 项目周期紧张、不接受灰度滚动迭代

## 最后一段

OMF 这个产品的位置很清楚——它是 SAP 把"订单数据中枢"从 ERP 里抽出来、放到 BTP 上的那一层。它的 API first、cloud-native、灰度发布这些工程做法，跟传统 SAP 项目是两套思维。对那些已经在做全球多渠道、跨履约方协同的企业，OMF 是有价值的；对单一渠道、单一仓的客户，没必要上。

---

参考来源 ｜ KPS Consulting 实施案例《SAP OMF verstehen – ein kurzer KPS-Leitfaden》（kps.com/de/insights/blog/2023/sap-omf/），叠加 SAP 官方 OMF / OMSA / OSTA 产品定位文档。
```
