扫描找到dc-4的网址

![01.png](../content/posts/dc4/01.png)

端口扫描后发现开放了22端口和80端口

![02.png](../content/posts/dc4/02.png)

使用浏览器访问网站看到一个登录页面

![03.png](../content/posts/dc4/03.png)

使用进行dirb扫描，没有找到有效目标

![04.png](../content/posts/dc4/04.png)

对登录框尝试爆破失败

![05.png](../content/posts/dc4/05.png)

使用sqlmap注入失败

![06.png](../content/posts/dc4/06.png)

尝试使用hydra爆破ssh失败

![07.png](../content/posts/dc4/07.png)

询问ai后发现是字典用的不行

![08.png](../content/posts/dc4/08.png)

使用密码登录后看到一个命令选项

![09.png](../content/posts/dc4/09.png)

进入后看到有可以选择执行的命令

![10.png](../content/posts/dc4/10.png)

使用bp拦截请求发现命令在post参数中明文传输

![11.png](../content/posts/dc4/11.png)

尝试修改指令，发现可以执行

 ![12.png](../content/posts/dc4/12.png)

使用nc建立反弹shell

![13.png](../content/posts/dc4/13.png)

![14.png](../content/posts/dc4/14.png)

查找可以suid提权的文件

![15.png](../content/posts/dc4/15.png)

到exim4的目录下查看exim的版本

![16.png](../content/posts/dc4/16.png)

查找该版本的漏洞

![17.png](../content/posts/dc4/17.png)

到目录下开启服务器

![18.png](../content/posts/dc4/18.png)

在反弹shell中下载，显示权限不足

![19.png](../content/posts/dc4/19.png)

进入/tmp再次尝试下载

![20.png](../content/posts/dc4/20.png)

运行显示权限不足

![21.png](../content/posts/dc4/21.png)

使用chmod进行修改

![22.png](../content/posts/dc4/22.png)

运行后提权成功

![23.png](../content/posts/dc4/23.png)

在/root下找到flag

![24.png](../content/posts/dc4/24.png)