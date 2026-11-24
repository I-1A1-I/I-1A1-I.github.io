1. #### Unprotected admin functionality

   ![01.png](../content/posts/access-control/01.png)

   题目要求删除指定用户，进入页面后没有发现管理员面板，查看robots.txt，找到管理员面板的入口

   ![02.png](../content/posts/access-control/02.png)

   进入该页面，点击delete删除对应用户

   ![03.png](../content/posts/access-control/03.png)

2. #### Unprotected admin functionality with unpredictable URL

   ![04.png](../content/posts/access-control/04.png)

   进入页面查看源码，发现管理员面板的入口，进入并删除对应账号

   ![05.png](../content/posts/access-control/05.png)

   ![06.png](../content/posts/access-control/06.png)

3. #### User role controlled by request parameter

   ![07.png](../content/posts/access-control/07.png)

   登录后访问/admin发现被拦截

   ![08.png](../content/posts/access-control/08.png)

   使用bp拦截请求，发现提交的参数包含Admin=false

   ![09.png](../content/posts/access-control/09.png)

   将false改为true后放包，成功进入管理员面板

   ![10.png](../content/posts/access-control/10.png)

4. #### User role can be modified in user profile

   ![11.png](../content/posts/access-control/11.png)

   根据题目提示，需要将账号的roleid改为2，登录后尝试拦截修改密码的请求

   ![12.png](../content/posts/access-control/12.png)

   再请求中加入"roleid":2后发送，再返回中可以看到roleid已经变为了2

   ![13.png](../content/posts/access-control/13.png)

   在账号页面出现管理员面板入口

   ![14.png](../content/posts/access-control/14.png)

5. #### User ID controlled by request parameter

   ![15.png](../content/posts/access-control/15.png)

   登录后看到在账号页面展示了账号的API key

   ![16.png](../content/posts/access-control/16.png)

   将提交的参数id改为carlos可以看到页面变成了carlos的id和API key

   ![17.png](../content/posts/access-control/17.png)

   将API key提交至Submit solution![18.png](../content/posts/access-control/18.png)

6. #### User ID controlled by request parameter, with unpredictable user IDs

   ![19.png](../content/posts/access-control/19.png)

   登录后来到账号页面，发现与上一题不同，这题的id为一个编号而不是用户名

   ![20.png](../content/posts/access-control/20.png)

   进入主页里的文章，发现左上角显示了作者

   ![21.png](../content/posts/access-control/21.png)

   找到一篇carlos的文章并点击作者名，在跳转的页面中可以看到作者的userId

   ![22.png](../content/posts/access-control/22.png)

   将其替换账号页中的id即可进入carlos的账号页面

   ![23.png](../content/posts/access-control/23.png)

7. #### User ID controlled by request parameter with data leakage in redirect

   ![24.png](../content/posts/access-control/24.png)

   登录后尝试将请求中的id更改为carlos，发现被跳转到了登录页面，但查看bp的网络记录可以发现，在302的跳转包中，依然包含了完整的网络内容

   ![25.png](../content/posts/access-control/25.png)

   其中包含了carlos用户的API key

   ![26.png](../content/posts/access-control/26.png)

8. #### User ID controlled by request parameter with password disclosure

   ![27.png](../content/posts/access-control/27.png)

   登录后看到修改邮箱和密码的页面

   ![28.png](../content/posts/access-control/28.png)

   

   将修改密码栏的类型去掉之后发现，其中预填的内容为用户真实密码

   ![29.png](../content/posts/access-control/29.png)

   查看bp的流量记录，发现密码在传输中出现

   ![30.png](../content/posts/access-control/30.png)

   尝试将请求中的id改为carlos，找到carlos的密码

   ![31.png](../content/posts/access-control/31.png)

   尝试改为admin，发现返回不同，说明没有该账号

   ![32.png](../content/posts/access-control/32.png)

   尝试改为administrator，得到administrator的密码

   ![33.png](../content/posts/access-control/33.png)

9. #### Insecure direct object references

   ![34.png](../content/posts/access-control/34.png)

   在这一题中有一个聊天室，在聊天室中点击view transcript后会以txt的形式下载聊天记录

   ![35.png](../content/posts/access-control/35.png)

   注意到文件名从2开始递增

   ![36.png](../content/posts/access-control/36.png)

   在bp的流量记录中找到下载路径，将文件名改为1.txt，得到包含carlos密码的聊天记录

   ![37.png](../content/posts/access-control/37.png)

10. #### URL-based access control can be circumvented

    ![38.png](../content/posts/access-control/38.png)

    尝试访问/admin显示访问被拒

    ![39.png](../content/posts/access-control/39.png)

    访问主页，在bp中在请求头中添加X-Original-URL:/admin

    ![40.png](../content/posts/access-control/40.png)

    找到删除carlos的链接，将X-Original-URL改为/admin/delete，并添加参数username=carlos，提交即可删除

    ![41.png](../content/posts/access-control/41.png)

11. #### Method-based access control can be circumvented

    ![42.png](../content/posts/access-control/42.png)

    登录管理员账号后再管理员面板找到了给用户提权，降权的面板

    ![43.png](../content/posts/access-control/43.png)

    拦截提权请求

    ![44.png](../content/posts/access-control/44.png)

    退出登录后，登录wiener，用新的cookie替代消息中的cookie，发送显示无授权

    ![45.png](../content/posts/access-control/45.png)

    将请求方式改为get发送，发现修改成功

    ![46.png](../content/posts/access-control/46.png)

12. #### Multi-step process with no access control on one step

    ![47.png](../content/posts/access-control/47.png)

    与上一关类似，登录管理员后拦截提权请求，但因为这关使用了二次确认，要拦截确认时的请求

    ![48.png](../content/posts/access-control/48.png)

    更改cookie后发送即可

    ![49.png](../content/posts/access-control/49.png)

13. #### Referer-based access control

    ![50.png](../content/posts/access-control/50.png)

    与前两题类似，但11题是利用GET请求方式未能正确处理，12题是利用权限判断再第一步，之后不再做判断，而这题是判断referer，与12题做法一样，不在赘述



