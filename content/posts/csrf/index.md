+++
title = "PortSwigger CSRF Labs 通关笔记"
date = 2026-11-19T23:11:00+08:00
draft = false
tags = ["CSRF", "PortSwigger", "Web安全"]
categories = ["渗透测试"]
summary = "PortSwigger Web Security Academy CSRF 实验通关记录：无防护、token 校验绕过、SameSite、Referer 校验与 Cookie 注入。"
ShowToc = true
TocOpen = false
+++

> 环境：Burp Suite Community Edition · 靶场：PortSwigger Web Security Academy ｜ 进度：10 题完成 9 题（第 8 题需 Burp Pro）

## 写在前面

这是「PortSwigger 通关」系列的 CSRF 篇。CSRF 的核心不是“伪造请求”本身，而是浏览器会自动带上受害者的 Cookie，让攻击者能够借用受害者的登录态完成操作。

笔记按实验顺序整理：先看无防护的基础利用，再逐步分析 token 校验、SameSite 和 Referer 校验的绕过方式。

## 实验清单

| # | 实验 | 状态 |
|---|------|------|
| 1 | 无防护的 CSRF | ✅ |
| 2 | token 校验依赖请求方法 | ✅ |
| 3 | token 校验依赖 token 是否存在 | ✅ |
| 4 | token 未绑定用户会话 | ✅ |
| 5 | token 与 Cookie 重复 | ✅ |
| 6 | SameSite Lax + 方法覆盖 | ✅ |
| 7 | SameSite Strict + 客户端跳转 | ✅ |
| 8 | SameSite Strict + 兄弟域 | ⏸ 需 Pro |
| 9 | Referer 校验依赖请求头存在 | ✅ |
| 10 | 错误的 Referer 校验 | ✅ |

---

## 1. CSRF vulnerability with no defenses

![image-20261119173421082](01.png)

最基础的 CSRF。修改邮箱的请求只包含新的邮箱地址和用于证明身份的 Cookie，没有任何 CSRF token。

![image-20261119181013153](02.png)

Cookie 由浏览器自动携带，所以攻击者只需要在恶意页面里放一个自动提交的表单，就能借用受害者的登录态修改邮箱。

![image-20261119181740503](03.png)

## 2. CSRF where token validation depends on request method

![image-20261119181857172](04.png)

题目说明只对 POST 做了 token 校验，其他请求方法没有防护。

![image-20261119182008048](05.png)

用 Burp 拦截修改邮箱的请求，可以看到表单里除了 `email` 还有 `csrf`；去掉 token 后请求失败。

![image-20261119182126181](06.png)

但把请求方法改成 GET 后，修改邮箱却能成功。

![image-20261119182209790](07.png)

因此可以构造一个自动提交的 GET 表单来完成利用。

![image-20261119182353581](08.png)

## 3. CSRF where token validation depends on token being present

![image-20261119182439841](09.png)

这一题的问题是：服务端只在 token 存在时校验它，token 缺失时反而直接放行。

拦截请求可以看到正常的 `email` 和 `csrf`，但删除 `csrf` 参数后请求依然成功。

![image-20261119182735157](10.png)

所以构造一个只包含 `email` 的表单即可。

![image-20261119182917468](11.png)

## 4. CSRF where token is not tied to user session

![image-20261119182952255](12.png)

这一题的 session 和 CSRF token 没有绑定：token 只验证“是否有效”，不验证“属于哪个用户”。

从题干可以确认，用于识别身份的 session 和用于防 CSRF 的 token 是分离的。

![image-20261119183559526](13.png)

![image-20261119183622318](14.png)

进入题目后可以看到提交内容里带 `csrf`；删除 token 或填入错误值都无法通过，但用 `wiener` 登录后拿到的 token，也可以用来修改 `carlos` 的邮箱。

![image-20261119184453631](15.png)

![image-20261119184540315](16.png)

注意每次提交表单都会更换新的 token。如果在 Burp 里拦截拿到了 token，需要丢弃这个请求，避免 token 失效；然后取一个未使用的 token 放进恶意表单。

![image-20261119184943648](17.png)

## 5. CSRF where token is duplicated in cookie

![image-20261119185132358](18.png)

拦截提交请求可以发现，CSRF token 同时存在于 Cookie 中。

![image-20261119185254358](19.png)

服务端通过比较 Cookie 里的 token 和请求体里的 token 来判断，因此只要把两者改成同一个已知值就能通过。

![image-20261119185843902](20.png)

在恶意页面里先用 `Set-Cookie` 修改受害者的 Cookie，再提交表单即可。

![image-20261119192440594](21.png)

## 6. SameSite Lax bypass via method override

![image-20261119210151042](22.png)

这一题需要绕过 SameSite 的限制。拦截修改邮箱的请求后可以看到，表单只提交了 `email`。

![image-20261119210957764](23.png)

先写一个简单的恶意页面测试，发现跨站请求没有携带 Cookie。

![image-20261119211654411](24.png)

网站没有显式设置 SameSite，默认是 `Lax`：GET 请求会携带 Cookie，但 POST 不会。尝试用 GET 提交修改邮箱，被服务端拒绝。

![image-20261119212538736](25.png)

在参数里加上 `_method=POST` 后，服务端把请求当作 POST 处理，提交成功。

![image-20261119212704339](26.png)

在恶意表单里加入 `_method` 参数即可完成利用。

![image-20261119213115523](27.png)

## 7. SameSite Strict bypass via client-side redirect

![image-20261119220705996](28.png)

这一题的 SameSite 是 `Strict`，任何跨站请求都不会携带 Cookie。

![image-20261119221226662](29.png)

拦截请求并测试后发现，这一题接受 GET 请求。

![image-20261119221613609](30.png)

因此只要找到页面上的跳转方式，能够跳到：

```text
/my-account/change-email?email=1%402&submit=1
```

就可以把邮箱改成 `1@2`。对任意博客发表评论后，评论成功的提示页会在几秒后跳回博客；查看页面源码可以看到跳转函数。

![image-20261119221937961](31.png)

![image-20261119221948426](32.png)

其中 `postId` 是可控的。因此在恶意页面里用 `document.location` 先跳到成功提示页，再跳到修改邮箱的地址；这两步是同源跳转，会携带 Cookie，从而完成修改。

![image-20261119223537780](33.png)

## 8. SameSite Strict bypass via sibling domain

需要 Burp Pro，跳过。

## 9. CSRF where Referer validation depends on header being present

![image-20261119223720993](34.png)

用最基础的方式测试，请求因为带了 Referer 而被拒绝。

![image-20261119224304777](35.png)

在 Repeater 中多次尝试后发现，删除 Referer 请求头后请求可以通过。

![image-20261119225011188](36.png)

在恶意页面中加入：

```html
<meta name="referrer" content="no-referrer" />
```

这样表单提交时就不会带 Referer。

![image-20261119225312941](37.png)

## 10. CSRF with broken Referer validation

![image-20261119225913045](38.png)

这一题同样校验 Referer，但删除 Referer 不能通过。

![image-20261119225837339](39.png)

多次尝试后发现，只要 Referer 中包含靶场域名就能通过：

```text
0a97003004dc42f2801e2155009a0016.web-security-academy.net
```

![image-20261119230226707](40.png)

因此构造恶意网站链接时，让 URL 中包含这个域名，同时加入：

```html
<meta name="referrer" content="unsafe-URL" />
```

保证 Referer 发送完整 URL，从而绕过校验。

![image-20261119230905071](41.png)
