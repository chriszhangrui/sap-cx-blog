
# Joule Studio 升 2.0：SAP 把 Vercel 拉进来给 CX 前端

Sapphire 2026 散场已经两周，社交媒体上还在反复刷"200 个 Agent""51 个 Joule Assistant"这种数字。但盯着这些数字看是会错过重点的。

**真正改了 SAP CX 项目交付方式的，不是 Agent 数量，而是 Joule Studio 升 2.0 这件事。**

先把 Joule 这个名字理顺。很多人到现在还以为 Joule 就是 SAP 应用里那个聊天框，错了。Sapphire 2026 之后，Joule 已经被拆成五条产品线加一个新入口。

Joule Base 是嵌在 SAP 应用里的自然语言助手，所有 Cloud 订阅自带，最基础那一层。Joule Agents 是端到端跑业务工作流的智能体，单独按 PUPM 计费。Joule Studio 是建 Agent 的开发环境，本次升级幅度最大。Joule for Developers 嵌在 ABAP 和 Build 里做代码生成，Joule for Consultants 给 SI 做项目交付指引。最后还冒出来一个 **Joule Work**——直接取代传统应用导航的工作台，移动端已经 GA，桌面端今年 Q4 上。

![Joule 五条产品线](http://mmbiz.qpic.cn/sz_mmbiz_png/lWqJzSMIBLXmVmoqsYK38TfXhv1Av79q6PCxr9ibyLYaNugrP0JEhlEHJkMfVxJyX2MPp2wkoJ5u711gI547qLg9n1qzbL7qIbaeX2bD7ic4k/0?from=appmsg)

回到 Studio。Joule Studio 1.x 时代它更像是个低代码玩具，在 BTP 里画几个流程节点，调用预定义的 Joule Skill。开发者用着用着会很快撞到天花板：要写复杂逻辑得切回 ABAP 或 CAP，要接外部 LLM 框架基本不可能，要自定义 UI 几乎没空间。

**Studio 2.0 的整个产品形态变了。**

它现在同时支持低代码和 Pro-Code 两条路，并且把外面主流的开源 Agent 栈直接收编进来：LangChain、Pydantic AI、LlamaIndex 全都能在 Studio 里原生调用。开发工具侧支持 VS Code 和 Cursor 直接连进去做 Agent 开发。前端框架开放了 Vercel 的 Next.js。工作流引擎接进 n8n。底层算力可以挂 NVIDIA OpenShell。Sapphire 2026 之后，Joule Studio 设计期 2026 年内全部免费，运行期才按消耗计费。

把这个清单和上一代对照一下就知道 SAP 这次让步有多大。过去 SAP 在 CX 前端这件事上，是要求大家用 SAP Commerce 自带的 Composable Storefront（基于 Angular）或者更早的 Spartacus，前端框架的选项几乎是封闭的。

**现在 Vercel 直接进了 SAP CX 的官方推荐技术栈。**

这不是把 Next.js 列在某份兼容性文档里那种水平的合作。Vercel 是被 Joule Studio 当作一等公民的前端 Runtime 集成进来的，意味着 Joule 可以直接生成 Next.js 代码，部署到 Vercel 平台上，再通过 Joule 调度去消费 Commerce、Sales Cloud 或者 Engagement Cloud 的后端能力。SAP CX 项目里前端那部分，第一次有了 Composable Storefront 之外的、被 SAP 自己背书的选择。

对中国出海企业的 SAP CX 项目来说，这件事影响是直接的。原来用 Composable Storefront 跑欧洲市场的 D2C 站，性能、SEO、本地化加载这些指标其实一直是被诟病的，团队要找前端工程师也难，会 Angular 的 D2C 工程师在欧洲市场是稀缺品种。Next.js 的人才供给完全不是一个量级的。Vercel 的边缘部署对欧洲多区域多语言站点也比 SAP 自己跑的容器方案灵活。

我看到这次更新之后第一反应是，未来 12 到 18 个月内，新启动的欧洲 D2C 项目里，会有相当一部分团队会把"我们用 Joule Studio 接 Next.js 跑 Vercel"作为前端方案的默认选项。Composable Storefront 不会消失，但它会从"唯一选项"变成"路径之一"。

**Sapphire 2026 真正值得跟踪的不是 200 这个数字，是 SAP 把 Joule Studio 的边界主动往外推了一大步。**

200 个 Agent 摆在 Autonomous Suite 里更多是营销语义，因为大部分 Agent 客户压根用不上、也不会原样部署。Studio 2.0 的开放性才是会真正改变交付动作的事——它影响的是后面两到三年里，每个 SAP CX 项目要写多少代码、用什么框架、找什么人来做、跑在哪里。这个传导链条会比"宣布了 200 个 Agent"长得多。
