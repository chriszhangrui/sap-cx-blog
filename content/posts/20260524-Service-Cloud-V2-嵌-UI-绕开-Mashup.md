---
title: "Service Cloud V2 嵌 UI 绕开 Mashup"
date: 2026-05-24T15:00:00+08:00
draft: false
tags: ["SAP CX", "Service", "技术深度", "Service Cloud V2"]
description: "做过 C4C Mashup 的都知道那种嵌得进去但用着别扭的感觉。Service Cloud V2 的 Embedded Component 把扩展从页面级降到了实体级。"
source_url: "https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/embedding-custom-service-list-views-in-application-ui/ba-p/14393761"
---

做过 SAP Cloud for Customer（V1）实施的都对 Mashup 不陌生：在标准账户页里塞一个 iframe，把外部系统的页面嵌进来——能跑，但很难看。样式和原生 UI 是两套，参数靠 URL 拼，权限要自己再过一遍，搜索和分析平台也接不上。

Service Cloud V2 出来之后，"扩展"这件事的位置变了。SAP Community 上一篇技术博客（Jyoti Teotia, 2026 年 5 月 11 日）讲了一个看起来不大、但能改变项目设计思路的能力：**Custom Service 的 List View 可以作为原生组件嵌入标准或自定义 UI，并且接收来自宿主页面的过滤参数**。

听上去很普通，但如果脑子里还停留在 Mashup 时代，这一步是把扩展机制从"页面级"降到了"实体级"。

## 一个具体的场景：员工详情页里看到他名下所有工单

原文给的例子很典型：自建一个 Work Order（工单）服务，每个工单关联一个项目负责人（Project Lead，引用标准 Employee 实体）。需求是——打开员工详情，自动看到这个员工名下所有工单，不用再切到工单列表手动过滤。

在 V1 时代大概率会这么干：写一个 Web 页面，把过滤逻辑塞进去，再做成 Mashup 嵌到员工页。问题是：

- 这个 Web 页面的样式与 Fiori 体系永远对不齐，做了主题切换会更难看；
- 员工 ID 走 URL 传过去，刷新、深链、权限校验各处都得自己写一遍；
- 列表里点一行想跳到工单详情？又得在 Mashup 里手动调宿主框架的 API。

V2 的新路径是：把 Work Order 这个自建实体的列表视图，作为一个原生 List Card 直接嵌到 Employee UI 里，参数绑定走 metadata，不再走 URL。

![Mashup 与 Embedded Component 路径对比](https://community.sap.com/t5/image/serverpage/image-id/408546i32353D4261A0909E?v=v2)

## 关键不是 UI，而是 metadata.json 里的 objectReference

这个能力之所以能成立，前提是 Service Cloud V2 已经把"自建实体"做成了 metadata 驱动的 Custom Service。声明一个 Work Order 实体的时候，不是写代码，而是写一份元数据 JSON，里面定义字段、关联、可过滤、可搜索、可排序等属性。

原文给出的关键片段是 projectLead 字段的定义（精简版）：

```json
{
  "name": "projectLead",
  "dataType": "OBJECT",
  "filterable": true,
  "objectDefinition": [
    {
      "name": "id",
      "dataType": "STRING",
      "filterable": true,
      "keyType": "FOREIGN",
      "objectReference": {
        "associationType": "ASSOCIATION",
        "targetEntity": "sap.crm.md.employeeservice.entity.employee",
        "targetAttribute": "id",
        "targetService": "sap.crm.md.service.employeeService",
        "sourceEntity": "customer.ssc.workorderservice.entity.workOrder",
        "sourceAttribute": "projectLead.id",
        "keyGroup": "projectLead"
      }
    }
  ]
}
```

这一段里有三个值得停下来看的设计：

- **objectReference** 把"自建实体引用标准实体"用元数据声明出来，不是靠代码层 join。targetEntity 指向 SAP 标准 Employee 实体，targetService 指明服务路径，sourceEntity / sourceAttribute 描述关联怎么落到自建实体的字段上——这是一份契约，平台拿着这份契约去解释跨实体的关联。
- **filterable: true** 不只是"能过滤"，它同时是 UI 设计态可以选作绑定参数的"白名单"。一个字段没声明 filterable，到了 Embedded Component 的绑定弹窗里就根本看不到。
- **keyType: FOREIGN** 配合 **keyGroup: projectLead** 把多个属性打包成一个逻辑外键组（id / displayId / formattedName 共属一个 keyGroup）。这是 V2 处理"显示一个 ID 加一个可读名"这种常见诉求的标准做法，不再像 V1 那样靠扩展字段拼。

换句话说，**这次 UI 嵌入能成立，本质是因为元数据层把关联、过滤、引用都讲清楚了**。UI 只是消费者。

## 设计态：Design App 在做的其实是参数绑定

元数据准备好之后，剩下的事在 Design App（V2 的可视化设计工具）里完成。流程是这样的：

- 在 Custom Service 自己的设计应用里，往 Embedded Component 区段加一个 List Card，选好这个 List Card 准备接收哪些过滤参数（弹窗里列出的就是 filterable: true 的字段）；
- 打开想嵌入的目标 UI（这里是 Employee UI），进入 Adaptation Mode（适配模式），新建一个 Section，点 Add Embedded Component；
- 系统列出当前所有"已经声明了 Embedded Component"的 Custom Service，挑选 Work Order；
- Bind Parameter 把宿主页的 Employee.id 字段绑到 List Card 的 projectLead.id 过滤参数上；
- 保存退出，员工页打开时这块组件已经按当前 Employee.id 过滤好了。

整个过程没有写一行前端代码。这点对项目实施意味着什么？意味着**原本要排给前端开发的"标准 UI 上嵌一个外部列表"的工时，可以让顾问在设计态完成**。这是 V2 一直在推的方向：把可配置的部分从代码层往下压到元数据加设计态。

## 运行时：External Service 必须自己处理过滤参数

设计态再优雅，运行时还是要落到一次 HTTP 请求。原文写得很简洁但很关键：

> "When a filter parameter is passed from the UI to the embedded component, the external service must intercept and process this request. The external service returns the filtered dataset back."

翻译成实施视角：**Embedded Component 在前端帮你做了参数绑定，但后端服务是不是真的能按这个过滤条件返回数据，平台不替你做。**

如果 Custom Service 是 V2 的 SDK 自建实体（数据落在平台上），过滤是平台自动处理的；但如果 External Service 指向自己的中间件（比如出海企业常见的"V2 加外部订单系统"组合），**过滤参数要自己解析、自己拼到下游 OData 或 REST 的查询里、自己保证权限隔离**。

一个常见的踩坑模式：在 External Service 的代码里漏掉 $filter 的处理，结果嵌入组件里看到的是"全员工的所有工单"——这种 bug 在测试环境很难发现，因为单员工数据量小，看上去和"已经过滤好"没区别。上生产之后某个员工的列表加载到几千条才会暴露。

## 对比 Mashup：放弃了什么，得到了什么

放弃的部分：

- **承载任意外部系统**的能力。Mashup 可以把任何 HTML 页面甚至外部 SaaS 嵌进来，Embedded Component 只能消费"以 Custom Service 或 Standard Service 形式注册到平台"的东西。
- **UI 自由度**。Mashup 那个 iframe 里想画什么画什么，Embedded Component 走原生 List Card，只能用平台提供的列表样式。

得到的部分：

- **原生权限链路**。当前用户的 Authorization、Business Role 自动作用到 List Card 的查询上，不用自己处理 token 透传。
- **一致的 UX**。主题切换、深色模式、响应式断点全部跟随平台升级，不需要在每次 release 之后回归外部页面的样式。
- **可治理**。哪个 UI 嵌了哪个 Custom Service 的哪个 List View，平台自己有元数据可查；Mashup 一旦多了，运维基本只能靠 Excel 手动维护。

## 落地建议：什么样的项目应该走这条路

适合的场景：

- 已经在用 Service Cloud V2，并且有 Custom Service 自建实体（这是前置条件，没有 V2 metadata 体系就用不了）；
- 需求是"在标准 UI 的某个上下文里看到自建实体的相关数据"——员工详情看工单、客户详情看自建合同、设备详情看自建巡检记录都属于这一类；
- 对这个嵌入视图的样式和交互没有特别的差异化诉求，平台原生 List Card 够用。

不要碰这条路的情况：

- **还在 V1（C4C）上**——这个能力不向前兼容，V1 项目继续用 Mashup；
- **要嵌的内容是富交互页面**（地图选点、三维查看器、复杂报表）——这种还是 Mashup 或者直接做一个独立的 Side-by-Side 应用更合适；
- **External Service 在你掌控之外**——比如直接对接第三方 SaaS API，对方未必接受额外的 $filter 协议，硬接会留下数据安全的口子；
- **需要嵌入到目前还不支持 Embedded Component 的标准 UI**——原文最后那条 Note 很诚实地写了：不是所有标准 UI 都开了这个扩展点，遇到不支持的需要走 Feature Request。这是看似细节、但项目排期里能让你卡两个月的事，**评估阶段必须先确认目标 UI 是否在支持列表里**。

## 一个更大的判断

这个能力本身不大，但它是 Service Cloud V2 一系列动作的一个缩影：扩展从"页面级"下沉到"实体级"，配置从"代码"下沉到"元数据"，权限和数据从"接口对接"下沉到"平台原生"。Sales Cloud V2 的 Custom Service、ESM 的 Service Categories 都在朝同一个方向走。

对出海企业的 IT 团队来说，这个方向意味着**实施成本和运维成本的曲线在改变**——前者会因为元数据驱动而下降，后者会因为治理工具的成熟而下降。但代价是**项目早期对元数据建模的投入要前置**：metadata.json 这一层做不细，后面所有看似"在设计态点几下"的能力都用不起来。

正在做 V2 实施或评估的团队，建议把"我们的 Custom Service metadata 写得是否完备"作为方案评审的一个独立检查项。这一项做扎实了，Embedded Component 只是顺手就能用上的能力之一。

参考来源：https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/embedding-custom-service-list-views-in-application-ui/ba-p/14393761
