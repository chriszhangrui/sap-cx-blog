---
title: "SAP Commerce前端要下线了，留给你的时间不多了"
date: 2026-05-17T23:10:00+08:00
draft: false
tags: ["SAP CX", "SAP Commerce Cloud", "Composable Commerce", "技术架构"]
description: "JSP Accelerator和WCMS Cockpit同时退役，2027年9月大限将至。这不是一次技术升级，而是一次架构站队。你选Spartacus还是Composable？"
source_url: "https://www.contentful.com/blog/sap-commerce-storefront-sunset/"
---

想象一下：你住的小区突然通知，电梯要拆了换新的，但楼梯也同时封闭改造。你唯一的选择是——要么在截止日期前搬进一部他们指定的新电梯，要么趁这个机会干脆换一套带更好电梯的房子。

SAP Commerce Cloud的客户现在面临的就是这个处境。

JSP-based Accelerator（那个从Hybris时代就陪着大家的前端框架）在2205版本已经被标记为deprecated，将在2027年9月从Commerce Cloud 2211中被彻底移除。与此同时，WCMS Cockpit——那个让运营团队直接编辑页面内容的管理后台——已经在最新的2211版本中消失了。

两条路同时封死。留给你的时间窗口，满打满算一年多一点。

## 这不是升级，是站队

很多团队第一反应是"好，那就换成SAP推荐的Composable Storefront（就是以前的Spartacus）加SmartEdit"。逻辑上没问题——官方路线，风险最低，ERP集成不用动。

但实操中你会发现一个问题。

Spartacus是一个Angular-based的PWA，通过OCC REST API跟后端通信。SmartEdit提供的是一个所见即所得的页面编辑器。对于产品目录驱动、内容量不大的B2B电商站，这套组合够用了。

但如果你的业务里，内容本身就是竞争力呢？

> 核心判断：如果你的电商站主要靠产品目录卖货、内容只是锦上添花，那就老老实实走Spartacus路线。但如果内容是你区别于竞争对手的武器——多市场、多语言、重营销活动、需要个性化——那Accelerator日落就是你重新选择架构的最佳窗口。

## 两种客户，两条路

做了这么多年SAP CX项目，我见过的Commerce客户大致分两类：

**第一类：产品目录驱动型。** 工业配件、电子元器件、化工原料这类B2B业务。产品页就是参数表，营销活动一年做不了几次，前端改版的频率以年为单位。这种客户，Spartacus + SmartEdit就是最优解。迁移路径清晰，SAP官方支持，不需要引入额外供应商。

**第二类：内容密集型。** 多市场、多品牌、重营销的零售或消费品企业。产品页只是冰山一角，底下还有着陆页、品牌故事、本地化内容、KOL合作页面、大促活动专区。运营团队每周都在催开发上新页面，每次改个banner要走发布流程。

第二类客户如果无脑选Spartacus，大概率在18个月内就会后悔——下一次数字化需求来了，发现新架构还是撑不住。

![SAP Commerce Cloud Migration Diagram](https://www.atwix.com/wp-content/uploads/2026/05/SAP-commerce-cloud-migration-diagram.jpg)

## Composable的诱惑和代价

所谓Composable Commerce（可组合架构），说白了就是把电商平台拆成一个个独立组件：前端用一个、CMS用一个、搜索用一个、支付用一个，中间全部通过API连接。每一层都可以独立演进，互不干扰。

听起来很美。但先别急着兴奋。

Composable架构的前提是你的团队有能力管理多供应商环境。你需要有人懂API编排，有人管版本兼容，有人处理跨系统的数据一致性。很多从Hybris迁移出来的团队，技术栈本来就是外包维护的，突然跳到Composable等于自己给自己挖坑。

另一个被低估的问题：数据迁移。

B2B电商不是搬几万个SKU那么简单。客户层级结构（母公司→区域→分公司→采购员），每一层有独立的审批流、信用额度、定价矩阵。一个分销商账户可能有2500个协议价覆盖800个SKU，价格逻辑分散在Commerce Cloud、ERP和销售团队的Excel里。三个数据源，从上线那天起就没对齐过。

这就是为什么迁移项目容易爆的原因——不是技术上做不到，是低估了数据复杂度。

## 时间线和决策框架

关键时间节点：

- 2025年起：WCMS Cockpit已从Commerce Cloud 2211最新版本移除
- 2026年7月：Commerce Cloud 2205（最后支持JSP Accelerator的版本）结束主流维护
- 2027年9月：JSP Accelerator从2211中彻底移除
- 2028年：扩展支持结束

如果现在启动评估，最合理的节奏是：

- 2026年Q2-Q3：完成架构评估，确定是走SAP原生路线还是Composable路线
- 2026年Q3-Q4：确定供应商选型（如果走Composable），启动内容审计
- 2027年H1：实施迁移，UAT测试，上线

留给评估的时间，其实只剩下这个夏天了。

## 一个容易被忽略的信号

有意思的是，SAP自己在Sapphire 2026上宣布了一个东西：Commerce Cloud cloud ERP edition。这是一个标准化的、深度集成S/4HANA的电商版本，原生支持AI，计划Q3 2026 GA。

这意味着什么？SAP自己也在重构Commerce的架构。如果你本来就深度绑定SAP ERP，等这个新版本出来再做决定，也许是另一个选项。但这又多了一层不确定性。

而对于那些ERP不在SAP体系内的客户（用Infor、NetSuite、Dynamics的），Commerce Cloud的整合优势本来就不存在。这些客户的答案更明确——趁现在跳出去，选一个不需要为SAP集成付溢价的平台。

> 说到底，Accelerator日落不是一个技术事件，而是一个战略抉择窗口。技术升级明年做也来得及，但架构选择的影响是5年起步的。你选的不是下一个前端框架，而是下一个五年的数字体验天花板。

参考来源：https://www.contentful.com/blog/sap-commerce-storefront-sunset/
