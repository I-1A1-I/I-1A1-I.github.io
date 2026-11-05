最近重新安装了burp suite。作为测试尝试通关并记录 portswigger Labs

这里是第一篇SQL injection

1. #### SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

   ![image-20261105000406596](../content/posts/sql/01.png)

   题目很明确给出了产品分类选项所使用的语句

   ![image-20261105000737032](../content/posts/sql/02.png)

   进入题目后可以看到给出了多个选项，点击Accessories后可以看到跳转页面的url为

   ```
   https://0ae600bc044ca55582e0ab2900370064.web-security-academy.net/filter?category=Accessories
   ```

   按照描述中所说的查询语句在Accessories后加上

   ```
   'or 1=1 -- 
   ```

   将查询语句变为

   ```
   SELECT * FROM products WHERE category = 'Accessories'or 1=1 -- ' AND released = 1
   ```

   其中--后的被视为注释，不会被解析，即可以查询所以商品。

   ![image-20261105001400221](../content/posts/sql/03.png)

2. #### SQL injection vulnerability allowing login bypass、

   ![image-20261105001920357](../content/posts/sql/04.png)

   题目指出登录功能存在漏洞且要求登录administrator用户以通关

   ![image-20261105002435735](../content/posts/sql/05.png)

   进入到题目内部可以看到，相比于上一题，右上角多了一个My account

   ![image-20261105002534424](../content/posts/sql/06.png)

   点击进入后可以看到一个简易的登录框，使用万能密码`administrator' or 1=1 -- `尝试登录

   ![image-20261105002700570](../content/posts/sql/07.png)

   ![image-20261105002724193](../content/posts/sql/08.png)

   成功登录，显示通关

3. #### SQL injection attack, querying the database type and version on Oracle

   ![image-20261105002826591](../content/posts/sql/09.png)

   可以看到题目说明了注入点在产品分类选项，要求用union攻击，并说明了此数据库为Oracle

   ![image-20261105003113766](../content/posts/sql/10.png)

   进入题目可以看到上方给出了需要显示的内容，显然为v$version的内容，在使用union之前先使用order by 来探测需要输出的列数

   ![image-20261105004041334](../content/posts/sql/11.png)

   ![image-20261105004113814](../content/posts/sql/12.png)

   可以看到按第二列排序时能正常显示，按3时会报错，说明一共2列。使用`union select null,banner from v$version --` 进行攻击，可以看到成功显示了目标信息

   ![image-20261105004400969](../content/posts/sql/13.png)

   

4. #### SQL injection attack, querying the database type and version on MySQL and Microsoft

   ![image-20261105004510969](../content/posts/sql/14.png)

   与前一题相似，只是这次攻击的是mysql和microsoft数据库

   ![image-20261105004810850](../content/posts/sql/15.png)

   其他步骤相同，将最后攻击的语句换为对应数据库的语法即可

5. #### SQL injection attack, listing the database contents on non-Oracle databases

   ![image-20261105005045690](../content/posts/sql/16.png)

   进入题目可以看到题目说明了漏洞与之前相同，但要求查询其他表的内容以登录administrator账号

   ![image-20261105005352999](../content/posts/sql/17.png)

   ![image-20261105005422895](../content/posts/sql/18.png)

   为了便于查看，这里去除了攻击中冒号之前的内容，现在只会显示攻击得到的数据，由于标题排除了Oracle，在使用mysql和PostgreSQL的语法进行查询后，可以看到这个网站使用的是PostgreSQL数据库。使用`'union select null,schema from information_schema.schemata --+`进行查询

   ![image-20261105010140654](../content/posts/sql/19.png)

   显然information_schema和pg_catalog都是数据库自带的，需要的数据在public中进一步查询public中有哪些表，使用`'union select null,table_name from information_schema.tables where table_schema='public' --+`

   ![image-20261105010412004](../content/posts/sql/20.png)

   显然用户数据存储于users_zxwabc中，现在查询该表中存在哪些列，使用`'union select null,column_name from information_schema.columns where table_name='users_zxwabc' --+`

   ![image-20261105010636124](../content/posts/sql/21.png)

   最后查询用户名和密码，使用`'union select username_znxksg,password_xjsenj from users_zxwabc --+`

   ![image-20261105010750407](../content/posts/sql/22.png)

   得到了administrator的密码，在登录页面登录即可

6. #### SQL injection attack, listing the database contents on Oracle

   ![image-20261105010912460](../content/posts/sql/23.png)

   和上一题相似，只是换成了Oracle数据库

   ![image-20261105012521215](../content/posts/sql/24.png)

   显然只有PETER不是Oracle默认用户，使用`'union select null,table_name from all_tables where owner='PETER' --`查询

   ![image-20261105012637036](../content/posts/sql/25.png)

   进一步使用`'union select null,column_name from all_tab_columns where table_name='USERS_XBDFHG' --`查询USERS_XBDFHG的列名

   ![image-20261105012905097](../content/posts/sql/26.png)

   最后使用`'union select PASSWORD_KMJELD,USERNAME_VIBASQ from USERS_XBDFHG --`查询该表的内容，使用管理员账号登录即可

   ![image-20261105013029794](../content/posts/sql/27.png)

7. #### SQL injection UNION attack, determining the number of columns returned by the query

   ![image-20261105013336260](../content/posts/sql/28.png)

   与之前类似，但要求直接确定查询列数，由于我在一开始就对列数进行了判断，这里就不多赘述了

8. #### SQL injection UNION attack, finding a column containing text

   ![image-20261105013532818](../content/posts/sql/29.png)

   这一题要求找出可以输出字符串的位置并输出

   ![image-20261105013735904](../content/posts/sql/30.png)

   进入题目后可以看到，在上方给出了要求输出的内容，在使用order by进行测试后可以得出该查询一共3列使用`'union select null,null,null --+`后可以正常显示![image-20261105013950539](../content/posts/sql/31.png)

   依次尝试将3个null替换为目标字符串

   ![image-20261105014046217](../content/posts/sql/32.png)

   ![image-20261105014134434](../content/posts/sql/33.png)

   可以看到在尝试第二个位置时显示成功

9. #### SQL injection UNION attack, retrieving data from other tables

   与之前内容相似，不在赘述

10. #### SQL injection UNION attack, retrieving multiple values in a single column

    ![image-20261105014555136](../content/posts/sql/34.png)

    其他内容与上一题相同，但这题的输出列只有一条

    ![image-20261105015430382](../content/posts/sql/35.png)

    进入题目测试后发现此题的数据库为PostgreSQL，查询列一共2条，但只显示第二列，其中第二列为字符串，这里有两种办法：

    1. 第一种，将密码和用户名拼接后输出，使用`'union select 1,username||'-'||password from users --`，其中||为连接符号，而-是将两段分开便于区分![image-20261105015817765](../content/posts/sql/36.png)

    2. 第二种，先查询用户名，在获得用户名后查询指定用户名的密码

       ![image-20261105015933125](../content/posts/sql/37.png)

       ![image-20261105020002555](../content/posts/sql/38.png)

    得到用户名，密码后登录即可

11. #### Blind SQL injection with conditional responses

    ![image-20261105020142640](../content/posts/sql/39.png)

    这一题说明了在cookie中存在一出注入点，如果查询到内容会在页面上显示Welcome back![image-20261105020406855](../content/posts/sql/40.png)

    进入题目后可以发现，在任意的商品页面右上角会显示Welcome back

    ![image-20261105020558808](../content/posts/sql/41.png)

    使用bp进行抓包可以看到传入了`Cookie: TrackingId=iAzNc4W7chWdqc7H;`

    ![image-20261105020726291](../content/posts/sql/42.png)

    ![image-20261105020751957](../content/posts/sql/43.png)

    ![image-20261105020815933](../content/posts/sql/44.png)

    ![image-20261105020900583](../content/posts/sql/45.png)

    在传入repeater模块测试后可以发现，该数据库为PostgreSQL，该查询有一列字符串类型的数据，由于不会显示查询内容，无法使用union查询，这里尝试拼接判断语句

    ![image-20261105021234520](../content/posts/sql/46.png)

    ![image-20261105021253971](../content/posts/sql/47.png)

    可以看到当and后为真时显示Welcome back，为假时不显示，可以以此来得到想要的内容

    ![image-20261105022721772](../content/posts/sql/48.png)

    多次尝试后可以看到administrator用户的密码为20位，这里可以使用脚本进行二分法以更快的得到答案，由于在提示中写明了密码只包含数字和小写字母，这里使用最简单的办法进行爆破。将请求转发到Intruder模块后将payload改为`' and (select count((select password from users where username='administrator' and password like '____________________')))=1--`将20个下划线全部添加为payload positions，使用sniper attack模式,在payload中添加所有的数字和小写字母![image-20261105023753435](../content/posts/sql/49.png)

    然后开始爆破。

    爆破需要一定的时间，注意如果长时间未与靶场通信，靶场会被销毁，由于密码是随机生成的，重新进入后密码将会变化，需要重新获得密码。

    ![image-20261105045357534](../content/posts/sql/50.png)

    按长度排序后，按位置拼接payload即可获得密码`m52hcj27kwjtdu2997b3`

    ![image-20261105045546472](../content/posts/sql/51.png)

    

12. #### Blind SQL injection with conditional errors

    ![image-20261105045654601](../content/posts/sql/52.png)

    题目说明了注入点与上一题相同是cookie,而这条会返回错误信息

    ![image-20261105050924920](../content/posts/sql/53.png)

    ![image-20261105050905512](../content/posts/sql/54.png)

    传入repeater模块后测试可知当搜索失败时，会返回网络服务错误，成功时正常会返回页面，且数据库为Oracle

    !![image-20261105052715292](../content/posts/sql/55.png)

    ![image-20261105052402898](../content/posts/sql/56.png)

    可以看到在构造的payload   'union (select case when **条件** then to_char(1/0) else null end from dual)--`中当条件为真时会引起报错，为假时可以正常显示页面，故可以构造payload获取密码

    ![image-20261105053246281](../content/posts/sql/57.png)

    ![image-20261105053306999](../content/posts/sql/58.png)

    ![image-20261105053503861](../content/posts/sql/59.png)

    ![image-20261105053519434](../content/posts/sql/60.png)

    经过简单的测试可以得到该密码为20位，且第一位为d,之后的内容与上一题相同，故不在赘述

13. ####  Visible error-based SQL injection

    ![image-20261105054019894](../content/posts/sql/61.png)

    题目说明了注入点为cookie

    ![image-20261105054342300](../content/posts/sql/62.png)

    测试后发现这一题会返回报错信息，同时查询语句为`SELECT * FROM tracking WHERE id = 'TrackingId'`其中TrackingId为cookie中提供的值

    ![image-20261105055029708](../content/posts/sql/63.png)

    测试各数据库的特征后可知数据库为PostgreSQL

    ![image-20261105055315742](../content/posts/sql/64.png)

    尝试拼接语句后报错，由报错信息可以推断，后台限制了字符串长度，故需要更短的方式

    ![image-20261105055705446](../content/posts/sql/65.png)

    可以看到通过将字符串转为int产生错误，可以将字符串内容通过报错信息带出

    ![image-20261105060052614](../content/posts/sql/66.png)

    ![image-20261105060147837](../content/posts/sql/67.png)

    可以看到第一条用户名为administrator其密码为z3yxrrsuqvw7l5gounvi

    

14. #### Blind SQL injection with time delays

    ![image-20261105053813644](../content/posts/sql/68.png)

    在这一题中注入点依然是cookie,但没有任何回显，但可以使用时间盲注，题目要求触发10秒延迟

    ![image-20261105060520873](../content/posts/sql/69.png)

    进入页面后使用pg_sleep(10)后可以看到右下角显示的响应时间比正常情况下慢了10秒

    

15. #### Blind SQL injection with time delays and information retrieval

    ![image-20261105060749558](../content/posts/sql/70.png)

    这一题要求使用时间盲注获得密码

    ![image-20261105061002116](../content/posts/sql/71.png)

    可以看到这一题与上一题一模一样，故构建时间盲注payload

    ![image-20261105061139571](../content/posts/sql/72.png)

    ![image-20261105061155138](../content/posts/sql/73.png)

    可以看到在payload`'and (select case when 条件 then pg_sleep(3) else null end)is null --+`中，当条件为真时延迟3.267秒，为假时延迟0.258秒，显然结合11题的payload可以获得密码，故不在赘述

16. ####  Blind SQL injection with out-of-band interaction和Blind SQL injection with out-of-band data exfiltration

    这两题为dnslog外带，由于题目屏蔽了其他dnslog网站，必须使用burp suite pro版本中的Burp Collaborator我安装的版本为社区版故不作演示

17. #### SQL injection with filter bypass via XML encoding

    ![image-20261105061951200](../content/posts/sql/74.png)

    这一题要求在WAF下进行渗透

    ![image-20261105062158849](../content/posts/sql/75.png)

    按照题目的指示找到注入点

    ![image-20261105062256194](../content/posts/sql/76.png)

    简单尝试后被WAF阻拦，需要进行WAF绕过

    ![image-20261105062411639](../content/posts/sql/77.png)

    安装BApp store 中的Hackvertor，并使用其对攻击进行编码

    ![image-20261105062745917](../content/posts/sql/78.png)

    在Hackvertor中输入payload 并用dec_entities进行编码，使用结果进行攻击

    ![image-20261105062933385](../content/posts/sql/79.png)

    得到了users表的列名

    ![image-20261105063031495](../content/posts/sql/80.png)![image-20261105063044499](../content/posts/sql/81.png)

    根据获得的列名写出查询的payload再次编码即可获得用户名和密码









