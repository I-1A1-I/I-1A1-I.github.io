查询到靶机的ip并使用nmap进行扫描

![01.png](../content/posts/dc2/01.png)

![02.png](../content/posts/dc2/02.png)

可以看到只开放了80tcp端口和7744ssh端口

，使用浏览器打开，但直接打开无法访问。根据nmap的信息，在/etc/hosts中192.168.72.130到dc-2的映射即可

![03.png](../content/posts/dc2/03.png)

![04.png](../content/posts/dc2/04.png)

可以看到第一个flag

![05.png](../content/posts/dc2/05.png)

flag中提到登录，但页面上并没有登录选项，所以使用dirb进行爆破

![06.png](../content/posts/dc2/06.png)

找到管理员登录页面

![07.png](../content/posts/dc2/07.png)

可以看到回显可以用来探测用户名

![08.png](../content/posts/dc2/08.png)

![09.png](../content/posts/dc2/09.png)	

根据flag提示使用cewl生成密码

![10.png](../content/posts/dc2/10.png)

使用bp进行爆破，但多次尝试后没能成功。结合flag，转而使用wpscan查询其他用户

![11.png](../content/posts/dc2/11.png)

![12.png](../content/posts/dc2/12.png)

尝试尝试用wpscan重新进行爆破进行爆破

![13.png](../content/posts/dc2/13.png)

找到jerry和tom的密码

![14.png](../content/posts/dc2/14.png)

登录后找到第二个flag

![15.png](../content/posts/dc2/15.png)

尝试查看登录后的页面，找到一处文件上传，但无法写入

![16.png](../content/posts/dc2/16.png)

在评论中发现一处xss![17.png](../content/posts/dc2/17.png)

尝试使用beef-xss进行攻击，但等待一段时间后除了本机没有其他连接

![18.png](../content/posts/dc2/18.png)

找到密码修改功能，但多次尝试后无法sql注入

![19.png](../content/posts/dc2/19.png)

尝试使用用户名和密码登录ssh，使用tom登录成功

![20.png](../content/posts/dc2/20.png)

找到了flag3的位置，但rbash无法使用cat进行读取

![21.png](../content/posts/dc2/21.png)

使用compgen -c查看能用的指令，发现vi可以使用，用vi阅读flag

![22.png](../content/posts/dc2/22.png)

使用vi进行提权

![23.png](../content/posts/dc2/23.png)

![24.png](../content/posts/dc2/24.png)

在jerry路径下找到flag4.txt

![25.png](../content/posts/dc2/25.png)

根据flag3的提示需要更换账号，使用su命令，切换到jerry

![26.png](../content/posts/dc2/26.png)

查找能使用suid提权的文件，但没有发现

![27.png](../content/posts/dc2/27.png)

结合flag4尝试git提权，输入sudo -l可以看到可以无需密码的使用git

![28.png](../content/posts/dc2/28.png)

进行提权

![29.png](../content/posts/dc2/29.png)

![30.png](../content/posts/dc2/30.png)

![31.png](../content/posts/dc2/31.png)

在root中找到最终flag

![32.png](../content/posts/dc2/32.png)







