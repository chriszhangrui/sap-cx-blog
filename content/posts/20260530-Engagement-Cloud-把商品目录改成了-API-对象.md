---
title: "Engagement Cloud 把商品目录改成了 API 对象"
date: 2026-05-30T23:10:00+08:00
draft: false
tags: ["SAP CX", "Engagement Cloud", "技术深度"]
description: "AI-assisted Product Finder 在 Q4 走向 GA，技术上真正变化的那一层不在前端，而在商品目录的接入方式。"
source_url: "https://emarsys.com/learn/blog/2025-in-review-the-year-of-ai-powered-innovations-for-sap-emarsys/"
---

做营销云项目的人都熟悉这种场景：客户告诉你，下周一要发一封"夏季新品 100 美元以下女鞋"的邮件，要按门店半径 50 公里圈定收件人，邮件里要动态拉商品图、价格、库存状态。

过去这件事在 Emarsys 上怎么做？运营要在后台手动选 SKU、拼模板，开发那边要事先把整份商品 feed 通过 SFTP 推过来——而且通常是天级全量。等到周一邮件发出去，feed 里那批限时折扣已经跑完了。

这是 Engagement Cloud 这两年一直在拆的一个旧约束：商品目录是一份静态资产，营销系统对它只能读，且读得不够细。Q4 2025 release 把 AI-assisted Product Finder 推到 GA，并把 product data 改成 API-loaded，看起来是"加了个 AI 助手"，实际上动的是更底下那一层。

## Product Finder 真正变化的不是前端

如果只看前端演示，AI-assisted Product Finder 给运营提供了一个自然语言搜索框：输入 women's shoes under $100，下拉出对应 SKU，然后把它们插进邮件正文。这看起来是一个 AI 给运营提效的功能，写一段 prompt-to-search 就能上。

但前端这层之所以能跑起来，靠的是 catalog 这一侧的重构——商品数据不再以 feed 文件的形式被周期性导入，而是以 API endpoint 的方式被持续更新。SAP 的官方表述是 AI-assisted Product Finder now supports API-loaded product data，这一句的工程含义是：商品对象被显式建模成可被 query 的资源，价格、库存、本地化字段、可见性都是可独立更新的属性。

一旦走到这一步，自然语言搜索才有意义。否则你搜出来的 100 美元以下女鞋很可能是上周的库存快照。

![AI-assisted Product Finder 数据流](/Users/I319510/wechat-publish/2026-05-30-engagement-cloud/diagram.png)

## Catalog 接入层：从 SFTP 全量到 API 增量

过去 Emarsys 接外部商品系统的标准做法是 SFTP——每天一次或每几小时一次，把整份商品 CSV 推过来，平台再做匹配和导入。这套方案在邮件营销时代够用，因为商品的"准实时"对邮件来说不是必需。

到了 Conversational Channel（WhatsApp、LINE）和 Mobile Wallet 这一波，准实时变成了刚需。一张已发出的 wallet pass 要在用户开启时显示当前库存和价格；一条 WhatsApp 推送如果带商品卡片，发出那一刻商品已下架就是事故。

Q4 release 把 Standard Sales Data API 推到 GA，配合 Product Data Feed 的 API 化路径，正式把数据接入层改成了三件事并行：

- 商品主数据走 Product Data API（增量 + 字段级更新）
- 销售事实数据走 Standard Sales Data API（线下线上合流，支持退款、本地货币）
- 行为与系统事件走 Engagement Events API（任意外部源 → 实时分群与触发）

这三条 API 把过去散落在 SFTP、CSV 导入、Connector 配置里的接入方式收拢成同构的 REST 通道。从架构上讲，Engagement Cloud 不再只是一个接收数据的营销 SaaS，而更像一个把商品、行为、销售三类对象都暴露为 API 的数据平台，营销编排只是它的一种使用方式。

## 一个典型的 Product Data 调用长什么样

以下示例基于公开文档结构改写（具体字段以 Engagement Cloud 实施手册为准），便于理解 API-loaded 模式下商品对象的更新粒度：

```http
POST /v3/product-data
Content-Type: application/json
Authorization: Bearer <token>

{
  "products": [
    {
      "sku": "SHOE-W-2604",
      "locale": "en-US",
      "price": 89.00,
      "original_price": 129.00,
      "availability": "in_stock",
      "category": ["women", "shoes", "running"],
      "updated_at": "2026-05-30T10:12:00Z"
    }
  ],
  "mode": "upsert"
}
```

关键不是这个 payload 长什么样，而是它意味着：商品的某个属性变了，调一次 API 就行；不需要重传整份目录，也不需要等下一个调度窗口。AI-assisted Product Finder 在搜索时拿到的就是这个最新状态。

## 自然语言查询，落到底层是哪几步

当运营在 Product Finder 里输入 women's shoes under $100，Engagement Cloud 在后台大致做了这几件事：

- 自然语言解析：把短语映射成结构化条件——category=shoes、gender=women、price<100、locale=当前营销活动的 locale
- Catalog 检索：在已索引的商品对象上跑过滤；这里和传统全文检索的区别是，可以混用文本相似度（running shoes 和 trail shoes 的语义距离）和结构化字段过滤
- Locale 与 visibility 校验：剔除当前活动地区不可见的 SKU、剔除已下架的、剔除供应链标记暂停的
- 回填到内容块：选中的 SKU 通过模板变量进入邮件 / Web / Push 的内容块

这套链路里有意思的是第二步——Engagement Cloud 没有公开声明它用了 embedding 做语义检索（也没披露用的什么模型），但从产品行为看（支持自然语言、支持模糊语义），catalog 这层一定做了向量化或类似的语义索引。这也是为什么 API-loaded product data 是前提：要做向量索引，就得能拿到商品的文本描述、属性、变体——靠 SFTP 的 CSV 是做不到字段级更新的。

## 和 Predictive Segmentation、Engagement Events 是什么关系

Product Finder 单看是一个选品工具，但放到 Engagement Cloud 当前的产品矩阵里，它其实是三条线的交汇点：

- Predictive Segmentation 解决"发给谁"——AI 预测用户在某个渠道（Email、Push、Web）的转化倾向
- Engagement Events 解决"什么时候发"——外部事件（订单、loyalty 升级、cart abandon）触发自动化
- AI-assisted Product Finder 解决"发什么"——动态选品 + 价格 + 库存

过去三件事是分开配置、分开维护的：分群在一个界面、自动化在另一个界面、商品块在邮件编辑器里手动拼。Q4 release 的整体方向是把这三件事在数据层串起来——它们共享同一份商品对象、同一份联系人画像、同一份事件流。从实施视角看，未来一个完整的实时个性化项目会越来越像在配置一个 query graph，而不是拼三个独立配置。

## 和 Commerce Cloud 的对接，多了一条更直接的路

过去 Emarsys 接 SAP Commerce Cloud 主要有两条路：CDC 同步用户与同意状态（这条历史里写过），SFTP 推商品和订单数据（很多老项目还在用）。Q4 release 上线了 Engagement Events 与 SAP Commerce Cloud Integration 之后，Cart 更新、购买、浏览这一类行为可以从 Commerce 直接走事件通道进入 Engagement Cloud——这套连接配好以后，Integration Monitor 还能反向追事件，排查谁的 cart abandon 没触发邮件。

这条路的意义在于：Commerce Cloud 这一侧不需要再为营销系统单独维护一份导出逻辑。事件是 Commerce 本来就会发的（OCC 内部已经有事件总线），现在只是在出口处接到 Engagement Cloud 的 ingest 上。

## 什么场景适合切到 API-loaded，什么时候先别动

适合的场景：

- 商品 SKU 在 5 万以上，且每天有几千条价格 / 库存级别的变化（服饰、3C、跨境零售）
- 用了 Conversational Channel 或 Mobile Wallet，这两个渠道对实时性容忍度低
- 营销活动跨多个地区、多 locale，每个地区可见 SKU 子集不同——这是出海品牌最常见的形态

先别急着切的场景：

- 商品目录一周才更新一次，一份 CSV 就够用——切 API 反而要重写 ETL
- 现有 SFTP 链路稳定且监控到位，团队没有 API 错误重试和 backpressure 处理经验
- Engagement Cloud 还没在你这边上 production——先把 contact、segment、journey 跑通，再谈 catalog 实时化

## 两个会踩的坑

第一个坑：API-loaded 不等于零延迟。商品对象 upsert 进 Engagement Cloud 之后，进入索引、被 Predictive Segmentation 重新评分、被 Product Finder 检索，整个过程仍然有秒到分钟级的传播延迟。要做"5 分钟内必达"的促销推送，需要把 Engagement Events 直接和模板里的硬编码 SKU 绑定，而不是依赖 Product Finder 现搜。

第二个坑：自然语言搜索的可解释性。women's shoes under $100 命中的那几个 SKU，运营如果说不清楚为什么是这几个，合规审计来问的时候就被动了——尤其在欧盟运营的客户，DSA 和 Digital Markets Act 都对自动化决策的可解释性有要求。建议做项目时，把 Product Finder 的查询条件 + 命中 SKU 列表落到日志，留至少 6 个月。

回到开头那个场景。"夏季新品 100 美元以下女鞋"这种 brief，今天在 Engagement Cloud 上不再需要前端运营手动去 catalog 里挑——但要让它真的跑得对，关键不是 AI 助手有没有装，而是商品数据这条链路有没有切到 API 模式、与 Engagement Events 和 Predictive Segmentation 是不是共用同一份对象。前端那个搜索框只是结果。

参考来源：https://emarsys.com/learn/blog/2025-in-review-the-year-of-ai-powered-innovations-for-sap-emarsys/
