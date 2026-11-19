继续portswigger  Academy 的CSRF(跨站脚本攻击)

1. #### CSRF vulnerability with no defenses

   ![image-20261119173421082](01.png)

   这一题是最基本的CSRF![image-20261119181013153](02.png)

   通过bp拦截修改邮件请求可以发现，请求中只包含了要修改的邮箱地址和由于证明身份的cookie,由于cookie会由浏览器自行添加，只需要在恶意网页中构造一个自动提交的表单，即可修改受害者的邮箱

   ![image-20261119181740503](03.png)

2. #### CSRF where token validation depends on request method

   ![image-20261119181857172](04.png)

   由题干可知，这一题未防护其他提交方式![image-20261119182008048](05.png)

   使用bp拦截可以发现，提交的表单中不仅包含email,还有一个csrf值，且去掉之后无法通过![image-20261119182126181](06.png)

   但将请求转换为get后发现可以成功修改邮箱

   ![image-20261119182209790](07.png)

   所以构造一个get表单，自动提交即可![image-20261119182353581](08.png)

3. #### CSRF where token validation depends on token being present

   ![image-20261119182439841](09.png)

   在这一题中对请求拦截后发现提交了email和csrf但测试发现，去除csrf属性后提交依然可以通过![image-20261119182735157](10.png)

   所以构造一个只包含emil的表单即可

   ![image-20261119182917468](11.png)

4. #### CSRF where token is not tied to user session

   ![image-20261119182952255](12.png)

   从这一题的题干可以看出，这题的用于识别身份的session和用于防csrf的csrf值未进行绑定

   ![image-20261119183559526](13.png)![image-20261119183622318](14.png)

   进入题目可以看到提交中包含csrf，且去除和填入错误的值均无法通过，但测试后发现，登录账号wiener获得的csrf值，也可以用于更改账号carlos的邮箱

   ![](15.png)![image-20261119184540315](16.png)

   需要注意的是每次提交表单都会更换新的csrf值，所以如果通过bp拦截获得了csrf值，需要丢弃该包，以避免csrf失效。提取一个未使用的csrf将其加入恶意网页的表单中

   ![image-20261119184943648](17.png)

5. #### CSRF where token is duplicated in cookie

   ![image-20261119185132358](18.png)

   拦截提交可以发现csrf存在于cookie中![image-20261119185254358](19.png)

   ​	且判断是通过比对cookie中的csrf和提交的csrf，将两者改为一样的值就可以通过检测![image-20261119185843902](20.png)

   所以在恶意网站中先使用set-cookie改变cookie值，再提交即可

   ![image-20261119192440594](21.png)

6. #### SameSite Lax bypass via method override

   ![image-20261119210151042](22.png)

   这一题需要覆盖同源策略，在拦截后可以发现修改邮箱的表单只提交了email

   ![image-20261119210957764](23.png)

   但简单写了一个恶意网页测试可以发现并没有携带cookie

   ![image-20261119211654411](24.png)

   观察发现网站没有设置SameSite，默认为lax，get请求将会携带cookie，尝试使用get请求提交，发现被拒绝![image-20261119212538736](25.png)

   在参数中添加_method=POST后发现提交成功![image-20261119212704339](26.png)

   在恶意网站的表单中加入_method参数即可![image-20261119213115523](27.png)

7. #### SameSite Strict bypass via client-side redirect

   ![image-20261119220705996](28.png)

   在这一题中SameSite被设置为了Strict任何跨站的请求都不会携带cookie

   ![image-20261119221226662](29.png)

   拦截请求后，经过测试可以发现这题会接受get请求

   ![image-20261119221613609](30.png)

   所以只要找到页面上的跳转方式跳转到

   ```
   /my-account/change-email?email=1%402&submit=1
   ```

   就可以将邮箱改为1@2。对任意博客进行评论后发现评论成功的提示页面会在几秒后跳回博客，观察该页面源码发现跳转函数

   ![image-20261119221937961](31.png)

   ![image-20261119221948426](32.png)

   其中postId是可以控制的，所以在恶意页面使用`document.location`跳转到成功提示页面再跳转到更改邮箱，由于这两者是同源的，所以会携带cookie以完成更改

   ![image-20261119223537780](33.png)

8. #### SameSite Strict bypass via sibling domain

   需要pro，跳过

9. #### CSRF where Referer validation depends on header being present

   ![image-20261119223720993](34.png)

   用最基础的方式测试后发现，由于referer头而被拒绝

   ![image-20261119224304777](35.png)

   再repeater中多次尝试后发现去除referer元素后可以通过

   ![image-20261119225011188](36.png)

   在恶意页面中加入

   ```
   <meta name="referrer" content="no-referrer" />
   ```

   使得提交的表单中不带有referer

   ![image-20261119225312941](37.png)

10. CSRF with broken Referer validation

    ![image-20261119225913045](38.png)

    这一题中同样通过referer进行检测，但这题中去除referer不能通过检测

    ![image-20261119225837339](39.png)

    多次尝试后发现这一题中只要referer含有**0a97003004dc42f2801e2155009a0016.web-security-academy.net**就可以通过

    ![image-20261119230226707](40.png)

    所以构造恶意网站的链接使其链接中包含**0a97003004dc42f2801e2155009a0016.web-security-academy.net**，以绕过检测，同时加入

    ```
    <meta name='referrer' content='unsafe-URL' />
    ```

    以保证在referer中发送完整url

    ![image-20261119230905071](41.png)

    