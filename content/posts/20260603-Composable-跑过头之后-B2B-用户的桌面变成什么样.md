---
title: "Composable 跑过头之后，B2B 用户的桌面变成什么样"
date: 2026-06-03T11:15:00+08:00
draft: false
tags: ["SAP CX", "SAP Commerce Cloud", "Composable Commerce", "MCP", "B2B", "Joule"]
description: "B2B 买家完成一笔订单要切几个 App？SSO 给了万能钥匙，但建筑物本身没变。Composable 的下一步，是让复杂消失在用户面前。"
source_url: "https://www.knacksystems.com/blog/the-best-composable-commerce-experience-might-be-the-one-customers-never-notice"
---

先说一个画面。一个工业制造客户的采购员，要给自家工厂下一笔润滑油订单。他打开手机里 SAP Commerce Cloud 的 B2B 商城，找到 SKU；然后切到一个独立的合同价格门户核对去年签的折扣；切到 ERP 看库存；切到一个发票系统对前一笔账期；最后回到商城下单。

五个 App，一笔订单。SSO 给了他一把万能钥匙，但建筑物没变——他还是要在五个房间之间来回跑。

这是 Composable Commerce 留下的账。一笔在技术架构上很优雅，但落到真实用户手上完全不优雅的账。

## Composable 修好了后端，前端却散了

Knack Systems 那篇文章里有一句话挺扎心：API-first 让后端更敏捷了，但客户体验经常变得更孤立。

这话翻译过来就是：架构师赢了，用户没赢。

过去这几年，从 SAP Hybris 单体架构走出来的客户，几乎清一色拥抱了 Composable 路线——PIM 用一个、Checkout 用一个、CRM 一个、ERP 一个、忠诚度积分再一个。每一块都是 best of breed，每一块都跑得飞快。

然后呢？商城前端把这五个系统的入口排成一排放在 dashboard 上。SSO 一键登录，问题"解决"了。

没有。原文用了一个我很喜欢的词，叫 **Human Middleware**——人肉中间件。系统之间没真正接好的地方，就让客户自己来填补。复制粘贴 SKU、来回切 Tab、手工核对信息——本来该集成层做的活，最终落到了买家身上。

![B2B Composable Commerce](https://www.knacksystems.com/hubfs/B2B-Composable-Commerce.jpg)

## 手机是这件事的引爆点

桌面上还能忍。六英寸的屏幕上，App 之间的跳转就是工作流的杀手。

原文举了一个很现实的场景：仓库经理要在手机上同时验规格、查订单、发起退货。如果一个统一的体验做不到，他就会回到电话和邮件——两个比 Composable Commerce 出现之前还古老的渠道。

> "App 多到一个程度，用户开始抛弃这个系统、回去打电话。"——这其实是很多 SAP Commerce 项目上线半年后的真实写照。

中国出海企业做 B2B 商城最怕的就是这个：印尼的分销商、墨西哥的批发商、越南的二级经销商，他们对 App 切换的耐心比国内 KA 客户低得多。在他们手里，"切五次"和"放弃下单"中间没几步。

## 解药藏在病灶里

这里有意思的转折来了。原文给出一个反直觉的判断：让 Composable 失效的那个 API-first 架构，恰恰是修复它的关键。

逻辑是这样的——SSO 是给用户发了一张通行证，但每个房间还得自己进。**真正要做的不是给更多通行证，是让用户根本不用知道有几个房间。**

实现这件事的方式，叫 Single Pane of Glass，单一玻璃面板。客户不需要分清楚发票来自哪个系统、产品内容来自哪个 PIM、客户档案存在哪个 CRM——他只需要看到一个统一的品牌环境。

这话听起来像是另一种 Portal。但它和过去那种把所有系统的功能塞进一个门户的做法不一样：旧的 Portal 是把所有界面摆出来给你看，新的玻璃面板是把界面收起来，只把意图和动作露出来。

## MCP 把这件事从理论变成了路径

原文里那句最值得划线的话：**MCP 是 AI 系统的 USB。**

这个比喻有它的道理。过去做集成是这样的：每两个系统之间手写一根线，ERP 到 CRM 一根、CRM 到 PIM 一根、PIM 到 Commerce 一根——n 个系统就有 n×(n-1)/2 根线。哪个系统升个级，所有连着它的线都要重新拉。

MCP 把这个图解开成了星型：每个系统对外暴露一个 MCP server，AI Agent 作为 client 去连。系统之间不再直接互相挂钩，由 Agent 在中间做调度。

放到 Commerce 场景里就是这样：一个 AI 助手发现某个出海制造客户的某款工业润滑油在 ERP 库存即将见底，去 CRM 拉这个客户的合同价、去 PIM 拉规格说明、去物流系统看到货周期，然后在对话里直接给出一个"一键续单"。

数据变成意图。意图变成动作。这是和"打开五个 App 然后人肉拼"完全不同的体验。

## 这件事跟 SAP CX 有什么关系

这就是 SAP 这一年在 Commerce Cloud、Sales Cloud V2、Service Cloud V2 上反复推 Joule 和 MCP 的底层逻辑——Composable 不会回退，但 Composable 留下的体验账，AI 是来还的。

SAP Commerce Cloud 这边，Composable Storefront（原 Spartacus）继续作为前端选择，但它越来越像一个"挂载点"——上面挂的不再只是产品列表和购物车，还要挂 Joule 的对话入口、挂 MCP 的工具调用、挂 Engagement Cloud 的实时事件。

Sales Cloud V2 那边把 GenAI 装到了报价单的入口处，让销售不用切 SAP / CRM / Excel 就能完成一份完整的 deal sheet。Service Cloud V2 把 Joule 放在客服身后做路由分发，工单不用人工找系统调字段。

每一个动作的方向都一致：**把 App 数量收回到用户视线之外**。

![Future of B2B Commerce](https://www.knacksystems.com/hs-fs/hubfs/Future-of-B2B-Commerce.jpg?width=900&height=386&name=Future-of-B2B-Commerce.jpg)

## 出海企业要怎么看这件事

中国出海品牌做 B2B 商城落地的项目里，我看到一个很常见的死法：上线时所有系统都接通了，运行半年后客户开始流失，复盘原因——不是功能不够，是分销商嫌切换太多、回到了 WhatsApp 和邮件。

这种死法，再加一个新的 App 是救不回来的。再加一个 BI 看板也救不回来。

能救的方向只有一个——把现有的 SAP Commerce Cloud + Sales Cloud + ERP 之间已经写好的那些 API 重新组织一次，让它们从"暴露给系统"变成"服务于一个 Agent"。这不是技术换代，是把 API 的消费方从"另一个系统"改成"一个 AI"。

这件事不需要 rebuild，需要的是改 API 的边界、加 MCP server 的封装、把 Joule Studio 接上去。SAP 现在把 Joule Studio 设计期访问免费开放到 2026 年底，本质上就是在等客户做这一步。

## 最后

原文有一句话我特别想引：**"客户不想要更多 Portal、更多界面、更多 App。他们想要更少的障碍。"**

Composable 是手段。统一体验是目的。如果一个 SAP CX 项目做下来，用户感受到的"系统"反而比上一代更多——那架构再先进，钱也是白花的。

最有说服力的 Composable，是用户根本不知道它是 Composable 的那种。

参考来源：https://www.knacksystems.com/blog/the-best-composable-commerce-experience-might-be-the-one-customers-never-notice
