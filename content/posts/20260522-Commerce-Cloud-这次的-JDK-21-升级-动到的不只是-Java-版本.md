---
title: "Commerce Cloud 这次的 JDK 21 升级，动到的不只是 Java 版本"
date: 2026-05-22T11:00:00+08:00
draft: false
tags: ["SAP CX", "Commerce", "技术深度", "JDK 21", "Composable Storefront"]
description: "2026 年 8 月 31 日是 build 阻断硬截止。表面是换 Java，实际从 Spring 命名空间到 OAuth 流程到前端 Node 版本，全链路都要动一遍。"
source_url: "https://www.akkodis.com/en/blog/articles/sap-commerce-cloud-jdk21-upgrade-guide"
---

Commerce Cloud 客户大概率最近都收到过 SAP 的提醒：2026 年 8 月 31 日之后，CI/CD 上跑在 Java 17 的 build 会被永久拒收。这个截止日期背后是 SAP 在 2025 年 9 月发布的 2211-jdk21.1 框架更新——表面看是把 JVM 从 17 升到 21，往下扒一层会发现，从 Spring 大版本、Jakarta 命名空间、OAuth 实现、规则引擎一路到前端 Node 和 Angular 版本，全链路都要跟着动。

把这次更新当成普通的版本号变更去排期，绝大多数实施过的客户都会被回旋镖打到。

## 先看 SAP 给出的两个时间钉

2026 年 6 月 30 日是 Java 17 的最后一个安全补丁版本，过了这天，跑在 17 上的平台进入零补丁状态——监管侧的合规审计会立刻把它标红。8 月 31 日则是更硬的红线：SAP Cloud Portal 直接停掉对 Java 17 build 的接收，新功能、热修复、紧急回滚的发布通道全部冻结。

从 SAP 自己的项目经验看，一次完整的框架更新（评估到上线）需要 8 到 14 周，定制扩展越多周期越长。把这两个时间点倒推回去，2026 年第一季度还没启动的项目，基本要靠加班、缩范围或者临时停掉部分定制功能来抢窗口。

## 这次到底动了哪几层

把变更点按层堆起来，会更直观地看到这不是一次简单的换 JVM，而是一次纵向贯穿的栈级升级。

![框架更新触达的五个层面](http://mmbiz.qpic.cn/mmbiz_png/lWqJzSMIBLXefjgtxjpbT1Zzm5tgf2spBmEoKucwpLHhyHcPKLibq16jlKsswNDKXvn0ZlVc9U2psPPYtFBCCfR1hSEVhZibhfLDohLoWvmYY/0?from=appmsg)

## Spring 6 + Jakarta：命名空间一刀切

Spring 6 是这次更新里最容易被低估的破坏性变更。Spring 5 到 6 表面是大版本跳跃，实际牵动的是整个 Java EE → Jakarta EE 的命名空间迁移。所有 `javax.*` 的 import 都要改成 `jakarta.*`——这件事影响的不是 SAP 自己的代码，而是客户写过的每一段定制扩展、每一个第三方库的兼容版本。

```java
// 旧代码
import javax.servlet.http.HttpServletRequest;
import javax.persistence.Entity;

// 新代码（强制）
import jakarta.servlet.http.HttpServletRequest;
import jakarta.persistence.Entity;
```

SAP 提供了 OpenRewrite 自动化重构脚本来处理大部分批量替换。但根据 SAP 自己的说法，OpenRewrite 只覆盖一部分场景——任何走反射、动态代理、字节码增强的代码，以及依赖了内部 API 的旧扩展，都会落到剩下需要人工处理的部分里。习惯了装个 patch 就好的团队，这次会发现 patch 后面跟着一份长长的人工清单。

## OAuth：Implicit 和密码模式直接被砍

这一刀对很多老项目伤害最大。Spring Security OAuth（早期版本里 SAP Commerce 的认证骨架）已经在 2022 年宣布生命周期结束，新版本切换到 Spring Authorization Server。配套被移除的是两个流程：

- Resource Owner Password Credentials（用户名密码直传）
- Implicit Flow（前端直接拿 token 不走后端换码）

如果当年的 Composable Storefront 集成、移动端 App、SSO 桥接里用了上面任何一种走法，2211-jdk21 之后这条链路会直接断开。SAP 推的是 Authorization Code Flow + PKCE 这条标准路径，这意味着客户端要重写换码逻辑，移动端要做嵌入浏览器或外部浏览器跳转的方案选型。

> 踩坑警示：很多 B2B 客户的内部账号系统单点登录到 Commerce 那一段，当年就是用 ROPC 偷懒接的。这次不改完，B2B 用户登录会直接 401。

## Drools v8 → v10 不是顺手升级

Drools 升级看起来是因为 Spring 6 跟旧版 Drools 不兼容，被动跟随。但 v8 到 v10 涉及 KieBase 装载方式和规则文件语法的细节差异，对于在促销规则、价格规则、配送规则里写过深度定制 DRL（Drools Rule Language）的实施项目，规则文件需要逐条回归测试。这是一个安静的雷区——它不会让 build 失败，但会让线上某条本来好好跑的促销规则在某个边界条件下静默失效。

## 前端：Composable Storefront 同步换地基

后端动了，前端也躲不过。Node.js 20 在 2026 年 1 月进入 EOL，Composable Storefront（前身 Spartacus）必须迁到 Node.js 22；Angular 21 从 2026 年 2 月起成为推荐版本。这两条加在一起意味着：如果团队里前端和后端是分开排期的，必须把前后端框架更新同步起来做，否则会出现后端升完前端跑不起来或者前端升完 OAuth 链路对不上的尴尬中间态。

更早一些被处理掉的还有 Cockpit 系列扩展（admincockpit、cmscockpit 等），SAP 在 2025 年 Q3 已经把这批从产品里清掉。Accelerator 风格的传统 storefront 排在 2028 Q3 移除——还有大约两年缓冲，但路径已经清楚。

## 代价之外，到底拿到了什么

如果只把这次升级看成一次合规作业，会错过几个真正有价值的副产品。

- **Virtual Threads（虚拟线程，JDK 21 的 Project Loom）**：对 Commerce 这种典型的短请求 + 数据库 IO 密集型负载，虚拟线程意味着同样的硬件能撑住更多并发购物会话。促销活动峰值场景，基础设施支出能拿到肉眼可见的下降。
- **解锁新一代功能**：Intelligent Merchandising 的 AI 增强、Open Payment Framework、Composable Storefront 的最新组件，只在 JDK 21 这条主干上发布。停留在 17 等于主动锁死创新通道。
- **OAuth 现代化**：从安全姿态上看，强制走 Authorization Code Flow + PKCE 是这几年所有大平台的共同选择，能减少前端持有长期凭证的攻击面。

## 什么样的项目需要立刻动手

根据扩展深度做个粗判：定制代码量越大、第三方集成越多、OAuth 自接得越深，越要尽早启动评估。可以从这几个动作开始——

- 先跑一遍 OpenRewrite 的 dry-run，输出一份自动化能搞定 vs 必须人工处理的差距报告
- 把所有走 `javax.*` 的内部库依赖列出来，逐个查 `jakarta.*` 兼容版本是否存在，没有兼容版本的要么换库要么自己 fork
- 把所有 OAuth 客户端梳一遍——尤其是 B2B SSO、移动端、合作伙伴门户——确认现在用的是哪个 grant type，要换的全部排进迭代
- 规则引擎相关的促销 / 定价 / 配送规则做一次完整回归用例的录制，留给升级后做对比
- 前后端排期合并，避免出现长时间的中间态

## 出海企业视角下的几个特别提醒

Commerce Cloud 是 SAP CX 矩阵里少数几个国内有数据中心选项的产品，但出海做跨境电商和 B2B 全球分销的客户，绝大多数实例都跑在欧美数据中心。这次升级带来的额外注意点：

- 跨时区切换窗口要早定，欧美数据中心的维护窗口跟亚太业务的促销窗口经常打架
- 如果用了第三方支付通道（出海常见的 Adyen、Stripe、PayPal），重新走一遍 OAuth 链路的回归测试，特别是 PCI 合规相关的部分
- 跨境业务里常见的 ERP 集成（S/4HANA Cloud、Public Cloud、私有云混合）走 OAuth 的部分要单独排
- 做 B2B 跨国分销的客户，分销商门户走 SSO 的链路要重点测——Authorization Code Flow 在跨域 cookie 行为上跟 ROPC 完全不同

## 可以推迟，不能不做

SAP 这次留的窗口比过去任何一次主版本切换都更紧。8 月 31 日 build 阻断之后，没有补救空间——不像 EhP 升级可以拖延几年，这是一个实打实的硬墙。从产品策略上看，SAP 显然在用这次强制升级把所有客户拉到统一的现代技术栈起跑线上，给 AI 嵌入、Open Payment、Composable Commerce Services 这些新能力清场。

Commerce Cloud 上的客户不存在要不要做这个问题，只剩什么时候做和多大的代价做两个变量。把评估前置到 2026 年第一季度结束之前，是大多数中等复杂度实施的最后一个安全窗口。

参考来源：https://www.akkodis.com/en/blog/articles/sap-commerce-cloud-jdk21-upgrade-guide
