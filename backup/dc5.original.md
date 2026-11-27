

扫描得到dc-5的ip地址

![01.png](../content/posts/dc5/01.png)

使用nmap进行端口扫描，发现3个开放端口

![02.png](../content/posts/dc5/02.png)

使用浏览器访问

![03.png](../content/posts/dc5/03.png)

使用dirb扫描，没有能利用的内容

![04.png](../content/posts/dc5/04.png)

网页前几项都是纯文字，但contact中包含一个表单

![05.png](../content/posts/dc5/05.png)

查看源代码发现，这个表单以get形式跳转到thankyou.php

![06.png](../content/posts/dc5/06.png)

使用sqlmap注入失败

![07.png](../content/posts/dc5/07.png)

同时页面上也没有任何回显或者cookie，查看开放的rpcbind，发现未开启NFS

![08.png](../content/posts/dc5/08.png)

注意到thankyou.php下方的年份会变化，怀疑存在文件包含

![09.png](../content/posts/dc5/09.png)

![10.png](../content/posts/dc5/10.png)

使用bp爆破参数名，由于数据过多会被限速，使用多个参数在pitch模式下同时爆破，在获得的结果中的参数在单独爆破

![11.png](../content/posts/dc5/11.png)

![12.png](../content/posts/dc5/12.png)

加载index.php后确认会被解析

![13.png](../content/posts/dc5/13.png)

查看ngnix的配置文件

![14.png](../content/posts/dc5/14.png)

上传一句话木马后使用蚁剑连接

![15.png](../content/posts/dc5/15.png)

![16.png](../content/posts/dc5/16.png)

使用nc发送反弹shell

![17.png](../content/posts/dc5/17.png)

![18.png](../content/posts/dc5/18.png)

查找suid提权

![19.png](../content/posts/dc5/19.png)

查看exim4版本

![20.png](../content/posts/dc5/20.png)

尝试多种提权均以失败告终

![21.png](../content/posts/dc5/21.png)

![22.png](../content/posts/dc5/22.png)

找到screen4.5.0的漏洞

![23.png](../content/posts/dc5/23.png)

在kali开启服务后

![24.png](../content/posts/dc5/24.png)

切换到/tmp下载并执行payload

![25.png](../content/posts/dc5/25.png)

提取成功，在/root下找到flag

![26.png](../content/posts/dc5/26.png)