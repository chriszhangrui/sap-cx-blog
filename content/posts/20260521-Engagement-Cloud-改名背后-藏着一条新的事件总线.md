---
title: "Engagement Cloud 改名背后，藏着一条新的事件总线"
date: 2026-05-21T11:05:00+08:00
draft: false
tags: ["SAP CX", "Engagement Cloud", "Emarsys", "技术深度", "Event Mesh"]
description: "Emarsys 改名 Engagement Cloud 并切出 Enterprise Edition——变的是它跟 S/4HANA、Sales/Service V2 的事件链路。"
source_url: "https://www.spadoom.com/en/blog/sap-emarsys-now-sap-engagement-cloud/"
---

"把名字里的 Marketing 拿掉。"——这是 SAP 在 2026 年 2 月 19 日发布会上做的第一件事。SAP Emarsys Customer Engagement 正式更名为 SAP Engagement Cloud，同时拆出一个新档位 Enterprise Edition。

外界第一反应是品牌动作。但凡读过 SAP CX 这两年版本节奏的人都能看出来：这次改的不是名字，是**编排引擎能消费什么事件**。Spadoom 在 4 月给出了一份偏 SI 视角的拆解，里面有几个判断值得拎出来说。

![Engagement Cloud Enterprise Edition 跨云事件编排架构](https://www.spadoom.com/en/blog/sap-emarsys-now-sap-engagement-cloud/)

## 为什么要去掉 "Marketing" 这个词

Emarsys 在 2020 年被 SAP 以约 6 亿美元收购的时候，是一个相对纯粹的全渠道营销自动化产品——擅长邮件、分群、Pre-built Tactics（预置营销战术模板）这一套。六年过去，它实际上已经在干 Loyalty 触发、Commerce 个性化、Service 后置触达、广告受众管理这些事。继续叫它 "Marketing"，就把现在能做的事情卖小了。

"Engagement" 这个词在产品定位上做了三件事：第一，从 marketing department 的工具升级成横跨整个客户生命周期的编排层；第二，命名向 Sales Cloud V2、Service Cloud V2、Commerce Cloud 看齐，进入 SAP CX 一级公民序列；第三，正面对标 Adobe Journey Optimizer 和 Salesforce Marketing Cloud Engagement——后两家都是按"旅程编排"来定位，而不是"发邮件"。

## 两个 Edition 的真正区别在哪

Spadoom 把这次拆 Edition 形容为 "edition split"——很准确。**Emarsys Edition** 就是原来那个 Emarsys，所有功能、API、集成、合同条款全部不变，老客户不需要改任何东西；**Enterprise Edition** 是建在 Emarsys 之上的新档位，加了三块能力：跨云事件编排、S/4HANA 实时信号、Joule AI。

挑几个关键的对比项：

- Omnichannel 渠道（Email/SMS/Push/In-App/Web/Ads）—— 两版都有
- 60+ Pre-built Tactics、AI 分群个性化、收入归因 —— 两版都有
- Native 集成 SAP Commerce Cloud、SAP CDP —— 两版都有
- 跨云事件编排（Cross-cloud orchestration）—— 仅 Enterprise
- S/4HANA 实时信号（基于 Event Mesh）—— 仅 Enterprise
- Joule AI for Engagement —— 仅 Enterprise
- Sales Cloud V2 / Service Cloud V2 触发器 —— 仅 Enterprise
- 扩展 SDK 与高阶 API（自定义编排）—— 仅 Enterprise

一句话：**Emarsys Edition 仍然是一朵营销云；Enterprise Edition 是要把整个 SAP CX 的数据流灌进同一个编排器。**这不是同一个产品的两个价位档，是两种架构思路。

## Event Mesh 接进来意味着什么

Enterprise Edition 真正的技术杀招是 S/4HANA 直连。原文写得很直白：

> "The S/4HANA integration uses SAP's event mesh infrastructure, which means events flow in real time without batch processing delays."

翻译过来：库存变化、订单状态、信用额度调整、发货确认——这些原本只在后台流转的 ERP 事件，现在通过 SAP 的 Event Mesh（事件总线基础设施）实时推到 Engagement Cloud，再触发对应的客户互动。

举几个具体场景：某个热销 SKU 库存告急，营销侧自动把促销重心切到替代品；订单发货后触发个性化交叉销售推荐，推荐内容基于实际包裹里有什么；客户信用额度被下调，所有高客单价品类的促销自动 suppress（抑制）。

过去要做这种联动，得在中间架一层 BTP CPI 流程，或者干脆在 Marketing Cloud 里写一堆定时拉取——延迟通常以小时计。Event Mesh 把延迟压到秒级，而且不用客户自己维护中间件。这条路径跟 Sales Cloud V2 用 BTP 做侧扩展、BDC 用 Data Products 做数据基础是同一个方向：**SAP 在系统性地把"事件总线"变成 CX 各产品之间的标准接合面。**

## SAP Marketing Cloud 客户的迁移窗口正在关闭

这次发布另一条不那么显眼但更紧迫的信息：SAP Marketing Cloud（不是 Emarsys，是那个跑在 BTP 上的老营销云）的官方支持终止日期是 **2026 年 12 月**。原文给的数字是全球还有 2000+ 公司在跑 Marketing Cloud。

SAP 钦定的接班人是 Engagement Cloud（Emarsys Edition）。但这不是 lift-and-shift（原样搬迁）。原文列了几个迁移上的关键点：

- 能迁的：客户数据与分群（通过 export/import 或 CDP 桥接）、集成端点（重新对接 Emarsys API）、历史数据（导出归档）
- 要重建的：营销计划与活动工作流（编排模型完全不同）、Marketing Cloud 上的 BTP 客制化扩展、SAP Analytics Cloud 连接（数据模型不同）、同意管理配置（Emarsys 有自己的一套同意框架）

原文给的典型迁移周期是 8–14 周。倒推一下，从今天（2026 年 5 月）到 12 月只剩 7 个月窗口，留给评估和并跑验证的时间已经不多了。

## 什么样的项目适合上 Enterprise Edition

结合实施经验和原文的判断，**Enterprise Edition 不是 Emarsys Edition 的"升级版本"，是一种不同的项目类型**。判断标准大致是这三条：

- 客户的 SAP 资产足够厚——至少要有 S/4HANA + 至少一个 V2 云（Sales 或 Service），单纯只有 Commerce + Emarsys 的组合，跑 Emarsys Edition 已经能覆盖 80–90% 用例
- 业务场景里有明确的"跨系统编排"需求——比如服务投诉发生后必须自动抑制下一波营销、订单异常必须实时改写客户旅程；这种场景在零售、制造（含售后服务的）尤其常见
- 有专门的开发力量去用 SDK——Webhook 入口、自定义编排逻辑（serverless functions）、高阶分群计算 API、事件回放和调试工具，这些不是配置员能玩转的

反过来，**什么时候不要碰 Enterprise Edition：**

- SAP 资产薄、ERP 还在 ECC 上、短期没有 S/4HANA 路线图——付了 orchestration fee 也用不上 ERP 信号
- 业务诉求是"把邮件发好"——不要被销售话术拐到 Enterprise，Emarsys Edition 在邮件这条线上的能力本来就没缩水
- 原本就跑 Marketing Cloud 但客制化极重——先做评估再决定，把 BTP 客制逻辑抄过来未必比就地重写省钱

## 几个踩坑警示

**关于定价：**原文写得很谨慎——"SAP has not published definitive public pricing for Enterprise Edition at the time of writing"。Spadoom 的预估是 per-contact base 之外加一个跟跨云事件量挂钩的 orchestration fee。有跨境电商客户事件量很大，签合同前一定要把事件计费的口径锁清楚（什么算一个事件？同一个 order 的多次状态变更算几次？）。

**关于"中国市场可用性"：**Engagement Cloud（无论哪个 Edition）目前没有中国大陆数据中心。出海企业、跨境电商、在华外资总部对接的项目可以正常采购；纯境内业务的客户拿不到这套能力。这条对所有谈中国客户的方案都成立——把它写在 Discovery 阶段的硬性边界里，能省掉后面很多沟通成本。

**关于 Joule：**Joule for Engagement 当前形态主要还是建议生成、自然语言转工作流、效果分析建议——属于"AI 副驾"，不是"AI 自动驾驶"。落地评估时把 Joule 的 ROI 单独算一遍，不要把它的预期效果叠到核心营销 KPI 里。这条跟前阵子写 Joule Studio + MCP 的判断一致：AI 在 SAP CX 里目前是放在工作流后侧，不是前侧。

## 一句总结

Engagement Cloud 这次改名，把 Emarsys 从"营销云"重新定位成"编排层"——但只有 Enterprise Edition 真正享受到了这层重新定位带来的架构红利。**真正在变的不是产品标签，是 SAP CX 各个产品之间事件交换的方式。**从 BDC 的 Data Products，到 Joule Studio 的 MCP，再到这次 Engagement Cloud 接 Event Mesh，方向是一致的：把数据、模型、事件这三层各自标准化，让客户在编排层做拼装。

如果手上还在跑 SAP Marketing Cloud——12 月的支持终止时间表已经在倒计时了，越早开始评估越好。

参考来源：https://www.spadoom.com/en/blog/sap-emarsys-now-sap-engagement-cloud/
