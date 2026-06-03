---
title: "BTP 鉴权架构换挡：从 XSUAA 到 AMS 动了什么"
date: 2026-06-03T12:00:00+08:00
draft: false
tags: ["SAP CX", "BTP", "技术深度", "CAP", "AMS"]
description: "在 BTP 上做 S/4HANA 扩展，鉴权层正在从 scope 驱动切到 policy 驱动。这件事比换个配置文件复杂得多。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/from-xsuaa-to-ams-architecting-s4hana-cap-extensions-with-policy-based/ba-p/14056323"
---

一个 SAP S/4HANA 扩展项目里，常常出现这种需求：审批人按国家、按公司代码、按业务单元被分到不同的数据范围。一个全球供应商主数据应用，德国的审批人只能看到 DE 的请求，新加坡的审批人只能看到 SG 的请求，"全球管理员"则跨所有国家可见。

在 BTP 上做这件事，传统做法是给每个国家造一个角色——VendorReviewer_DE、VendorReviewer_SG、VendorReviewer_US……当业务上线 30 个国家时，角色清单变成 30 行。再叠加上"区域审核员""全球只读"，角色矩阵迅速膨胀到三位数。每多一个国家就要回到 xs-security.json 改 scope、回到 SAP BTP Cockpit 重发 role collection、再让 IT 把人重新分到新角色里去。这不是配置问题，是架构问题。

SAP 在 BTP 上正在把这个模型换掉。新的方向叫 AMS（Authorization Management Service），逻辑是把"角色"和"数据范围"解耦——角色保持稳定，数据范围由可在运行时评估的策略（policy）决定。这篇技术博客用一个完整的 Vendor Onboarding 应用，把 XSUAA 到 AMS 的迁移路径走了一遍。我把里面的关键架构判断拎出来，谈一下这件事对在 BTP 上做扩展应用的团队意味着什么。

## 一、问题不在 XSUAA，问题在 scope-driven 这个范式

先把术语理清楚。XSUAA（Extended Services for User Account and Authentication）是 BTP 上传统的 OAuth 授权服务器，配 IAS（Identity Authentication Service）做身份认证。IAS 负责"你是谁"，XSUAA 负责"你能做什么"，这个分工本身没问题。

问题在于 XSUAA 用 scope 来表达权限。Scope 是写在 xs-security.json 里、烧进 JWT token 的静态字符串。一个用户登录后，他的 token 里带着 vendor.read、vendor.approve、vendor.approve.DE 这些 scope，应用读到 token 就知道能干什么。

但凡权限只跟"动作"绑定（能不能审批），scope 就够用。一旦要按"业务上下文"（哪个国家的审批）切片，scope 就不够了——因为业务上下文是数据维度，不是动作维度。强行用 scope 表达，就出现 vendor.approve.DE、vendor.approve.SG 这类"角色爆炸"。每加一个国家，相当于在设计期把一个新分支烙进 token 结构。

AMS 的设计前提就是承认这一点：scope 只解决"动作"，"上下文"必须放到运行时去评估。所以 AMS 不再往 JWT 里塞授权信息，而是让 CAP 应用在每次访问时调用 AMS 引擎，由引擎按 policy 当场算出能不能访问、能访问哪些数据。

## 二、AMS 的两份配置：DCL 是关键

AMS 的项目结构里有一个 ams/dcl/ 目录，CAP 命令 `cds add ams` 会把它生成出来。DCL（Data Control Language）是 SAP 自己定义的策略语言，专门描述"什么角色在什么条件下能访问什么数据"。

DCL 目录下两类文件分工很清楚：

- **schema.dcl**：自动生成，定义可以参与策略判断的属性，比如 country、company_code。这一份是从 CAP 实体注解推导出来的。
- **basePolicies.dcl**：CAP 的 AMS 插件根据 @restrict 注解自动生成基础策略骨架。
- **reviewerPolicies.dcl**：手写的应用层策略，把"VendorReviewer 在 country=DE 的实例上可以 approve"这类规则写出来。

在 CAP 实体上加一行注解，country 字段就被注册为 AMS 可识别的授权属性：

```cds
@AMS.attributes: { country: (country) }
entity VendorRequests as projection on db.VendorRequests {
  ...
}
```

`cds build` 之后 schema.dcl 里就会出现 country 作为条件属性。后面手写的策略文件直接引用这个属性写规则。一个 VendorReviewer 角色配上不同的策略包，就能在德国实例和新加坡实例上表现不同——角色还是同一个角色，行为是策略决定的。

## 三、@restrict 这一行注解，背后是整条策略生成链

原文里有个细节值得停一停。CAP 的 @restrict 注解不是新东西，老 XSUAA 项目里就有。但在 AMS 模式下，@restrict 的语义变了——它从"声明 scope 检查"变成了"触发 AMS 策略生成"。

CAP 的 AMS 插件看到 @restrict 之后会自动写出 basePolicies.dcl 的骨架，再让开发者在 reviewerPolicies.dcl 里补具体条件。这一步把"谁能调这个服务"和"他能看到哪些行"解耦了：前者还是声明式的，后者交给运行时引擎。

迁移项目里这一点经常被忽视——团队以为只是把 xs-security.json 换个写法，结果 @restrict 的判定路径整个变了。原本的 unit test 里 mock 一个 JWT 就能跑过，AMS 模式下需要 mock AMS 引擎的策略响应。测试基础设施跟着重做。

## 四、迁移成本到底在哪里

把这套架构铺到一个真实项目里，迁移成本主要在三个地方：

**第一，身份与授权服务的拆分。** 原本 XSUAA 既做 token 颁发又做权限信息的载体，现在 IAS 负责颁发身份 token、AMS 负责授权决策，两个服务都要绑定到应用上，App Router 的路由配置要改。

**第二，策略文件的生命周期管理。** DCL 文件是新的产物，要进 Git、要走 CI/CD、要做版本管理。生产环境上策略变更不再走应用部署流水线，而是单独的 AMS 策略发布通道。变更影响面变小，但需要新的发布纪律——谁有权改 DCL、改完谁审、生产环境怎么回滚。

**第三，HANA 持久化层和审计的对齐。** AMS 是判定层，数据落库是 HANA 干的事。当审计问"为什么 X 用户在 6 月 3 日看到了不该看到的记录"，需要把 AMS 当时评估的策略快照、用户的属性快照、HANA 的数据快照三件事对得上。这一层日志策略要在项目早期就定。

## 五、什么样的项目该上 AMS，什么时候不要碰

**建议上的场景：**

- 多国家、多组织、多公司代码的全球性扩展应用。AMS 是为这种场景设计的，强行用 XSUAA 一定会撞角色爆炸的墙。
- 业务规则会持续变化的应用。比如审批阈值随业务调整、数据可见性按组织架构调整。AMS 的策略热更新比 XSUAA 的重新部署快两个数量级。
- 需要做精细化数据隔离的合规场景。GDPR、跨境数据隔离、SOX 合规都属于这一类——能在策略层留下审计痕迹比在代码里写 if 安全得多。

**暂时不建议上的场景：**

- 只有几个固定角色、权限模型基本不变的小型扩展。XSUAA 已经够用，迁到 AMS 是无谓增加复杂度。
- 高并发、低延迟敏感的业务流程。AMS 是运行时调用的服务，每次访问多一跳网络。链路上 P99 延迟敏感时要做容量压测，不能想当然。
- 团队还没用过 IAS、对 BTP 安全模型不熟悉的项目。AMS 不是"装个插件就用"，它是 IAS+AMS+CAP+HANA 的协同。先打稳 IAS 基础再谈策略化授权。

## 六、给做 BTP 扩展的团队几个落地动作

- 把现有 XSUAA 项目的角色清单导出来，统计有多少角色是"基础角色 × 数据维度"组合出来的。这个比例越高，迁 AMS 收益越大。
- 在新项目脚手架阶段就决定用哪条路。从 XSUAA 切到 AMS 需要重写 @restrict 语义、重写测试 mock、重新配置 IAS+AMS 服务实例，中途切换代价不小。
- 把 DCL 文件当代码资产对待。建独立的 review 流程、独立的环境提升流水线，不要塞进应用代码的 PR 里捎带改。
- 在 PoC 阶段就把"策略快照 + 用户属性快照 + 数据快照"的审计链路连起来。后期补这个比从零做难十倍。

## 写在最后

XSUAA 不会立刻被淘汰，SAP 自己在 BTP 上的很多服务还在用它。但当你做的是"S/4HANA 全球扩展"这一类应用——多国家、多组织、规则随业务长期演进——AMS 是该认真考虑的方向。它解决的不是某个 bug，是 scope-driven 这个范式在数据维度上的天花板。

更值得注意的信号是：SAP 自己的参考架构（Joule Studio、Pro-Code Agent 在 BTP 上的部署蓝图）越来越多地以 IAS+AMS 作为默认前提。一年内 AMS 大概率会成为"BTP 上做企业级扩展"的事实标准。今天踩坑积累的迁移经验，值得提前准备。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/from-xsuaa-to-ams-architecting-s4hana-cap-extensions-with-policy-based/ba-p/14056323
