---
title: "Sales Cloud V2 的扩展，绕不开 BTP"
date: 2026-05-19T14:05:00+08:00
draft: false
tags: ["SAP CX", "Sales", "技术深度", "BTP", "Cloud Foundry"]
description: "Adaptation 模式撑不住的需求，标准配置卡在哪里、Side-by-Side 三应用架构怎么落、四种扩展模式各自的边界，一篇讲清楚。"
source_url: "https://www.spadoom.com/en/blog/extend-sap-sales-cloud-v2-custom-cloud-foundry-apps-btp/"
---

SAP Sales Cloud V2（以下简称 SSCV2）开箱配齐了 Lead、Opportunity、客户层级、活动跟踪——只看 demo 完全够用。但凡真在企业里跑过项目的，都撞过同一堵墙：标准配置覆盖不了的需求，并不少见。

外勤销售要一个 AI 拜访规划器，按账户优先级排日程；食品分销商希望根据账单地址自动判定销售组织；呼叫中心想把 Twilio 的浏览器通话直接落到 SSCV2 的交互记录里。这些不是边角案例——是周二上午就会被业务摆到桌上的需求。

Adaptation 模式、autoflow、key user 设置这一套配置工具能解决基础问题，但有硬边界：你做不出一个同时拉三个系统数据的自定义 UI，写不进去复杂业务逻辑（查 S/4HANA、跑评分算法、回写结果），更装不下 WebSocket 流式通话组件，也注册不了带列表页和详情页的全新业务对象。

这不是产品缺陷，是 SaaS 的边界本身。问题不是要不要扩展，而是怎么扩展才不会把自己埋进升级地狱。SAP 在 BTP 上推 Side-by-Side 扩展模型，本质就是给"标准配置撞墙"准备的退路。

## 三应用 Cloud Foundry 架构：扩展的最小骨架

Spadoom 这家瑞士 SAP CX 合作伙伴最近公开了他们三年里在 SSCV2 扩展上沉淀下来的范式。要点很直接——所有生产级扩展都遵循同一个三应用 Cloud Foundry 架构：

- **AppRouter**（`@sap/approuter`）：唯一对外入口，负责 XSUAA 认证、CORS、路由、iframe 嵌入。SSCV2 只跟它对话。
- **Express Backend**（Node.js）：REST API、业务逻辑、OData 代理、WebSocket、外部系统集成都在这里。
- **Static UI Server**：嵌入 SSCV2 页面的 mashup 静态资源。

三个应用部署在同一个 CF Space。AppRouter 用 XSUAA 的 JWT 校验用户身份，再把请求路由到 Backend 或 UI Server。所有反向调用 SSCV2 REST API 的流量必须走 BTP Destination Service——不允许直接发 HTTP，原因是 Destination 帮你处理了 OAuth2 token 交换、证书管理和连接池。这是规矩，不是建议。

![三应用架构与四种扩展模式全景](http://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLUz7TDF8liaiaOI22L8JbXNiclYVHRNQ0RwzLtM4tu9fs8ia3Q9WNho2QQ0K05J5mDgCicsbbOkEHEGkMiaXNXRhZSjHD2u1FibO2adtQ/0?from=appmsg)

## 四种扩展模式，按复杂度递增

**1. HTML Mashup ——最简单也最快**

注册一个 URL，SSCV2 在指定页面用 iframe 加载——结束。不用注册元数据，不用做 OData 端点。但"简单"不等于"弱"，关键在于 SSCV2 通过 URL query 参数把上下文传给你：`accountId`、`contactId`、`opportunityId`，加上你自定义的任意参数。Mashup 拿到这些就能调外部接口、聚合多源数据、渲染原生 UI 做不出的视图。

Spadoom 给瑞士食品分销商做的方案是：在 SSCV2 客户页里同屏显示 S/4HANA 业务伙伴的送货地址、付款条款、未结订单。销售不用在两个系统间切窗口。另一个例子是给 OPO Oeschger 做的拜访规划器，AI 评分算法对每个销售名下 100 多个客户加权打分（逾期拜访 40%、ABC 分类 30%、拜访节奏 20%、地理聚类 10%），整个拖拽日历都嵌在 mashup 里。

> ⚠️ 一个写在 SAP Note 0003704111 里的硬约束：mashup iframe 高度大约被锁在 800px。UI 设计要预先按这个上限规划，不然滚动条会让用户体验崩盘。

**2. Custom OData Service——把自定义实体变成"原生公民"**

当你想让 SSCV2 把外部数据当作一等公民——出现在导航、能搜能筛、有标准列表页和详情页——就要走 Custom Service 这条路。注册流程六步：

```
1. POST OData V4 元数据到 SSCV2 的 repository-service
2. DIRECT 模式创建 data connector，指向你的 CF AppRouter URL
3. 在 UI Designer 里配置 work list / detail / quick view
4. 通过 IAM service 把服务分配给角色
5. 如有需要，在关联实体上创建 extension fields
6. 在 Express backend 上注册 OData endpoint
```

注册完成后，SSCV2 就把你的 CF 应用当成原生数据源调。用户在导航看到自定义实体，搜索筛选都走标准 UI。Spadoom 用这个模式做了支付网关（Wallee）和产品追踪服务集成。相比 mashup 的优势在于——你的数据进入了 SSCV2 的关系模型，可以通过 extension field 跟客户、联系人、商机产生关联。

**3. Webhook——事件驱动的反应式扩展**

SSCV2 的 autoflow 能触发 webhook，发 HTTP POST 到你的 CF 应用。这是搭建实时联动的标准姿势，不用轮询、不用人工干预。一个真实场景：Intelligentfood 项目里写了个 webhook handler，新客户在 SSCV2 创建时被触发，读账单地址，按国家代码（瑞士、奥地利、法国、荷兰）判定销售组织，然后回写到客户的 `salesArrangements` 字段。原本要管理员手动指派的活，一秒内自动完成。

Webhook 用 Basic Auth 而不是 XSUAA——它是服务到服务调用，没有用户上下文。SSCV2 对失败的 webhook 会按指数退避重试，所以你的 handler 必须幂等（idempotent）。这点没踩过坑的，迟早会踩。

> ⚠️ 另一个早期踩的坑：SSCV2 的 `$batch` API 通过 Destination Service 调时返回 HTTP 405——不支持。如果 webhook handler 要更新多条记录，老老实实用并发单调，`Promise.all()` 实测 30 路并发也没问题。

**4. API Integration——和 ERP 双向同步**

从 CF 应用反向调 SSCV2 REST API，统一走 Destination Service，两种认证模式各管一摊：

- **OAuth2SAMLBearerAssertion**：用户身份传播。需要尊重 SSCV2 角色权限的操作走这条。
- **Basic Auth（技术用户）**：服务到服务。后台作业、集成任务、webhook 这种没有用户上下文的场景。

自定义数据存在哪？走 extension fields。通过 `repository-service` API 创建，每条实体响应里都有一个 `extensions` 对象供读取。Spadoom 拿这个存拜访封锁标记、客户分组码、集成同步时间戳。

> ⚠️ Extension field 的删除是**不可逆**的。一旦删掉，所有记录上对应的值全部清空，没有回滚机会。任何变更动作都先在 QAS 租户验证。

S/4HANA 的双向集成走 SAP Integration Suite 的 iFlow——这是 SAP 官方推荐姿势，Intelligentfood 项目里 S/4 业务伙伴和 SSCV2 客户的双向同步就用这条路。CRM 和 ERP 同时跑的客户，这是绕不开的范式。

## 认证和多租户：复杂度集中地

Precisely 2026 SAP Trends Survey 的数据：46% 的组织把"技能差距"列为 BTP 落地最大障碍，43% 列"开发复杂度"。这两块的复杂度恰恰都在认证和多租户上。Spadoom 沉淀下来的实践有三条：

- **XSUAA 不可商量**。每个 CF 应用都配一个 XSUAA 实例。AppRouter 校验 JWT 转发给 backend，Express 用 `@sap/xssec` 解析用户身份和 scope。
- **自定义 IDP 实现 SSO**。扩展跑在 SSCV2 mashup iframe 里时，用户已经登录过——不配 IDP 的话 iframe 会再弹一次登录框。把 SAP IAS 配成可信 IDP，AppRouter 里设 `identityProvider: "sap.custom"`，用户无感知。
- **多租户用环境变量隔离**。每个客户独立 BTP subaccount + 独立 XSUAA 实例。租户配置走 `.env.<tenantId>` 文件，部署时通过 `manifest.yml` 变量替换注入。同一份代码，独立基础设施。

## 哪些项目值得走 BTP 扩展，哪些别碰

这套架构有它的成本曲线。从 Spadoom 的项目周期看：基础 HTML mashup 1-2 周；带 CRUD 的 Custom OData service 3-6 周；CTI、拜访规划这种多模式复合扩展，2-4 个月。这是对一支具备 BTP 经验的团队的估算，新团队再加 30%-50%。

**什么样的项目适合走 BTP 扩展**：

- 客户 IT 团队对 Node.js / 前端 / OAuth 有储备，或者你能找到本地化的可靠合作伙伴；
- 业务方明确要求保持 Clean Core——后续要跟 SAP 升级节奏，不能改 SSCV2 内核；
- 需要嵌入第三方系统（电话、IM、ERP 实时数据），且 SSCV2 标准集成覆盖不到；
- 有持续的扩展需求，能摊薄三应用基座的初始投入。

**什么样的项目不该上 BTP 扩展**：

- 只为了一两个简单字段或简单工作流——adaptation 模式 + autoflow 就能搞定，别杀鸡用牛刀；
- 客户 IT 完全没有云原生能力，又不打算长期外包——三应用架构上线后没人维护，会变成新的技术债；
- 本质上要的是 ERP 重度集成而不是 CRM 体验扩展——直接走 Integration Suite + 标准 iFlow 更划算。

SSCV2 这一代产品最值得肯定的，是它把"扩展"这件事的边界画清楚了——内核的归内核，扩展的归 BTP，再不允许改产品代码。代价是开发栈变厚（要懂 CF、XSUAA、Destination、IAS、autoflow），但换来的是可升级、可隔离、可复制。从架构治理角度看，这个 trade-off 是对的。

参考来源：https://www.spadoom.com/en/blog/extend-sap-sales-cloud-v2-custom-cloud-foundry-apps-btp/
