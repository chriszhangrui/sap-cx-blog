---
title: "Joule 的 Pro-Code 路径，到底在解决什么"
date: 2026-05-22T10:00:00+08:00
draft: false
tags: ["SAP CX", "AI", "Joule", "BTP", "技术深度"]
description: "Joule Studio CLI 把 Agent 扩展落到 6 层 YAML：DTA→Capability→Scenario→Function→Action。可视化够用，Pro-Code 才能进 Git。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-sap/building-custom-joule-capabilities-getting-started/ba-p/14390050"
---

在 Joule Studio 的可视化编排里点几下，确实能拉出一个能跑的 Agent。但凡做过两个项目以上的人都知道，到了第三个 Capability，可视化界面的复用、版本控制、CI/CD 就开始挡路。SAP 在 2026 年初放出了 Joule Studio CLI 的 Pro-Code 路径，把所有产物落到一组 YAML 文件上。这条路径不是给所有人准备的，但它解释了 Joule 这套 Agent 框架在底层到底是怎么组织的。

最近读了 SAP Community 上 david_bizer 的一篇起步教程，把这条 Pro-Code 路径的目录结构、认证、部署链路完整走了一遍。借这篇文章，把 Joule 的扩展架构拆开看看。

## DTA 是什么——Joule 的扩展骨架

DTA 全称 Design Time Artifacts，是 Joule 把所有可扩展产物组织起来的层级模型。这棵树有 6 层，从最粗的项目清单一直到最细的原子动作。理解这棵树，比理解任何一个 YAML 字段都重要。

```
Digital Assistant (da.sapdas.yaml)
└── Capability (capability.sapdas.yaml)
    └── Scenario
        └── Function
            └── Action Group
                └── Action
```

这 6 层每一层都有明确职责，混淆任何一层都会在部署时被打回来。

- **Digital Assistant**（da.sapdas.yaml）是项目清单，决定你这个工程是要发一个独立助手出去（部署完会拿到一个独立 URL），还是把内容塞进 SAP 出厂的统一助手 sap_digital_assistant。如果只是后者，da.sapdas.yaml 这个文件根本不需要写。
- **Capability** 是独立的能力包。每个 Capability 一个文件夹，一个 capability.sapdas.yaml，namespace 必须写死成 joule.ext。写错就会收到一条莫名其妙的 401："User is not authorized to deploy sap capabilities"——这是教程里特别标出来的坑。
- **Scenario** 是会话上下文，但它的 description 字段不是给开发者读的注释。Joule 平台运行时会读这段 description，扔给 LLM 去判断"用户这句话该不该路由到这个 Scenario"。换句话说，这是路由提示词，不是文档。
- **Function** 是逻辑单元，定义参数槽位（slots）、响应模板（response_context）和动作组的拼装。
- **Action Group** 是带条件的动作组合。多个后端调用可以按 condition 串起来。
- **Action** 是原子操作。api-request、set-variables、message 是常用的。SpEL（Spring Expression Language）表达式在这一层落地，BTP destination 通过别名引用，环境差异隔离在这里。

## 为什么要把 Scenario 和 Function 拆开

这是 Joule 这套架构里最值得注意的设计——也是和很多通用 Agent 框架不一样的地方。

一般做 LLM Agent 的时候，习惯把"用户意图识别"和"工具调用"放在一个 Function Calling 协议里：LLM 看着工具描述，自己挑工具，自己填参数。Joule 把这件事拆成了两层。Scenario 那一层，LLM 只做一件事：根据 description 判断"要不要把这条用户消息交给这个能力"。一旦命中，参数提取（slots）走的是平台规定好的字段，Function 拿到结构化参数后做的是确定的编排——HTTP 调用、SpEL 计算、变量传递、按条件分支。

这个设计放弃了一部分灵活性。LLM 在 Function 内部不会自由发挥，能干什么完全由 YAML 决定。换来的是企业级场景里更想要的两件事：可审计（每一步都可以被日志追到具体 action）和可重放（同样的输入走出来的执行路径是确定的）。这是给 SAP 客户做出来的取舍——客户买的是稳定，不是惊喜。

## 一个能跑的最小例子

教程里给的样例是查 Northwind 的 OData 服务。Capability 配置长这样：

```yaml
schema_version: 3.27.0
metadata:
  namespace: joule.ext
  name: northwind_products
  version: 1.0.0
  display_name: Northwind_Products
  description: >
    Look up product information from the Northwind OData service
system_aliases:
  NorthwindService:
    destination: Northwind
```

system_aliases 这一段是关键。NorthwindService 是逻辑别名，destination: Northwind 指向 BTP Cockpit 里配置好的 Destination。Function YAML 里只写别名，从不直接出现 URL。生产环境换 Destination，源码不改。

Function 里调 API 的那段长这样：

```yaml
action_groups:
  - actions:
      - type: api-request
        method: GET
        system_alias: NorthwindService
        path: "/Products?$filter=...&$top=1"
        result_variable: api_result
  - condition: api_result.status_code == 200 && api_result.body.value.size() > 0
    actions:
      - type: set-variables
        variables:
          - name: product
            value: <? api_result.body.value[0] ?>
```

注意两种 SpEL 写法。condition 字段里直接写表达式，不要包 `<? ?>`，包了反而编译报错。path、content、value 这种字符串字段要嵌入表达式时才用 `<? ?>` 包起来。这两种写法的区别教程里被特别强调过——也是新手最容易栽的地方。

## 认证和部署的真实路径

Joule CLI 的认证走的是 IAS（Identity Authentication Service）OpenID Connect。整个链路要做四件事：在 IAS 里建 OIDC 应用、加 Cli2Joule 依赖（Application 选 das-ias，API 选 joule-dt-api）、生成 client secret、最后跑 joule login。

用户角色至少要拿到三个：end_user、capability_admin、extensibility_developer。原文作者特意点出来：extensibility_developer 是最容易被遗漏的那一个，缺了它部署命令直接报授权错误。

完整开发循环的命令也很简单：

```bash
joule lint                              # YAML schema 校验
joule compile                           # 产出 .daar 部署包
joule deploy -c -n "my_assistant"       # 编译并部署
joule launch "my_assistant"             # 拉起测试
joule update "sap_digital_assistant" \
  --capability-file capability.sapdas.yaml   # 更新到统一助手
```

最后一条 joule update 的用法很值得记住。如果不想发独立助手，而是把 Capability 推到 SAP 出厂的 sap_digital_assistant 里，就用 update。这样这个 Capability 会和 SAP 自带的所有 Joule 能力一起，在 S/4HANA Private Cloud 的 Fiori Launchpad 里被调用。

## Pro-Code 路径适合谁

这条路径的判断标准不复杂，但要诚实地评估三件事：

- **有没有可复用的 Capability。** 如果只是给某个部门做一个查库存的小助手，Joule Studio 可视化编辑就够了。一旦预期要做 5 个以上的 Capability、要在多套环境之间迁移、要把 YAML 进 Git 管起来，Pro-Code 才有意义。
- **团队里有没有 SAP BTP 的工程能力。** YAML 容易上手，但 IAS 配置、BTP Destination、SpEL 表达式、joule.ext 的 namespace 约束这些坑，没踩过 BTP 不会一上来就懂。出海企业里如果团队对 BTP 还在摸索阶段，先做一个原型再做项目化迁移会稳妥得多。
- **是否需要和代码 Agent 互通。** Pro-Code 路径走的是 Joule 平台的原生扩展，主要解决"配置驱动 + 后端调用"的场景。如果你的 Agent 本身就是用 Python/LangChain 写的，更适合走 A2A（Agent-to-Agent）协议。两条路线不互斥，可以混用。

什么情况下不要碰：业务变化频繁、规则没沉淀清楚的场景，把 YAML 当代码写会让运维很痛苦——每次小改都要走 lint/compile/deploy 一遍。

## 几个写在最后的判断

Joule 这套 DTA 模型，是 SAP 把多年来在企业 Agent 上的设计经验沉淀出来的。和那些只靠 LLM 自由调度的 Agent 框架比，它选择把"路由"和"执行"严格分层、把执行路径写成确定的编排。这个设计在演示场景里没那么炫，但它解决的是企业客户最在意的那几件事：可控、可审计、可上线。

对出海企业来说，关注 Joule 的 Pro-Code 路径有两个直接的现实意义。一是 SAP S/4HANA Cloud Public/Private Edition 上的 Joule 已经具备开放扩展能力，做项目时不必再依赖 SAP 标准 Capability 等版本更新；二是 BTP 的 Destination + IAS 这套组合，本来就是 SAP 客户出海后处理多区域、多租户的标配，Joule 把它直接复用过来，意味着扩展时不需要再多搭一套身份和路由系统。

下一篇这个系列会展开 RBAC——基于 IAS 用户组和属性，去控制哪些用户能调哪些 Scenario。这是企业 Agent 真正要落地时绕不过的话题。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-sap/building-custom-joule-capabilities-getting-started/ba-p/14390050
