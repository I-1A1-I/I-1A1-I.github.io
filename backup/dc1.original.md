确保kali和dc1都使用nat模式

使用nmap -snr查询有那些ip

![image-20261121021830087](../content/posts/dc1/01.png)

对这些ip分别使用namp进行找到目标ip和端口

![image-20261121022037141](../content/posts/dc1/02.png)

使用浏览器访问80端口，进入一个drupal网站，在找回密码处发现，使用已有的用户名和未注册的用户名回显不同

![image-20261121023631464](../content/posts/dc1/03.png)

这里输入了admin

![image-20261121023833580](../content/posts/dc1/04.png)

尝试爆破发现有次数限制

![image-20261121024134324](../content/posts/dc1/05.png)

在源码中找到Drupal的版本![image-20261121030857863](../content/posts/dc1/06.png)

在msfconsle中搜索drupal，找到符合版本的内容![image-20261121031023074](../content/posts/dc1/07.png)

![image-20261121031125934](../content/posts/dc1/08.png)

填入信息后发起攻击成功获得shell

![image-20261121031251417](../content/posts/dc1/09.png)

获得第一个flag![image-20261121031407120](../content/posts/dc1/10.png)

根据提示在/var/www/sites/default中找到第二key

![image-20261121033421875](../content/posts/dc1/11.png)

现在需要登录mysql，需要升级到交互式的shell

![](../content/posts/dc1/12.png)

一步步查询出用户名和密码

![image-20261121043951545](../content/posts/dc1/13.png)

![image-20261121044350401](../content/posts/dc1/14.png)但显然密码经过哈希加密在网站根目录下查找名字包含hash的文件

![image-20261121045622271](../content/posts/dc1/15.png)

读取该文件发现其使用了一个名为user_hash_password的函数，在文件中并没有定义该函数，所以其应该来自包含的/includes/password.inc中

![image-20261121050743860](../content/posts/dc1/16.png)

在password.inc中找到了改函数，![image-20261121051131426](../content/posts/dc1/17.png)

阅读全部函数后发现，虽然生成过程使用了环境变量加盐，但并不影响检查函数，所以将文件复制到自己的环境中运行得到加密的asd。将检测函数修改为直接接受储存的密码后，验证可以通过。

![image-20261121055039189](../content/posts/dc1/18.png)

![image-20261121055021573](../content/posts/dc1/19.png)

![image-20261121055153832](../content/posts/dc1/20.png)

将数据库中admin的密码更改为计算得到的哈希值，即可使用asd密码登录

![image-20261121132143028](../content/posts/dc1/21.png)

由于等了很久爆破带来的限制还没有到期，进入mysql清除登录记录，找到登录记录

![image-20261121131701549](../content/posts/dc1/22.png)

并删除

![image-20261121131911684](../content/posts/dc1/23.png)

成功登录进入管理页面

![image-20261121132311337](../content/posts/dc1/24.png)

再content中找到flag3![image-20261121132631365](../content/posts/dc1/25.png)

![image-20261121132655157](../content/posts/dc1/26.png)

根据flag的提示查找perm为4000的文件，进行suid提权

![image-20261121190148192](../content/posts/dc1/27.png)

发现find可以用于提权，尝试后whoami输出root

![image-20261121190912019](../content/posts/dc1/28.png)

在/root下找到最后一个flag![image-20261121191117626](../content/posts/dc1/29.png)
