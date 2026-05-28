---
title: "Sales Cloud V2 把 GenAI 装在报价单的入口处"
date: 2026-05-28T21:10:00+08:00
draft: false
tags: ["SAP CX", "Sales", "Sales Cloud V2", "GenAI", "BIE", "技术深度"]
description: "BIE 不是新的 AI 模块，它是把生成式 AI 嵌进 email-to-quote 这一段最脏的活里。Elements、Classes、Scenarios 三层配置才是真正的看点。"
source_url: "https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/how-business-information-extraction-in-sap-sales-cloud-v2-accelerates-sales/ba-p/14355774"
---

做过 Sales Cloud 项目的人都见过这种邮件——客户用一段散文式的描述发来报价请求：要 24 套 VK-220 阀门、12 套 FA-90 滤芯、发到 Porto 工厂、第 39 周设备停机检修前必须到货、价格按"现有协议"走、附件里还有完整清单。这种邮件落到销售手里，第一反应不是开报价，是先做翻译——把自然语言里夹着的型号、数量、地点、时间窗、商务上下文一项一项拆出来，再录入 CRM。

这一步过去几乎没有自动化方案。OCR 解决格式问题但拿不准语义，规则引擎能匹配关键词但碰到上下文就崩，传统 NLP 模型训练成本高、客户场景一变就要重训。结果就是销售把大半时间花在做"录入员"——这件事在 Sales Cloud V1 时代就是死结。

现在 SAP 在 Sales Cloud V2 里给了一个名字叫 BIE 的能力（Business Information Extraction，业务信息抽取），用 GenAI 处理这段最脏的活。值得看的不是"又来一个 AI 功能"，而是它怎么把 GenAI 接进一个本来很硬的 CRM 配置模型里——三层结构，admin 可配，每个客户场景都能定制。

## BIE 解决的问题，到底卡在哪一段

BIE 在官方文档里被定义为：用于配置抽取逻辑、从邮件里 AI 辅助生成 sales quote 的能力。这句话很容易被当成营销词，但仔细看会发现它把切入点划得很精——不是替代销售判断价格、不是自动审批、不是替代议价，而是只把"读邮件—识别字段—填进 quote"这一段拿出来做。

这个边界划得有意思。Sales Cloud V2 整体在 2026 年的方向是 AI 嵌入各层（参考 Q1 2026 release），但具体到 BIE，它选了一个执行链路上的局部岗位：报价单入口处的非结构化数据转结构化数据。前面（销售判断）和后面（定价、审批、合同）都不动，只把中间最重复、最低价值的那一段交给 GenAI。这种边界感比"AI agent 替你做销售"那种宽叙事要可靠得多。

## 三层配置模型：Elements、Classes、Scenarios

BIE 真正的设计点在它的配置模型。SAP 把抽取逻辑拆成三层，admin 在管理界面里组合：

![BIE 的三层配置模型](https://community.sap.com/t5/image/serverpage/image-id/388430i01201BFE279120BA/image-size/large?v=v2&px=999)

**Elements**——明确字段，从输入里提取的具体值。标准 elements 包括 Product ID、Product Name、Quantity、Unit、Requested Date 这类标准报价字段；客户可以扩展自定义 element，比如 Customer Material Number、合同引用号、资产或序列号、客户参考编号。这一层接近传统的实体识别（Named Entity Recognition），但模型基于 GenAI 重新做了。

**Classes**——推断分类，从内容里推导的归类标签。这一层是 BIE 比传统 NER 更值钱的地方。delivery priority 是从"week 39 shutdown 前"推断出 Urgent；request type 是从"replacement"或"new product"推断出来的；commercial context 是从"按现有协议走"推断出 contract-linked。这些分类不是输入里有的字面值，是从语义里推出来的。

**Scenarios**——业务场景，把 elements 和 classes 组合成一个具体的用例。同样是 email-to-quote，做备件复购的客户和做项目型订单的客户配的 scenario 完全不一样。前者可能就用标准 elements 加 delivery priority 一个 class；后者要加 contract reference、project ID、asset ID 这些 element，再加 request type、commercial context、product family 这些 class。

为什么是这三层？因为如果只暴露一个"训练你自己的模型"接口，门槛就过高，admin 配不动；反过来如果只给一个固定模板，每家客户场景的差异又吃不进来。Elements + Classes + Scenarios 这套结构，本质是把 GenAI 的 prompt 模板拆成"显式槽位 + 推断槽位 + 业务上下文"三层，admin 在 UI 上组合就行，背后真正的 prompt 工程被封装掉了。

## 从原文档示意，看 SAP 把 BIE 放在哪一段

原文里 SAP 给了一个流程图，把 BIE 摆在 quote 流程的最前段——客户邮件进来，BIE 抽取，销售拿到一个"更好的草稿"开始干活，后面定价、校验、商务对齐都不变。这个位置很关键：BIE 不接管 quote 的生成决策，只接管 quote 草稿的生成动作。销售依然要复核、调整、盖章，只是起点从"白纸"变成"已经填好 70% 字段的草稿"。

这种摆放方式让 BIE 落地的争议小得多。如果直接做 auto-quote 全自动，合规、定价权、商务关系都要重新设计；只做 draft 辅助，业务流程和审批权限都不动，admin 配完就上线。

## 配置示例（基于原文 scenario 设计）

```yaml
# 简单 scenario - 备件复购场景
scenario: "email_to_quote_simple"
elements:
  - product_id        # 标准
  - product_name      # 标准
  - quantity          # 标准
  - requested_date    # 标准
classes:
  - delivery_priority: [urgent, standard, planned]

# 复杂 scenario - 项目型订单场景
scenario: "email_to_quote_project"
elements:
  - product_id
  - quantity
  - customer_material_number   # 自定义
  - contract_reference         # 自定义
  - asset_serial_reference     # 自定义
classes:
  - request_type: [new_product, replacement]
  - delivery_priority: [urgent, standard, planned]
  - commercial_context: [contract_linked, project_driven, ad_hoc]
  - product_family: [spare_part, accessory, consumable, finished_good]
```

同样的 BIE 引擎，给到不同客户的 scenario 配置不同。这意味着实施方不需要为每个客户重新做 AI 模型，只需要在 admin 界面里组合 elements 和 classes 就能做出客户的报价场景定义。这套抽象比"我帮你训一个专属模型"工程量低得多——但前提是底层 GenAI 模型对你的行业语言和产品命名有足够覆盖。

## 与竞品的差异点在哪

Salesforce 的 Einstein for Sales 也在做邮件解析，但更偏向"销售助手"——总结邮件、提示下一步动作、推荐产品。它的边界更宽，但也更难落到"自动建一个 quote"这种具体动作上，因为 SF 对 quote 对象的写权限和审批链需要单独配。Microsoft Dynamics 365 Sales Copilot 走的是另一条路，重交互、轻配置，admin 几乎没有 elements/classes 这种结构化抽取层可调。

BIE 的差异是它假设了 quote 这个对象的存在，并且把抽取目标 hard-link 到 quote 字段上。这是个"上窄下宽"的设计——上层抽取范围被场景限定（只做 quote 草稿），但下层配置模型给得很细（admin 能定制每一个 element 和 class）。这种设计适合销售流程已经标准化、quote 模型相对固定的客户；不适合销售流程极度个性化、连 quote 字段都不固定的项目型公司。

## 实施落地——什么样的客户该上、什么时候不要碰

**建议优先评估的场景**：

- **跨境/出海制造商**，海外渠道用邮件下单为主，产品 SKU 数量大、命名规范化（用 ID 不用描述），BIE 抽取准确率会高，落地成本低。
- **跨境分销商**，海外终端客户场景固定（备件、复购、项目订单三类为主），用三个 scenario 就能覆盖大部分入口。
- **在华外企的总部对接团队**，海外总部的报价请求邮件本来就用英文，GenAI 模型对英文场景成熟度高，是相对低风险的试点入口。

**建议先观望的场景**：

- 产品命名极度自由文本化、客户在邮件里用绰号或行业黑话——抽取质量会很差，先做产品主数据治理再说。
- 报价里 80% 的逻辑在定价、配置、规则引擎那一段（典型的 CPQ-heavy 行业），BIE 只解决最前 20%，性价比有限。
- 合规要求"所有 quote 必须人工录入留痕"，BIE 的 draft 模式在审计上要重新对话，先把流程权限模型谈清楚。

**实施踩坑提示**：

- KPI 起手不要选成本节省，选时延（intake 到 draft 的小时数）和首次报价完整度——前者立刻可见，后者反映 elements/classes 配置质量。
- 一开始只配一个最高频的 scenario，跑两周看抽取准确率，再加 element 或 class。多 scenario 同时上，分不清是模型不准还是配置不对。
- 客户邮件里的产品命名要在 BIE 上线前先治理一轮——这一步比配 scenario 重要得多，没干净的 product master 就没准的抽取。

## 最后一点观察

BIE 在 SAP 整个 AI 路线里属于"小切口、深插入"的那种功能。Joule Studio 那一线在做的是宽 agent 平台，BIE 这一线做的是把 GenAI 嵌进具体业务对象的入口处。两者并不冲突——Joule 偏交互层，BIE 偏数据入口层。做出海项目时，Joule Agent Hub 在中国还没区域化方案的情况下，BIE 这种嵌入业务流的局部 AI 能力反而更先有路径上线。

从架构看 BIE 真正给到的东西，不是"我们也有 AI"，而是一套把 GenAI 模型工程封装成 admin 可配置抽取规则的设计。Elements、Classes、Scenarios 这三层抽象——这一套抽象比模型本身更值钱。

参考来源：https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/how-business-information-extraction-in-sap-sales-cloud-v2-accelerates-sales/ba-p/14355774
