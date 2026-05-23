---
title: "CAP Operator 换掉了 BTP 多租户 SaaS 的皮"
date: 2026-05-23T08:00:00+08:00
draft: false
tags: ["SAP CX", "BTP", "技术深度", "CAP", "Kyma"]
description: "在 Kyma 上声明式管理租户，按需起 MTXS Pod 跑迁移，闲时不占资源——SAP 把 CAP 多租户的运行模型从 Cloud Foundry 切到了 Kubernetes 原生。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/part-2-runtime-architecture-amp-cost-efficiency-gains/ba-p/14317915"
---

做过 BTP 上 SaaS 多租户应用的人，多半有过这种体验：每个 tenant 上线都要手动跟一遍订阅、订阅完还要盯 cds deploy 别失败、数据库迁移失败要人工补救、闲时那个 MTXS 进程一直挂着烧内存。一两个客户能扛过去，几十个就扛不住了。

SAP 这一两个季度悄悄推了一个东西，叫 CAP Operator——在 Kyma 上跑，本质是把 CAP（Cloud Application Programming Model）多租户的整套生命周期管理从命令式改成了 Kubernetes 原生的声明式。这不是换个部署工具的事，背后是 SAP 对"BTP 上怎么经济地跑大规模 SaaS"这件事的重新答题。

## 先看现状：Cloud Foundry 的多租户模型卡在哪

BTP 上传统跑 CAP SaaS 的路径是 Cloud Foundry。CF 是命令式的——cf push、cf deploy、订阅来了由 Service Manager 触发 HDI 容器创建，应用里嵌一个常驻的 MTXS（Multitenancy Extension Service）进程，负责 onboarding、unsubscribe、数据库 schema 升级。

这套路在小规模下能跑，但有几个结构性问题：

- MTXS 是常驻进程。哪怕一周没人订阅，它的内存和 CPU 也得一直占着。十几个应用、几十个租户摊下来，相当于一直在烧空转资源。
- 状态没有"自愈"。cds deploy 跑一半挂了、租户记录卡在中间状态，运维得手动登进去捞日志、补操作。SAP 内部把这类活叫 Day-2 Operations，传统 CF 模型基本靠人。
- CF 的 application instance 内存是"硬隔离"的。买了 1G 内存就是 1G，没法 over-commit。租户多了，账单线性涨。
- 日志没法预过滤。每个 app 把日志全量打到 SAP Cloud Logging，按 ingest 计费，规模一上来日志成本会反超计算成本。

这些问题单看都能绕，但 SaaS 厂商租户数量上 50 之后就绕不动了。

## CAP Operator 的核心改动：从命令式到声明式

Operator 模式是 Kubernetes 生态的成熟玩法——定义一组 CRD（Custom Resource Definition），写一个控制器进程持续监听这些资源的状态，让实际状态向期望状态收敛。CAP Operator 就是把这套东西落到 CAP SaaS 多租户的语境里。

SAP 定义了三个核心 CR：

```yaml
apiVersion: sme.sap.com/v1alpha1
kind: CAPApplication       # 应用元数据：BTP 服务实例绑定、域名
kind: CAPApplicationVersion # 一次发布的镜像版本与部署清单
kind: CAPTenant             # 一个 tenant 的生命周期对象
```

对开发者来说，"订阅一个新租户"这个动作不再是"去 BTP cockpit 点订阅 + 等 webhook + 触发 onboarding 脚本"——而是创建一个 CAPTenant 资源，剩下的交给 Operator。Operator 自己去调用 Service Manager 起 HDI 容器、自己去拉 MTXS Job、自己去监控 cds deploy 是否成功、失败了自己重试或上报。

这一层抽象的价值，原文一句话讲得很到位：

> The Operator does not just trigger the Service Manager; it monitors the entire lifecycle of the tenant database schema. If a cds deploy fails, the Operator detects the Unhealthy state and can automatically retry or alert.

翻译过来就是：状态机从"应用代码"挪到了"平台层"。CAP 应用本身不再背运维这个包袱。

![CAP Operator 架构](https://mmbiz.qpic.cn/sz_mmbiz_jpg/lWqJzSMIBLUJ4q8ARs5RWZLanNKNgH9ycEMyIoCEXyDXU94NMa4vAxkM0KKAvhPn72rdjIib6JchchhajyGXZVZibhiaZ8oG3UcOA9go6XPG68/0?from=appmsg)

## 真正的成本杀手：Zero-Idle MTXS

这是这次架构改动里最有意思的一块，也是直接影响账单的部分。

CF 模型里 MTXS 是常驻 sidecar 或常驻进程。CAP Operator 干的事是：把 MTXS 改成事件驱动的短生命周期 Job Pod。具体流程：

- 平时不存在。集群里没有任何 MTXS Pod 在跑，零内存零 CPU 占用。
- 订阅事件触发时，Operator 拉起一个 Job Pod，专门跑这一次的 onboarding 或 schema 升级。
- 跑完 Pod 自动销毁，资源还回 Node 池子。
- 多个租户同时升级，可以并行起多个 Job Pod，租户级并发。

SAP 给的数字是：在大规模租户场景下，运行时内存占用能降 20%–30%。这个数字来自 MTXS 这一块的省，不是整个集群的省，但已经足够说明问题——你不再为"等订阅"这件事付钱。

Cloud Foundry 上做不到这件事的根因，是 CF 的资源单元是 application instance（一个常驻进程），没有 Job 这种"跑完即焚"的原语。Kubernetes 有 Job、有 CronJob、有 Pod 生命周期管理，CAP Operator 才能把 MTXS 拆成这种用完就走的形态。

## 资源利用率：bin-packing 与 HANA Cloud 共享池

第二个成本维度是计算密度。CF 的 instance 内存是预留的、独占的。Kubernetes 不一样，Pod 的 resource request 和 limit 是分开的，可以 over-commit——也就是说同一台 Node 上可以塞下"声明总内存超过物理内存"的 Pod，因为它们不会同时打满。

CAP 多租户场景里这个特性的收益很明显：你的 100 个租户不可能同时活跃，平均下来真正在用的是 10%–20%。CF 你得为 100 个的峰值买单，Kubernetes 只用为活跃峰值买单。

HANA Cloud 这一层 SAP 用了同样思路：与其给每个应用各起一个小 HANA 实例，不如让 CAP Operator 通过 Service Manager 把多租户的 HDI 容器塞到一个大 HANA Cloud 实例里。HANA 本身的"基础占位成本"不再被多次重复支付。

> As you grow from 10 to 100 tenants, your infrastructure costs grow at a much flatter angle because the management overhead only consumes resources when it is actually working.

从 10 个租户长到 100 个，成本不是线性的——这是 SaaS 厂商最在意的曲线。

## 容易被忽略的成本：日志

原文专门拿出一段讲日志，这个点容易被低估。Cloud Logging 是按 ingest 量计费的，租户多了之后日志体量增长极快，很多团队最后发现日志账单超过了计算账单。

CAP Operator 配套用的是 Kyma Telemetry 模块，底层是 Fluent Bit 以 DaemonSet 形式跑在每个 Node 上。关键是 LogPipeline 这个 CR——可以在日志离开集群之前，先在 Node 本地按 namespace、按属性过滤掉不需要的部分。一个示意：

```yaml
apiVersion: telemetry.kyma-project.io/v1alpha1
kind: LogPipeline
metadata:
  name: cap-saas-prod
spec:
  input:
    application:
      namespaces:
        include: ["cap-prod"]
      containers:
        exclude: ["sidecar-debug"]
  filters:
    - custom: |
        Name grep
        Match *
        Exclude $level (DEBUG|TRACE)
  output:
    http:
      dedicated: true
      uri: https://<cloud-logging-endpoint>
```

CF 的模型是每个 app instance 各自往 Cloud Logging 推，没有统一的过滤入口。Kyma 这套做法对应的是：一条集群级的管道，一次绑定，所有应用共用，volume 在出口前就已经被砍下来了。规模化场景下日志成本能压住，全靠这一步。

## 这意味着什么：BTP Runtime 的"暗中切换"

放到 BTP 大盘里看，这次改动其实是 SAP 在悄悄完成一件大事：把 BTP 的 SaaS 运行时主战场从 Cloud Foundry 迁到 Kyma。表面上 BTP 同时支持 CF 和 Kyma 两个 environment，但 SAP 自己新出的开发样板（btp-cap-multitenant-saas）、新的运营工具（CAP Operator）都先押 Kyma。

这件事对几类不同角色的影响完全不一样：

- **跨境/全球化 ISV**，要在 BTP 上发布商业 SaaS 卖给海外客户：CAP Operator 是值得直接上的方案。租户增长曲线对成本不敏感这一条，足以决定要不要做这次切换。
- **出海企业自建扩展**，已有 Cloud Foundry 上的 CAP 应用：不用急着迁。CF 在小规模下还是稳的，但新写的多租户应用建议直接 Kyma 起。
- **SAP 生态 SI 合作伙伴**：这是新一波交付能力的分水岭。会写 CAP + 会用 Operator + 懂 Kubernetes Telemetry 的团队，未来两年在 BTP 客户里溢价空间会拉开。
- **纯 ABAP 客制化思维的实施方**：可以认为这条路彻底关上了。SAP 自己都把多租户的运维抽象到 Kubernetes 控制器层了，再回头去碰 SAP 核心系统改 ABAP 这件事，已经不在 SAP 的方法论里。

## 什么场景下不要碰

客观说几条不适合的：

- 单租户应用、内部用工具、租户数量不会过 10 个：上 CAP Operator 是过度工程。CF 上一个简单 cds deploy 就够了。
- 团队完全没有 Kubernetes 经验：Operator 模式的可观测性、Reconciliation Loop、CR 状态调试，对从 CF 来的人有一段陡峭的学习曲线。低估了这条曲线，上线之后排障会很痛。
- 国内纯内贸场景：BTP Kyma 在国内没有数据中心，跨境延迟和合规都过不去——这条路天然只服务出海和外资在华场景。

另一个值得提醒的踩坑点：CAPTenant 的状态机出错时，调试入口是 kubectl describe + Operator 控制器日志，而不是应用日志。这是 CF 老用户最容易忽略的地方——出问题先去看 CR 状态和 Operator 事件流，别去翻业务 Pod 的日志。

## 小结

把这次架构变化压缩成一句话：SAP 把 CAP 多租户 SaaS 的运行模型，从"应用自己管租户"重构成了"平台帮你管租户"。承载这个变化的载体是 Kyma + CAP Operator。代价是 Kubernetes 这一层的运维复杂度上来了，收益是租户数量增长时成本曲线变平、运维从命令式动作变成声明式资源、日志和资源都可以在集群层做精细化优化。

BTP 这两年看上去没什么大动作，其实底层运行时已经悄悄换了半张皮。下次做 BTP 上的多租户 SaaS 选型，CAP Operator 这个名字值得提上桌面。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/part-2-runtime-architecture-amp-cost-efficiency-gains/ba-p/14317915
