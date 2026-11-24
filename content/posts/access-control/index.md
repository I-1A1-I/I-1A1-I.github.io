+++
title = "PortSwigger Access Control Labs 通关笔记"
date = 2026-11-25T00:10:00+08:00
draft = false
tags = ["PortSwigger", "Access Control", "Web安全"]
categories = ["渗透测试"]
summary = "PortSwigger 访问控制 13 个 Labs 通关记录：robots.txt 泄露后台、隐藏 admin URL、role 参数/资料改 roleid、IDOR、302 泄漏、密码泄露、X-Original-URL、方法绕过、多步流程与 Referer 绕过。"
ShowToc = true
TocOpen = false
+++

> 环境：Burp Suite Community Edition · 靶场：PortSwigger Web Security Academy ｜ 进度：13/13

## 写在前面

这篇记录 PortSwigger **Access control** 主题的 13 个 Lab。核心都是同一个问题：**前端隐藏了功能，后端有没有真正校验权限？**

按实际做题顺序记录：robots.txt 泄露后台 → 隐藏 admin URL → 改角色参数/资料 → 水平越权与 IDOR → 302 泄漏、密码泄露 → URL/方法/多步/Referer 绕过。

## 实验清单

| # | 实验 | 状态 |
|---|------|------|
| 1 | Unprotected admin functionality | ✅ |
| 2 | Unprotected admin functionality with unpredictable URL | ✅ |
| 3 | User role controlled by request parameter | ✅ |
| 4 | User role can be modified in user profile | ✅ |
| 5 | User ID controlled by request parameter | ✅ |
| 6 | User ID controlled by request parameter, with unpredictable user IDs | ✅ |
| 7 | User ID controlled by request parameter with data leakage in redirect | ✅ |
| 8 | User ID controlled by request parameter with password disclosure | ✅ |
| 9 | Insecure direct object references | ✅ |
| 10 | URL-based access control can be circumvented | ✅ |
| 11 | Method-based access control can be circumvented | ✅ |
| 12 | Multi-step process with no access control on one step | ✅ |
| 13 | Referer-based access control | ✅ |

---

## 1. Unprotected admin functionality

![01.png](01.png)

题目要求删除指定用户。进入页面后没有发现管理员面板，查看 `robots.txt`，找到管理员面板的入口：

![02.png](02.png)

进入该页面，点击 delete 删除对应用户：

![03.png](03.png)

## 2. Unprotected admin functionality with unpredictable URL

![04.png](04.png)

进入页面查看源码，发现管理员面板的入口，进入并删除对应账号：

![05.png](05.png)

![06.png](06.png)

## 3. User role controlled by request parameter

![07.png](07.png)

登录后访问 `/admin`，发现被拦截：

![08.png](08.png)

使用 Burp 拦截请求，发现提交的参数包含 `Admin=false`：

![09.png](09.png)

将 `false` 改为 `true` 后放包，成功进入管理员面板：

![10.png](10.png)

## 4. User role can be modified in user profile

![11.png](11.png)

根据题目提示，需要将账号的 `roleid` 改为 `2`。登录后尝试拦截修改密码的请求：

![12.png](12.png)

在请求中加入 `"roleid":2` 后发送，在返回中可以看到 `roleid` 已经变成了 `2`：

![13.png](13.png)

在账号页面出现管理员面板入口：

![14.png](14.png)

## 5. User ID controlled by request parameter

![15.png](15.png)

登录后看到账号页面展示了账号的 API key：

![16.png](16.png)

将提交的参数 `id` 改为 `carlos`，可以看到页面变成了 carlos 的 id 和 API key：

![17.png](17.png)

将 API key 提交至 Submit solution：

![18.png](18.png)

## 6. User ID controlled by request parameter, with unpredictable user IDs

![19.png](19.png)

登录后进入账号页面，发现与上一题不同，这题的 `id` 是一个编号，而不是用户名：

![20.png](20.png)

进入主页里的文章，发现左上角显示了作者：

![21.png](21.png)

找到一篇 carlos 的文章并点击作者名，在跳转的页面中可以看到作者的 userId：

![22.png](22.png)

将其替换账号页中的 `id`，即可进入 carlos 的账号页面：

![23.png](23.png)

## 7. User ID controlled by request parameter with data leakage in redirect

![24.png](24.png)

登录后尝试将请求中的 `id` 更改为 carlos，发现被跳转到了登录页面。但查看 Burp 的网络记录可以发现，在 302 跳转包中依然包含了完整的响应内容：

![25.png](25.png)

其中包含了 carlos 用户的 API key：

![26.png](26.png)

## 8. User ID controlled by request parameter with password disclosure

![27.png](27.png)

登录后看到修改邮箱和密码的页面：

![28.png](28.png)

将修改密码栏的类型去掉之后发现，其中预填的内容为用户真实密码：

![29.png](29.png)

查看 Burp 的流量记录，发现密码在传输中出现：

![30.png](30.png)

尝试将请求中的 `id` 改为 carlos，找到 carlos 的密码：

![31.png](31.png)

尝试改为 `admin`，发现返回不同，说明没有该账号：

![32.png](32.png)

尝试改为 `administrator`，得到 administrator 的密码：

![33.png](33.png)

## 9. Insecure direct object references

![34.png](34.png)

这一题中有一个聊天室，在聊天室中点击 view transcript 后，会以 txt 的形式下载聊天记录：

![35.png](35.png)

注意到文件名从 2 开始递增：

![36.png](36.png)

在 Burp 的流量记录中找到下载路径，将文件名改为 `1.txt`，得到包含 carlos 密码的聊天记录：

![37.png](37.png)

## 10. URL-based access control can be circumvented

![38.png](38.png)

尝试访问 `/admin`，显示访问被拒：

![39.png](39.png)

访问主页，在 Burp 中于请求头添加 `X-Original-URL: /admin`：

![40.png](40.png)

找到删除 carlos 的链接，将 `X-Original-URL` 改为 `/admin/delete`，并添加参数 `username=carlos`，提交即可删除：

![41.png](41.png)

## 11. Method-based access control can be circumvented

![42.png](42.png)

登录管理员账号后，在管理员面板找到给用户提权、降权的面板：

![43.png](43.png)

拦截提权请求：

![44.png](44.png)

退出登录后，登录 wiener，用新的 Cookie 替代请求中的 Cookie，发送显示无授权：

![45.png](45.png)

将请求方式改为 `GET` 发送，发现修改成功：

![46.png](46.png)

## 12. Multi-step process with no access control on one step

![47.png](47.png)

与上一关类似，登录管理员后拦截提权请求。但这关使用了二次确认，要拦截确认时的请求：

![48.png](48.png)

更改 Cookie 后发送即可：

![49.png](49.png)

## 13. Referer-based access control

![50.png](50.png)

与前两题类似，但第 11 题是 `GET` 请求方式未被正确处理，第 12 题是权限只在第一步判断、后续步骤不再校验；这题则是判断 `Referer`，做法与第 12 题一样，不再赘述。
