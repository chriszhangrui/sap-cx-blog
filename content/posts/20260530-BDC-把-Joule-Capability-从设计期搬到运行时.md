---
title: "BDC 把 Joule Capability 从设计期搬到运行时"
date: 2026-05-30T01:10:00+08:00
draft: false
tags: ["SAP CX", "BDC", "Joule", "Knowledge Graph", "A2A", "技术深度"]
description: "Joule 上一份 YAML 一个用例的扩展模型，到了跨系统、问法发散的场景就堆不动了。SAP 官方原型把它拆成 Joule、A2A、Knowledge Graph、OData 四层，BDC 从语义层接上来。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/beyond-joule-capabilities-a-knowledge-graph-driven-runtime-for-sap-business/ba-p/14380937"
---

> 本文是技术深度解读，话题：BDC + Joule。

在 Joule 上做一个新功能，今天最常见的路径是这样：写一份 YAML 把 Capability 定义出来，把 OData API 绑上去，把字段映射好，过一轮发布。再来一个新问题——再来一份 YAML。

这套机制在场景数量可控的时候是顺的。可一旦用户开始问跨系统的问题，或者同一个意图换了二十种说法，YAML 数量就开始指数膨胀。一个 SAP 老用户，光采购侧就可能问出几十种"未结订单"的变体——按供应商、按金额、按交期、按工厂、按工厂×物料组×状态——你不可能为每种变体都开一份 Capability。

最近 SAP Community 上有一篇文章给出了一种思路：把 Capability 从设计期拆下来，搬到运行期，让一个跑在 BTP 上的 Knowledge Graph（知识图谱）去承担"哪种问题对应哪种 OData 模式"的决策。这是 SAP 自己的人写的原型，跑在 BTP Cloud Foundry 上，用 A2A 协议（Agent-to-Agent，v0.3.0）接 Joule，用 PostgreSQL 当临时知识图谱存储。

这篇文有意思的地方不在于它发了什么新产品，而在于它清楚地把 Joule + A2A + Knowledge Graph + Business Data Cloud（BDC）这四层放在一起，画出了一种"运行时分层"的架构。今天就把它拆开看看。

## 为什么是"运行时"，不是"设计时"

先把核心思路说清楚。原作者把整个对话流程分成四层，每一层只负责一件事：

- **Joule 拥有对话**——用户的自然语言入口，渲染最终的卡片
- **A2A 拥有路由**——把意图按系统/领域分发给具体的 Agent，本身很薄
- **Knowledge Graph 拥有访问模式**——已经被验证过的 OData 查询模板都存在这里
- **OData 拥有数据**——S/4、Ariba、SuccessFactors 这些源系统

这种分法的目的，是让"新增一个系统""调整一个业务术语""换一种问法"这三件事互不影响——动哪一层只改哪一层。

关键的转移在第三层。传统 Capability 模型里，一个查询长什么样、用哪个 API、过滤哪个字段，全部在 YAML 里写死，是设计期的活。这套模型把这件事推到运行时——KG 里存的不是 Capability 定义，而是**已经被执行成功并且被人审过的 OData 模式**。新的问题进来，先去 KG 查模板，命中就直接组装 $filter 跑，不需要任何新的 YAML。

## 一次问答跑出来是什么样的

原文给了一个例子，挺能说明问题。用户对 Joule 说："Show me my active purchase orders from supplier Bosch"——这个问题事先没有任何 Capability 为它准备过。

运行时的链路是这样的：

```
1. Joule 把意图通过 A2A endpoint（JSON-RPC 2.0）转发出来
2. KG.match("active_PO") 命中已验证模板：
   GET /A_PurchaseOrder?$filter=Status eq 'Open'
3. SlotExtractor 从原句里抽出 "Bosch" → SupplierID
4. 在模板基础上拼出最终 OData 调用：
   GET /A_PurchaseOrder?$filter=Status eq 'Open'
        and Supplier eq '0001234'
5. 用 KG 里和模板一起存的 ResponseTemplate 渲染成
   Joule Card——和原生 Capability 视觉一致
```

用户感知不到背后是个外挂 Agent。但底层架构和传统 Capability 完全不是一回事。

## 和传统 Capability 的差别

SAP Architecture Center 文档里给 Joule 自定义 Agent 列了两条路：低代码走 Joule Studio，专业代码走 Bring Your Own Agent（BYOA，自建 A2A endpoint 注册成 Joule Scenario）。原型走的是后者，但它在 BYOA 上又加了一层——把"一个 endpoint 对应一个用例"变成"一个 endpoint 解析所有 KG 已知的模式"。

几个具体差异值得记一下：

- 传统 Capability：一个用例一份 YAML，API、过滤、字段全写死；新问法 = 新版本
- KG-Driven：模式存在 KG 里，不在 YAML 里；运行时做 Slot 提取拼 filter；新供应商、新年份、新单据，同一份模式都能处理
- 治理对象也变了——不再是"这份 YAML 谁负责"，而是"这条 OData 模式谁审核过"，由领域负责人盯着 query+result 这一对，开发反而往后退了一步

什么时候用哪种？原文给的判断很务实：场景简单、范围明确、查询可预期，传统 Capability 就够了；跨系统、问法发散、探索式查询多，才有必要上 KG-Driven。这两条路并不互斥——Capability 的每一次构建，本身就是在往 KG 里灌验证过的模式。

## BDC 在这个故事里的位置

这是这篇文章对 BDC 这条线最有价值的判断。

这套架构不依赖 BDC——纯靠 OData 也能跑——但有个绕不过去的问题叫"冷启动"。KG 里没有模式之前，几乎所有意图都会落到"暂不支持"。光靠用户问、人工审、慢慢攒，覆盖率上不来。

BDC Data Products 的设计目标，恰好是把跨应用的业务定义、领域分类、实体关系做成"语义就绪"的模型。如果能拿这些模型给 KG 做**初始播种**——先生成一批 provisional（待验证）模式，再用真实的 OData 调用把它们升级成 validated——冷启动问题就被压低了一个量级。

原文里有句话挺关键："Raw OData 告诉你数据的形状，BDC 告诉你数据是什么意思"。OData 给的是 EntityType、字段名、关系外键，BDC 给的是这些东西在业务语境里到底对应什么——这件事在做"自然语言到查询的匹配"的时候差别巨大。一个用户说"未付款的发票"，OData 不会告诉你 PaymentStatus = 'Open' 才是答案，BDC 的语义层有可能告诉你。

还有第二条 BDC 通道：分析型问题。"今年和 Bosch 的累计采购额"这种跨期聚合查询，靠 OData 实时跑性能上吃不消，需要预计算或聚合数据，这是 BDC 数据产品天然适合承接的领域。原型还没做，但同一个 A2A 路由层接进去就行。

不过原作者也写了一个我比较欣赏的"诚实备注"：BDC 的语义模型给冷启动一个起点，但 tenant 级的客制、被限制访问的实体、版本差异，依然要靠 live validation 才能被信任——BDC 把冷启动的痛减轻了，但消不掉验证这一步。

## 真正的工程难点在哪

这篇文章另一个让我觉得值得读的地方，是它把"哪些事架构图上看着挺顺、上生产没那么简单"列了出来。

**意图匹配。**原型用 PostgreSQL 全文检索找模式，速度够快但同义词上很脆——"open orders" 和 "not invoiced" 在某些上下文下是一回事，"cancelled" 在不同模块含义又不同。生产级应该上混合检索：全文 + 向量相似度，让语义近似补全词法缺口。

**Slot 提取。**把"Bosch"这个自由文本对应到 SupplierID 听上去简单，实际上用户会输入部分名、别名、错拼，生产级要做 alias、模糊匹配，甚至接 master data 查表——本质就是个迷你 MDM 的活。

**授权。**这是原型现在最弱的一环。当前用 XSUAA JWT 做接入认证，但调 OData 用的是 service token——也就是说没有按用户做行级权限。生产路径是 BTP Destination Service 的 Principal Propagation，让 OData 调用以用户身份执行。SAP Reference Architecture 的标答是 IAS App2App + 命名用户上下文（走 Agent Gateway），原作者也明确说这是 roadmap 里的事。

**Schema 漂移。**S/4HANA 升级有可能悄悄改字段，已经验证过的模式可能某天突然不工作。这条 KG 治理体系里要靠"N 天没被执行的模式标记复审"来兜底。

还有一个更微妙的事，原作者自己点出来了：他文中说的 "knowledge graph"，其实当下的实现是个**已验证模式存储**——PostgreSQL 表 + 全文索引，不是真正带类型化关系的图。要变成真知识图谱，还得加一张边表，把"采购发票 → P2P → 付款"这种流程链条编码进去。这一步是规划中的下一步，原文坦白讲了。

## 和 Agent Gateway 的关系

SAP 官方为自定义 Agent 接 Joule 准备的入口叫 Agent Gateway，按 SAP Architecture Center 的说法，它会承担 A2A 0.3.0 端点和 IAS App2App 鉴权。这个组件目前还**未 GA**。

原型选择了一个过渡路径：自己在 BTP Cloud Foundry 上搭一个 A2A endpoint，前面挡着 SAP AppRouter 做 XSUAA。等 Agent Gateway GA 之后，入口会从"自建 AppRouter"切到"SAP 托管端点"，鉴权模型也会改——但下面 KG + OData 那两层一个字都不用动。这是分层带来的实际收益。

## KG 的 Schema 长什么样

原型把 KG 拆成三张主表，分别承担节点角色：

```
kg_services    -- 哪些 OData 服务被纳管
kg_entities    -- 业务实体（A_PurchaseOrder, Supplier, ...）
kg_patterns    -- 已验证的查询模式：
                  intent_keywords / entity_id /
                  filter_template / response_template /
                  validated (bool) / last_executed_at
```

每一条 pattern 的治理是这样的：成功执行一次，自动 propose 一条 provisional 模式；领域负责人审完才置 validated=true，运行时才会信任它。每一次 validation 的证据——查询、响应、时间戳、用户上下文——都附在模式上。N 天未被使用的模式标记待复审，**不删**，因为业务季节性的查询年中可能根本不出现。

分页这件事也用了一个细节：A2A 协议里有个 contextId 字段，原型把 skip 偏移和 pattern ID 编进去——用户说"再来 10 条"的时候，下一轮直接拿同一个模式从对应偏移量继续跑，不再走一遍 KG 查找。这是把对话状态外置的一个干净做法。

## 什么样的项目可以借鉴这套思路

这不是个能直接落地的方案——它本身是个 personal prototype，作者自己写得很清楚不代表 SAP 官方路线图。但它指出的方向，对几类客户值得参考：

- **跨系统数据问答场景多的全球化运营企业。**采购在 S/4，合同在 Ariba，差旅在 Concur，员工在 SuccessFactors——用户从来不按系统提问。这种诉求用一份份 Capability 堆是堆不完的，A2A + KG 这条路至少给了一个可演进的架构。
- **已经在用 BDC 做语义层、想往 AI 入口接的出海企业。**这种客户的 Data Products 已经在跑，把它当成 KG 的播种源，是把现有投资延伸到对话场景的最经济路径。
- **SI 合作伙伴。**纯做 Capability 一锤子买卖，存量越大维护越贵，未来的差异化会从"我能交付多少 Capability"变成"我能策展多少高质量的 OData 模式"。这套机制把交付的对象悄悄换掉了。

什么时候不要碰：单系统、查询模式有限、用户群体小且查询稳定的项目，老老实实走 Joule Studio 或 BYOA 一份 YAML 就行，没必要为了"未来可扩展性"先付一笔架构税。

## 几条踩坑警示

- 别把 Principal Propagation 留到上线前才做——service token 跑起来快，但接到生产那天要补的洞最深
- 全文检索做意图匹配会撞墙，向量召回这一步建议从一开始就纳入设计，否则 KG 一大就要重写
- BDC 的语义模型不是"开箱即用的 KG 种子"，每一条经过 BDC 来的 provisional 模式，都还要有 live validation 这一关，不然 tenant 级客制会让你的 Agent 自信地答错
- Agent Gateway GA 之前自建 endpoint 没问题，但鉴权层要按"将来一定会换"的方式封装，别把 XSUAA 的细节漏到业务逻辑里

## 写在最后

这篇 prototype 文章最难得的不是它给了多少代码，是它把"为什么这么设计"说透了。Joule 拥有对话、A2A 拥有路由、KG 拥有访问模式、OData 拥有数据——四件事各归其位，新增一个系统不影响其他人，新增一种问法只动 KG，鉴权方式换了不影响下游。这种分层不是 PPT 上画出来的整齐，是工程语义上的整齐。

至于"谁该拥有这个 KG"——这是原作者最后留下的开放问题。我倾向于认为：**领域负责人持有，平台团队提供运行时**。把语义化的资产留在业务侧，把生命周期管理留在工程侧，这两件事不应该混。

如果你正在做 SAP 上的 AI 项目，无论是接 Joule 还是自建对话入口，这篇 prototype 值得花半小时翻一遍——它解决的，本来就是"我画了 50 个 Capability 之后还是覆盖不完"这种烦人的现实问题。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/beyond-joule-capabilities-a-knowledge-graph-driven-runtime-for-sap-business/ba-p/14380937
