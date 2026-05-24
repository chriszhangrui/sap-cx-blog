---
title: "BDC 的数据产品，怎么变成 Claude 可调用的工具"
date: 2026-05-24T22:10:00+08:00
draft: false
tags: ["SAP CX", "BDC", "技术深度", "MCP", "Datasphere", "Claude"]
description: "Sapphire 2026 SAP 和 Anthropic 把 MCP 推到 BDC 中央。但社区开发者半年前就跑通了——把 Datasphere 当作 Claude 可调用的工具集。"
source_url: "https://community.sap.com/t5/technology-blog-posts-by-members/connect-claude-ai-to-sap-datasphere-using-mcp-an-implementation-guide/ba-p/14387720"
---

SAP Sapphire 2026 上 SAP 和 Anthropic 宣布了一项战略合作，把 Claude 和 MCP（Model Context Protocol）摆到 SAP AI 战略的中央。这条消息在公告稿里只是一行字，但真正值得看的，是社区里一位叫 Saravanan Chinnaswamy 的开发者半年前就已经把这件事跑通——他用一个 200 行不到的 Python 服务，把 Claude 直接接到了 SAP Datasphere 的 REST 和 OData 接口上，做了一个 MCP server。

这件事的技术含义，不是"AI 接了个数据库"。是 BDC（SAP Business Data Cloud）的数据产品，第一次以一种 Agent 友好的方式被外部模型直接消费。

## 从找视图开始的问题

原作者写这个东西的起因很朴素：在 Datasphere 里翻 Space、找 View、找模型这件事太烦。一个中型租户里，Space 几十个，每个 Space 下面 view、analytic model、table 加起来上百个，光定位"我要的那张表叫什么"就能花掉半小时。他想要的体验是：直接问 AI 助手"SALES_SPACE 里有哪些表"或"给我看 MONTHLY_ACTUALS 前 10 行"，立刻拿到答案。

这个需求做成传统集成会怎么样？写一个 BI 网关，前面套个 NLQ 层，把自然语言翻成 SQL，再去查 Datasphere——SAP Analytics Cloud 的 Just Ask 大致是这条路线。问题是它跟一个具体的 UI 锁死了，而且语义层是 SAP 自己写死的。

MCP 路线完全相反。它不预设 UI，也不预设语义层。它做的是一件更基础的事：把 Datasphere 的 API 抽象成一组可被任意模型自动发现、自动调用的"工具"，剩下的对话编排交给 Claude。

![MCP × Datasphere 架构示意](https://community.sap.com/t5/image/serverpage/image-id/405270i52D083A1065F9806/image-size/large)

## 三层架构里发生了什么

整个集成只有三层：用户端的 Claude（Desktop 客户端，Web 版还不支持 MCP）、本地跑的 Python MCP server、远端的 SAP Datasphere REST/OData 接口。

真正承重的是中间这一层。它做四件事：

- 用 FastMCP 框架启动一个 MCP 协议服务，对外声明自己有哪些 tool；
- 每个 tool 是一个 Python 函数，函数前用 `@mcp.tool()` 装饰；
- 函数的 docstring 是 Claude 读取的"工具说明书"——Claude 通过它判断这个工具能干什么、什么时候该调用；
- 函数体内部就是普通的 HTTP 调用，把 Datasphere 当成普通 REST 后端。

最小代码长这样：

```python
from fastmcp import FastMCP
import requests

mcp = FastMCP('datasphere-mcp')

@mcp.tool()
def list_spaces(auth_token: str) -> dict:
    """List all available spaces in the Datasphere tenant."""
    url = f'{DATASPHERE_BASE_URL}/api/v1/dwc/catalog/spaces'
    headers = {'Authorization': f'Bearer {auth_token}'}
    response = requests.get(url, headers=headers)
    return response.json()
```

就这么多。FastMCP 把 MCP 协议的 stdio 通信、JSON-RPC 握手、工具发现这些东西全包掉了。开发者只需要写"这个工具叫什么、它读哪个 API、返回什么字段"。

## 第一版暴露了 6 个工具

原作者在第一版里实现了 6 个 tool，分两组：

- **list_spaces** — 列出租户里所有的 Datasphere Space；
- **list_tables** — 列出某个 Space 下的表、视图和 analytical model；
- **get_table_schema** — 拿到某个对象的列名、数据类型、是否可空；
- **get_space_objects** — 调试用，直接 dump 原始 API 响应；
- **run_odata_query** — 用 OData 协议查询某个 entity，支持 $filter 和 $select；
- **preview_table** — 取某张表/视图的前几行做样例。

前 4 个是 metadata 类，后 2 个是 query 类。这个分组不是随手分的——它对应了 BDC 数据产品消费的两种模式：先发现，再查询。Claude 接到一个自然语言问题，先用 metadata 工具弄清楚"我要的表叫什么、有哪些字段"，再用 query 工具拿数据。中间所有的 API 拼接、字段映射、过滤条件转换，都是 Claude 自动完成的。

## 客户端配置长这样

Claude Desktop 端注册一个 MCP server，配置文件 claude_desktop_config.json 里加一段：

```json
{
  "mcpServers": {
    "datasphere": {
      "command": "C:\\...\\Python311\\python.exe",
      "args": ["C:\\...\\server.py"],
      "env": {
        "DATASPHERE_BASE_URL": "https://....hcs.cloud.sap",
        "DATASPHERE_COOKIES": ""
      }
    }
  }
}
```

重启 Claude Desktop 后，对话框里多出一个工具图标，里面会列出 datasphere 这个 server 暴露的所有 tool。然后就可以问"我的租户里有哪些 Space"——Claude 自己挑工具、自己调用、自己把结果翻译成自然语言。

## 它现在还做不到什么

原作者很坦白地列了几条限制，对应到 BDC 落地，每一条都是踩坑预警：

- **只读**。不能 write-back、不能部署模型、不能改 Space 配置。BDC 数据产品的"治理可控"从这一层就保住了——MCP 工具的能力上限是显式声明的，模型再聪明也只能调你给它的工具。
- **只能访问打了 Allow consumption 标记的对象**。Datasphere 里的表和视图必须在 Space 设置里勾上"允许被消费"，analytical model 必须先 deploy。这是 Datasphere 原本的访问控制机制，MCP 没绕过它，也绕不过。
- **认证用浏览器 cookie**。开发者端从浏览器开发者工具里把登录后的 cookie 拷下来贴到 .env 里——这个方案 demo 可以，正式环境绝对不行。Cookie 几个小时就过期，团队多人用更是灾难。原作者下一步要做的就是切到 OAuth 2.0 client credentials。

> 这条限制其实最有意思：BDC 的 MCP 化能不能上规模，瓶颈不在 AI，而在认证。OAuth client credentials + 服务账号管理 + token 短时效轮转，这些是企业 IT 的基本功，但放在 Agent 场景里要重新审视——一个 Agent 跑一段长任务，token 中间过期了怎么办？这是 SAP×Anthropic 那条战略合作真正要解决的问题。

## 这件事跟 SAP 自己的方向是什么关系

SAP 自己在 BDC 上有两条 AI 路径。一条是 Joule + Just Ask，把对话式分析做进 SAC 和 Datasphere 的 UI 里，用户在 SAP 自家产品里问问题。另一条是 Joule Studio + Pro-Code Agent，让客户在 BTP 上写自定义 Agent，调用 Joule 的工具集。

Sapphire 2026 上跟 Anthropic 这次合作，加了一条新的：让外部 Agent（不是 SAP 自家的 Joule）也能直接消费 SAP 数据。社区开发者半年前的实验，本质上就是这条路径的最早形态。

这三条路径在客户端不冲突——Joule 管 SAP UI 内的助手体验，Joule Studio 管自定义 Agent，MCP 管让外部 LLM 把 SAP 当工具源。但在数据产品层是一致的：都靠 BDC 的 Data Product Catalog、metadata、Allow consumption flag 这套元数据治理。换句话说，BDC 押注的不是某个 Agent 框架赢家，而是"我把数据产品做扎实，谁来都能接"。

## 什么样的客户值得现在动手

结合出海企业、跨境电商、在华外企的 IT 现实，可以分三档：

- **已经在用 Datasphere 或 BDC 的客户**，如果数据团队抱怨"找数据比用数据还累"，做一个 MCP server 是低成本高价值的内部工具——拿一周时间封装 5-10 个常用 tool，团队效率立刻看得见。
- **跨国制造、跨境零售总部对接 SAP 的 IT 部门**，可以把这个模式放进 Agent 化路线图——未来总部或区域分析 Agent 调取经营数据，路径就是 MCP 而不是再做一遍 ODBC/JDBC 网关。
- **SAP 生态 SI 合作伙伴**，这是一个非常清晰的 offering 切入点：BDC 实施 + MCP 工具集封装 + 客户自有 Agent 接入，三段式打包卖。

## 什么时候先别碰

三种情况要按住：第一，连 Datasphere 都还没上的客户，先把数据产品做出来再谈 MCP，不要为了 AI 而 AI；第二，Web 版 Claude 不支持 MCP，要桌面版才行——内部安全策略不允许装桌面端的客户，这条路就先封住；第三，认证还停留在 cookie 阶段的实验代码，绝对不要直接搬到生产环境，等 OAuth client credentials 版本出来再说。

BDC 把数据产品做成 Agent 可消费的工具，这个方向已经从战略层进到代码层。Sapphire 2026 那条公告只是把它正式化了。技术决策者真正要回答的问题，不是"该不该跟"，而是"先在哪个 Space 跑通第一组工具"。

参考来源：https://community.sap.com/t5/technology-blog-posts-by-members/connect-claude-ai-to-sap-datasphere-using-mcp-an-implementation-guide/ba-p/14387720
