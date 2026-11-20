+++
title = "PortSwigger SQL 注入 Labs 通关笔记"
date = 2026-11-05T06:30:00+08:00
draft = false
tags = ["SQL注入", "PortSwigger", "Web安全"]
categories = ["渗透测试"]
summary = "PortSwigger Web Security Academy SQL 注入 17 个实验的通关记录：UNION、盲注、带外与 WAF 绕过。"
ShowToc = true
TocOpen = false
+++
> 环境：Burp Suite Community Edition · 靶场：PortSwigger Web Security Academy ｜ 时间：2026-11 ｜ 进度：17 题完成 15 题（2 题需 Burp Pro）

## 写在前面

这是「PortSwigger 通关」系列的第一篇。SQL 注入的实验从最基础的 WHERE 子句开始，覆盖 UNION、盲注（布尔 / 报错 / 时间）、带外和 WAF 绕过。笔记按实验顺序整理，每题记录目标、关键 payload 和踩坑点，方便复习时快速定位。

## 实验清单

| # | 实验 | 状态 |
|---|------|------|
| 1 | WHERE 子句注入，检索隐藏数据 | ✅ |
| 2 | 登录绕过 | ✅ |
| 3 | 查询 Oracle 的数据库类型与版本 | ✅ |
| 4 | 查询 MySQL / SQL Server 的数据库类型与版本 | ✅ |
| 5 | 列出非 Oracle 数据库的内容 | ✅ |
| 6 | 列出 Oracle 数据库的内容 | ✅ |
| 7 | UNION 攻击：确定查询列数 | ✅ |
| 8 | UNION 攻击：寻找可回显字符串的列 | ✅ |
| 9 | UNION 攻击：从其他表取数据 | ✅ |
| 10 | UNION 攻击：单列回显多个值 | ✅ |
| 11 | 条件响应盲注 | ✅ |
| 12 | 条件错误盲注 | ✅ |
| 13 | 可见错误回显注入 | ✅ |
| 14 | 时间延迟盲注 | ✅ |
| 15 | 时间延迟盲注 + 数据提取 | ✅ |
| 16 | 带外注入（interaction / exfiltration） | ⏸ 需 Pro |
| 17 | XML 编码绕过 WAF | ✅ |

---

## 1. SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

![01.png](01.png)

题目直接给出了商品分类使用的查询语句。

![02.png](02.png)

进入靶场后可以看到多个分类，点击 Accessories 后，跳转地址为：

```
https://0ae600bc044ca55582e0ab2900370064.web-security-academy.net/filter?category=Accessories
```

按题目给出的查询语句，在 `Accessories` 后加上：

```
' or 1=1 -- 
```

拼接后实际执行的语句变成：

```
SELECT * FROM products WHERE category = 'Accessories'or 1=1 -- ' AND released = 1
```

`--` 之后的内容会被当作注释，不再参与解析，条件恒为真，于是所有商品都会被返回。

![03.png](03.png)

---

## 2. SQL injection vulnerability allowing login bypass

![04.png](04.png)

题目说明登录功能存在注入，要求以 administrator 身份登录。

![05.png](05.png)

进入靶场后，右上角比上一题多了一个 My account 入口。

![06.png](06.png)

点进去是一个简易登录框，用万能密码尝试登录：

```
administrator' or 1=1 -- 
```

![07.png](07.png)

![08.png](08.png)

登录成功，靶场判定通关。

---

## 3. SQL injection attack, querying the database type and version on Oracle

![09.png](09.png)

题目说明注入点在商品分类参数，要求使用 UNION 攻击，并提示数据库是 Oracle。

![10.png](10.png)

页面顶部给出了要回显的内容（Oracle 版本）。在用 UNION 之前，先用 ORDER BY 探测查询列数。

![11.png](11.png)

![12.png](12.png)

按第 2 列排序正常、按第 3 列报错，说明查询共 2 列。构造 UNION 后成功带出版本信息：

```
' union select null,banner from v$version --
```

![13.png](13.png)

---

## 4. SQL injection attack, querying the database type and version on MySQL and Microsoft

![14.png](14.png)

与上一题思路相同，只是目标换成了 MySQL 和 Microsoft SQL Server。

![15.png](15.png)

步骤完全一致，只需把取版本的语句换成对应数据库的语法（MySQL / SQL Server 用 `@@version`）：

```
' union select null,@@version --
```

---

## 5. SQL injection attack, listing the database contents on non-Oracle databases

![16.png](16.png)

漏洞类型与前面相同，但这次要查询其他表的数据，拿到 administrator 的密码。

![17.png](17.png)

![18.png](18.png)

为了便于查看，这里省略了页面中与注入数据无关的内容。题目排除了 Oracle，用 MySQL / PostgreSQL 的语法试探后，确认后端是 PostgreSQL。先列出所有 schema：

```
' union select null,schema from information_schema.schemata --+
```

![19.png](19.png)

`information_schema` 和 `pg_catalog` 都是数据库自带的系统 schema，业务数据在 `public` 里。继续查 `public` 下有哪些表：

```
' union select null,table_name from information_schema.tables where table_schema='public' --+
```

![20.png](20.png)

用户数据在 `users_zxwabc` 表中，接着查它的列名：

```
' union select null,column_name from information_schema.columns where table_name='users_zxwabc' --+
```

![21.png](21.png)

最后取出用户名和密码：

```
' union select username_znxksg,password_xjsenj from users_zxwabc --+
```

![22.png](22.png)

拿到 administrator 的密码后，回登录页登录即可通关。

---

## 6. SQL injection attack, listing the database contents on Oracle

![23.png](23.png)

与上一题相同，只是数据库换成了 Oracle。

![24.png](24.png)

Oracle 里 schema 就是 owner，除了系统用户，只有 `PETER` 是业务 schema。用它查表：

```
' union select null,table_name from all_tables where owner='PETER' --
```

![25.png](25.png)

再查 `USERS_XBDFHG` 的列名：

```
' union select null,column_name from all_tab_columns where table_name='USERS_XBDFHG' --
```

![26.png](26.png)

最后取出该表内容，用管理员账号登录即可：

```
' union select PASSWORD_KMJELD,USERNAME_VIBASQ from USERS_XBDFHG --
```

![27.png](27.png)

---

## 7. SQL injection UNION attack, determining the number of columns returned by the query

![28.png](28.png)

与前面一样，只是要求先确定列数。前面每道题我都先做了 ORDER BY 探测，这里不再重复。

---

## 8. SQL injection UNION attack, finding a column containing text

![29.png](29.png)

本题要求找出能回显字符串的列，并把指定字符串带出来。

![30.png](30.png)

页面顶部给出了要回显的字符串。用 ORDER BY 测出查询共 3 列，构造 3 个 NULL 的 UNION 后页面正常显示：

```
' union select null,null,null --+
```

![31.png](31.png)

依次把 3 个 NULL 换成目标字符串：

![32.png](32.png)

![33.png](33.png)

第二个位置成功回显，说明第 2 列是字符串类型。

---

## 9. SQL injection UNION attack, retrieving data from other tables

思路与上一题相同，不再赘述。

---

## 10. SQL injection UNION attack, retrieving multiple values in a single column

![34.png](34.png)

其余与上一题相同，但本题只有一个回显列。

![35.png](35.png)

测试后确认数据库是 PostgreSQL，查询共 2 列、只有第 2 列回显且为字符串。有两种做法：

**做法一**：用 `||` 把用户名和密码拼成一列输出，中间加分隔符便于区分：

```
' union select 1,username||'-'||password from users --
```

![36.png](36.png)

**做法二**：先查出用户名，再按用户名查密码。

```
' union select null,username from users --
' union select null,password from users where username='administrator' --
```

![37.png](37.png)

![38.png](38.png)

拿到用户名和密码后登录即可。

---

## 11. Blind SQL injection with conditional responses

![39.png](39.png)

题目说明注入点在 Cookie 中：查询命中时，页面会显示 Welcome back。

![40.png](40.png)

进入靶场后，任意商品页右上角都会显示 Welcome back。

![41.png](41.png)

用 Burp 抓包可以看到请求里带了：`Cookie: TrackingId=iAzNc4W7chWdqc7H;`

![42.png](42.png)

![43.png](43.png)

![44.png](44.png)

![45.png](45.png)

把请求发到 Repeater 测试后确认是 PostgreSQL，查询里有一列字符串类型的数据。页面不回显查询结果，UNION 用不上，改用布尔条件拼接：

![46.png](46.png)

![47.png](47.png)

`AND` 后的条件为真时显示 Welcome back，为假时不显示，据此可以逐位推断内容。

![48.png](48.png)

多次试探后确定 administrator 的密码是 20 位。提示说明密码只含数字和小写字母，用二分法可以加速，但这里先用最直接的办法：把请求发到 Intruder，payload 改成对 20 个下划线的 `LIKE` 匹配：

```
' and (select count((select password from users where username='administrator' and password like '____________________')))=1--
```

把 20 个下划线都标成 payload position，用 Sniper 模式，字典放 0-9 和 a-z。

![49.png](49.png)

然后开始爆破。

爆破耗时较长；靶场长时间无通信会被销毁，而密码是随机生成的，重新进入后需要重新提取。

![50.png](50.png)

按响应长度排序，再按位置拼接 payload，得到密码 `m52hcj27kwjtdu2997b3`：

![51.png](51.png)

---

## 12. Blind SQL injection with conditional errors

![52.png](52.png)

注入点同上，仍在 Cookie 里；本题的差异是会返回错误信息。

![53.png](53.png)

![54.png](54.png)

发到 Repeater 测试：条件不成立时 SQL 报错、页面返回 500；成立时正常返回 200，且数据库为 Oracle。

![55.png](55.png)

![56.png](56.png)

构造 `CASE WHEN 条件 THEN TO_CHAR(1/0) ELSE NULL END`：条件为真时触发除零错误，为假时页面正常，据此可逐位判断密码：

```
' union (select case when 条件 then to_char(1/0) else null end from dual)--
```

![57.png](57.png)

![58.png](58.png)

![59.png](59.png)

![60.png](60.png)

简单测试后确定密码为 20 位、首位是 `d`，其余步骤同上一题，不再赘述。

---

## 13. Visible error-based SQL injection

![61.png](61.png)

题目说明注入点在 Cookie。

![62.png](62.png)

测试发现本题会把数据库错误回显到页面；后台查询是 `SELECT * FROM tracking WHERE id = 'TrackingId'`，TrackingId 就是 Cookie 的值。

![63.png](63.png)

用各数据库的特征语法探测，确认是 PostgreSQL。

![64.png](64.png)

拼接语句时报错，从报错看后台对字符串长度有限制，需要更短的写法。

![65.png](65.png)

把字符串强制转成 int 触发类型错误，就能让错误信息把内容带出来。

![66.png](66.png)

![67.png](67.png)

第一条就是 administrator，密码 `z3yxrrsuqvw7l5gounvi`。

---

## 14. Blind SQL injection with time delays

![68.png](68.png)

注入点仍在 Cookie，但页面没有任何回显，只能用时间盲注；题目要求触发 10 秒延迟。

![69.png](69.png)

用 `pg_sleep(10)` 后，右下角的响应时间比平时多出约 10 秒，验证成功。

---

## 15. Blind SQL injection with time delays and information retrieval

![70.png](70.png)

本题要求用时间盲注把密码取出来。

![71.png](71.png)

靶场与上一题完全一样，直接构造时间盲注 payload：

```
' and (select case when 条件 then pg_sleep(3) else null end) is null --+
```

![72.png](72.png)

![73.png](73.png)

条件为真时响应约 3.267 秒、为假时约 0.258 秒，差异明显。结合第 11 题的按位爆破即可拿到密码，不再赘述。

---

## 16. Blind SQL injection with out-of-band interaction / data exfiltration

这两题是 DNS 带外（OAST）。靶场屏蔽了第三方 DNSLog 平台，官方要求使用 Burp Collaborator（Pro 功能），我这里是社区版，暂不演示。

---

## 17. SQL injection with filter bypass via XML encoding

![74.png](74.png)

本题要求在 WAF 拦截下完成注入。

![75.png](75.png)

按题目提示找到注入点。

![76.png](76.png)

简单尝试就被 WAF 拦了，需要绕过。

![77.png](77.png)

从 BApp Store 安装 Hackvertor，用它对 payload 编码。

![78.png](78.png)

把 payload 放进 Hackvertor，用 `dec_entities` 编码后拿去打。

![79.png](79.png)

拿到了 users 表的列名。

![80.png](80.png)

![81.png](81.png)

按列名写出查询 payload，同样编码后发送，即可拿到用户名和密码。

---

## 附：各数据库关键语法对照

| 用途 | MySQL | PostgreSQL | Oracle | SQL Server |
|------|-------|------------|--------|------------|
| 版本 | `@@version` | `version()` | `SELECT banner FROM v$version` | `@@version` |
| 列出表 | `information_schema.tables` | 同左 | `all_tables` | `information_schema.tables` |
| 列出列 | `information_schema.columns` | 同左 | `all_tab_columns` | 同左 |
| 字符串拼接 | `CONCAT(a,b)` | `a\|\|b` | `a\|\|b` | `a+b` |
| 注释 | `#` / `-- ` | `-- ` | `-- ` | `-- ` |
| 时间延迟 | `SLEEP(5)` | `pg_sleep(5)` | `dbms_pipe.receive_message(('a'),5)` | `WAITFOR DELAY '0:0:5'` |
