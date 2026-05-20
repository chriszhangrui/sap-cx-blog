---
title: "Composable Storefront 这次为什么换地基"
date: 2026-05-20T22:25:00+08:00
draft: false
tags: ["SAP CX", "Commerce", "Composable Storefront", "OAuth2", "技术深度"]
description: "做一个 OTP 登录页这种小事，把 SAP Commerce Cloud 新版前端的认证地基全翻了一遍：授权码流、CSRF 表单回传、CMS 组件挂载——三层都得对得上。"
source_url: "https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/building-a-custom-otp-based-authentication-page-for-sap-commerce-composable/ba-p/14342770"
---

起点是个看上去挺简单的需求：在 Composable Storefront（SAP Commerce 的可组合店铺前端）上加一个 OTP 登录入口——用户输完邮箱密码，再用一次性验证码二次校验，登录成功后回首页。

如果这是 Accelerator 时代的 storefront，前端拦个表单、后端接一个 controller 就完了。但在 JDK21 起的 Composable Storefront 上做这件事，等于把整条认证链路重走一遍：前端组件经过 SmartEdit 做 CMS 映射、OAuth2 授权服务器走授权码流、token 通过表单 POST 回调而不是 JSON——三个原本相对独立的东西必须严丝合缝地对上。

这才是 Composable Storefront 真正的成本所在：它把 storefront 从「一个被 Commerce 后端塞数据的页面」变成了「OAuth2 协议下的标准 Client」，自由度大了，但代价是每个看起来很小的需求都得在协议层面想清楚。

## 一、地基已经换了：authorizationCodeFlowByDefault 不是开关，是分水岭

先把背景说清楚。Composable Storefront 在升级到 JDK21 兼容版之后，开启了一个看起来不起眼的特性开关：

```
authorizationCodeFlowByDefault: true
```

打开它的意思是：从此之后，登录不再走老的 implicit / password grant，而是走 OAuth2 标准的 Authorization Code Flow（授权码模式）。前端把用户引导到授权服务器的 `/oauth/authorize`，授权服务器渲染（其实是 Spartacus 提供的）自定义登录页，用户填表单 POST 到 `/login`，授权服务器签发 code，前端再拿 code 换 token。

这件事在协议上一点都不新鲜，但在 SAP Commerce 这边是结构性的改动：

- 登录页不再是 Storefront 内部的一个路由，而是 OAuth2 授权服务器的一部分；
- 登录提交不是前端 fetch 去打 REST API，而是浏览器原生的 form POST，带 CSRF token、Set-Cookie 这一整套；
- Storefront 的角色变成了 OAuth2 Client，token 拿到后才进入「业务态」，之前都是协议态。

Storefront 这边的配置其实就是告诉框架：我的授权服务器在哪、CSRF 端点和登录端点叫什么。

```typescript
provideConfig(<AuthConfig>{
  authentication: {
    baseUrl: 'https://localhost:9002/authorizationserver',
    loginUrl: '/oauth/authorize',
    tokenEndpoint: '/oauth/token',
    customLoginPage: {
      csrfEndpoint: '/csrf',
      loginFormEndpoint: '/login'
    }
  }
})
```

为什么这件事重要？因为很多人会按老经验把 OTP 登录写成「前端一个组件，后端一个接口」，结果发现表单怎么提都不对，CSRF 校验过不了，session cookie 莫名其妙丢了——这都是 implicit flow 的思维残留。在新地基上，登录这件事的真正主体是授权服务器，Storefront 只是个调用方。

## 二、双步 OTP 怎么落到协议上：tokenId 是 username，tokenCode 是 password

OTP 登录的实现思路本身不复杂，分两步：

- Step 1（请求验证码）：前端把邮箱密码先校验一遍合法性，调 `createVerificationToken`，授权服务器创建一个 tokenId、并通过邮件等带外通道把 tokenCode 发给用户；
- Step 2（提交验证码）：前端跳到验证页，用户输入 tokenCode，前端把 tokenId + tokenCode 表单 POST 给授权服务器的 `/login`。

注意第二步：授权服务器并不知道什么是 OTP，它只认它能消费的协议——一个带 username、password、CSRF 的 form POST。所以 Spartacus 的实现里有一个非常关键的小动作：

- 把 tokenId 作为 username 字段提交
- 把 tokenCode 作为 password 字段提交

这是把 OTP「贴」进 OAuth2 协议的方式。授权服务器侧识别 tokenId 是一次性凭证，按「token 化的密码登录」处理，然后照常走授权码流的后续步骤。整条链路在协议上没有偏离，OTP 完全靠 Commerce 后端自己识别。

Step 1 这一步前端代码长这样：

```typescript
this.verificationTokenFacade
  .createVerificationToken({
    loginId, password,
    purpose: 'LOGIN',
  })
  .subscribe({
    next: (result) => {
      this.routingService.go(
        { cxRoute: 'verifyToken' },
        { state: { loginId, password,
                   tokenId: result?.tokenId,
                   expiresIn: result?.expiresIn } }
      );
    }
  });
```

Step 2 提交时模板里的字段命名要严格对齐：

```html
<input formControlName="tokenCode"
       name="password" required />
<input type="hidden"
       formControlName="tokenId"
       name="username" />
<input *ngIf="csrf" type="hidden"
       formControlName="csrf"
       [name]="csrf.parameterName" />
```

![OTP 登录时序图](https://community.sap.com/t5/image/serverpage/image-id/380377i4294939F8C8E1C5F/image-size/large?v=v2&px=999)

*OTP 登录的完整时序：前端组件 → VerificationTokenFacade → 授权服务器 → Commerce 后端 → 邮件通道（图源 SAP Community）*

## 三、CMS 组件映射：不是写完就生效，得让 SmartEdit 找到你

写好三个 Angular 组件还不够，因为 Composable Storefront 不是把组件硬编码到路由里的。它的渲染逻辑是：CMS 后台配置好哪个 ContentSlot 用哪个 CMS Component（比如 ReturningCustomerOTPLoginComponent），前端启动时根据 CMS 配置去查找已注册的 Angular 组件并挂载。

这层映射是在前端模块里通过 `provideConfig` 注册的：

```typescript
provideConfig(<CmsConfig>{
  cmsComponents: {
    ReturningCustomerLoginComponent: {
      component: CustomLoginComponent,
      guards: [NotAuthGuard, CustomLoginGuard],
    },
    ReturningCustomerOTPLoginComponent: {
      component: CustomOtpLoginComponent,
      guards: [NotAuthGuard, CustomLoginGuard],
    },
    VerifyOTPTokenComponent: {
      component: CustomOtpVerifyComponent,
      guards: [NotAuthGuard, CustomLoginGuard],
    }
  }
})
```

两个细节值得拎出来：

- `guards: [NotAuthGuard, CustomLoginGuard]` —— NotAuthGuard 阻止已登录用户进登录页；CustomLoginGuard 确保组件只在 OAuth2 授权流的「自定义登录页上下文」里渲染，不会被错误地挂到普通页面。
- CMS 侧需要一段 impex（SAP Commerce 的批量数据导入脚本），把 CMSFlexComponent、ContentPage、ContentSlot 都建出来。也就是说，前端组件写好之后，Commerce 后端的内容目录也要同步更新——这是 Composable 路线下「前端独立部署」的另一面：你以为前端独立了，但内容契约还在 Commerce 那边。

![CMS 组件结构](https://community.sap.com/t5/image/serverpage/image-id/380380i14EEF4B5B88711E5/image-size/large?v=v2&px=999)

*CMS 组件 → Angular 组件的注册关系，guards 控制了组件渲染的上下文（图源 SAP Community）*

## 四、这个设计放弃了什么，换来了什么

把视角从「实现一个 OTP」拉回到架构层面，可以看清这条路线在做什么取舍。

放弃的部分：

- 放弃了 Accelerator 时代「JSP + 后端 controller」的整体性。前端写一行字、后端调一段逻辑，那种顺手是没了。
- 放弃了 implicit / password grant 这种「前端一把梭」的便利。Token 流转必须走标准 OAuth2，调试链路变长。
- 放弃了 storefront 完全独立的幻想。CMS 内容仍然在 Commerce 后端，B2B 复杂业务流（报价、合同、approval）依赖的还是后端 API。前端能解耦的是 UI 渲染层，不是业务层。

换来的部分：

- 授权服务器作为协议主体，意味着 Storefront 可以替换。今天用 Spartacus 做的前端，明天用 Next.js 重写一份，OAuth2 这一层接口稳定。这是「composable」一词的真正含义——可替换的组件，而不是更花哨的前端。
- 登录这种安全敏感链路集中到授权服务器，CSRF、session、redirect 都是标准实现，不再散落在每个 storefront 项目里。安全审计的成本会下降。
- 自定义认证类型（OTP、社交登录、企业 SSO）有了统一的接入位置——都通过「token 化的 username/password」+ 自定义 CMS 组件这套模式落地，不需要每次都改协议。

## 五、什么样的项目适合走这条路，什么样的不要碰

Composable Storefront 不是给所有 Commerce Cloud 客户准备的。从这次 OTP 实现的复杂度倒推，几个判断：

- 适合：出海 D2C 品牌、跨境零售、有海外业务的制造企业，前端体验是核心竞争力，且团队有现代前端工程能力（Angular/Next.js + CI/CD + 灰度发布），愿意把前端当一个独立产品来运营。
- 适合：多前端形态的场景——同一套 Commerce 后端要支撑 Web、移动 H5、嵌入式小程序、合作伙伴门户。Composable 的协议化设计让多个前端共享同一套授权流和 API。
- 不适合：B2B 项目里那些「卖给采购员、登录一次用一年」的简单门户。把 Authorization Code Flow + 自定义 CMS 组件这套搬进去，维护成本远高于 Accelerator 直接堆功能。
- 不适合：还在用 Accelerator 跑得好好的项目，没有明确的前端体验诉求。OAuth2 改造、CMS 重建、组件库迁移，每一项都是百人日级别的投入。

一个具体的踩坑提示：如果你正在做 Accelerator → Composable 的迁移，先把 OAuth2 授权服务器配置和登录页流程跑通，再考虑业务功能迁移。地基没换好就开始装修，所有问题都会冒出来——CSRF 不匹配、redirect loop、session cookie 跨域，这些不是业务 bug，是协议没对齐。

参考来源：https://community.sap.com/t5/crm-and-cx-blog-posts-by-sap/building-a-custom-otp-based-authentication-page-for-sap-commerce-composable/ba-p/14342770
