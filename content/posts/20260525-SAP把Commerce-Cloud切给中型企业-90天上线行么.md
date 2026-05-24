```markdown
---
title: "SAP把Commerce Cloud切给中型企业，90天上线行么"
date: 2026-05-25T10:00:00+08:00
draft: false
tags: ["SAP", "Commerce Cloud", "ERP Edition", "出海B2B", "S/4HANA"]
description: "Sapphire 2026上SAP端出了Commerce Cloud ERP Edition：没有独立前端，跑在S/4里，90天上线。这刀切到了出海中型B2B最痛的地方。"
source_url: "https://www.spadoom.com/en/blog/sap-commerce-cloud-erp-edition-for-sme"
---

90天。这是SAP给新版 `Commerce Cloud, ERP Edition` 定的实施目标。对比经典版的6到12个月，这刀直接砍掉了2/3的项目周期。前端构建成本？**归零**。

5月12日的Sapphire Orlando，SAP把这件事放到了开场keynote里。媒体把焦点都给了200个AI Agent和Autonomous Enterprise的宏大叙事，但真正动到中型B2B企业架构的，是这个不太起眼的子产品。

![SAP Commerce Cloud ERP Edition](https://www.spadoom.com/hubfs/sap-commerce-cloud-erp-edition-for-sme.jpg)

## 它把Commerce从前台变成了后台

老的Commerce Cloud是一个独立平台。客户登进去，是一套自己的console；前端是Composable Storefront（前身Spartacus）；要跟ERP对接，得自己拉中间件、写集成层、搞身份联邦。这套东西跑全了，企业B2C零售商爽得不行——百万SKU、多店铺、多品牌、复杂促销，它都扛得住。

但对一个年营业额5000万欧元的瑞士工业品分销商来说，这套东西太重了。预算扛不住，团队也撑不起。

ERP Edition干了一件事：**把Commerce从一个独立产品，降级成了S/4HANA里的一组功能模块**。用户不再登录单独的电商后台。打开S/4，订单录入、商品目录、合同查询、发票、服务工单——这些原本要在Commerce里完成的动作，全变成了ERP里的原生界面。

主数据走 `SAP Business Data Cloud`（产品、价格、订单同步），身份和同意走 `SAP Customer Data Cloud`，`Joule` 预装在每个新界面里。客户不再需要自己去焊接Commerce和ERP，SAP直接在产品层把这条链路设计死了。

## 这件事对我们意味着什么

我接触过的客户里，做出海B2B的中国制造企业有一个长期的纠结。海外分销网络要电商化，选SAP经典Commerce Cloud吧——预算和实施周期都顶到了天花板；选Shopify Plus或者本地三方平台吧——又跟总部的S/4HANA对不上，订单、价格、库存又得做一层中间件。两边都不舒服。

ERP Edition把这个长期僵局给解开了。中国出海企业如果已经在用或正打算上S/4HANA Public Cloud，那么海外B2B电商这件事，现在多了一个原生路径。SKU不上百万、不需要多店铺多品牌的复杂玩法，能接受标准化流程换速度——这套东西就是给你设计的。

反过来说，做的是海外DTC、多品牌矩阵、上百万SKU、深度商品运营——经典版还是经典版，**ERP Edition不是来替代它的，是来填它身下那块空白市场**。

## 但有几个钉子还没拔

Spadoom作为首批试点伙伴，在原文里点出了两个真实的开放问题。我把它原文翻译过来——这是DSAG（德语区SAP用户组）CX工作组里正在讨论的话题，不是营销话术。

> "如果你已经在用CDC或BDC的非ERP Edition部署，迁移路径是什么？SAP表达了正确的意图，但公开的迁移手册还没出来。"

第二个钉子是**Private Cloud版本的交付时间**。Public Cloud在Sapphire Orlando上发了，Private Cloud承诺2026年底。对很多需要数据驻留控制的客户来说——尤其是欧洲DACH区域和中国出海企业有数据合规要求的——Private Cloud才是真正能落地的形态。路线图方向是对的，但2026年底能不能准时交付，得等下半年才有答案。

## 试点项目里学到的三件事

Spadoom在瑞士做了第一个工业制造业试点。原文里他们没点客户名字，但抽出来的几条经验，对未来要做ERP Edition项目的甲方IT和SI都有用：

- **配置纪律比客制化代码更重要**——没有前端要写、没有集成层要搭，项目成败完全看团队对范围的克制。90天目标是真的，但代价是客户得接受标准流程，放弃过去客制的那些"特色"。
- **Joule需要配置，不是装上就用**——AI能力默认包含，但要让它在你的具体行业里给出有用回答，提示词和grounding得花真金白银的时间调。试点团队在这件事上投入很多，他们说值得。
- **ERP团队会变成电商团队**——因为电商界面在S/4里，上线之后维护它的就是ERP的那批power user，不是另起炉灶的"数字化团队"。这件事在中型企业里反而更健康，但要在项目初期就跟客户组织对齐好。

## 一个具体的行动建议

如果你是中国出海B2B企业的CIO或CX架构师，未来六个月有两件事可以做：

一是**盘清楚你目前的SAP合同**。如果S/4HANA Public Cloud已经在用、或者下个财年的转型计划里，那么ERP Edition的GA节奏（Public Cloud已经发了，Private Cloud年底）就直接进入你的roadmap讨论。不要等2027年再来问"这个能不能上"。

二是**让你的SI伙伴去拿首批试点案例**。这个产品现在是"早期阶段"——这意味着SI的delivery playbook还在成型，前几个项目的甲方有更多议价空间，也能拿到SAP更多direct support。等明年这个产品进入主流交付，标准化的同时议价权也会被收回去。

SAP把Commerce Cloud切成两半这件事，本质是承认了一个十年没解决的问题——经典版对中型企业一直太贵太重。承认问题、做出回应，这是好事。但产品早期总有钉子没拔，路线图上的承诺也得看年底能不能兑现。

现在能做的，是**在SAP和SI都还在学这个产品的窗口期，把自己排到队伍前面**。

---

参考来源：https://www.spadoom.com/en/blog/sap-commerce-cloud-erp-edition-for-sme
```
