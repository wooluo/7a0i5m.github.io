# hackme



![image-20200611225953118](assets/hackme.assets/image-20200611225953118.png)



![image-20200611230407031](assets/hackme.assets/image-20200611230407031.png)

![image-20200612015742052](assets/hackme.assets/image-20200612015742052.png)

打目录打了很多次都只打到了两个目录

/login和/uploads

![image-20200612015957032](assets/hackme.assets/image-20200612015957032.png)

简单测试但没发现任何线索之后，注册新帐号evil尝试登录

![image-20200612020121280](assets/hackme.assets/image-20200612020121280.png)

近乎完美的“来注入我”的提示。

burpsuite抓包注入

但是手注无反应sqlmap跑一下

--data 加上抓包成功的发送内容。--cookie带登录的cookie

```bash
sqlmap -u "http://192.168.43.36/welcome.php" --data "search=oscp" --cookie "PHPSESSID=j2fjanfrtb5pd2mdhemqcfqlal" -dump
```

![image-20200612022440539](assets/hackme.assets/image-20200612022440539.png)



![image-20200612022617365](assets/hackme.assets/image-20200612022617365.png)

```bash
superadmin：Uncrackable
```

使用这个凭证重新登录

![image-20200612022809200](assets/hackme.assets/image-20200612022809200.png)

制作一个gif phprshell上传

![image-20200612024101407](assets/hackme.assets/image-20200612024101407.png)

![image-20200612024609921](assets/hackme.assets/image-20200612024609921.png)

![image-20200612024708572](assets/hackme.assets/image-20200612024708572.png)



![image-20200612024830888](assets/hackme.assets/image-20200612024830888.png)

查看home

![image-20200612025718743](assets/hackme.assets/image-20200612025718743.png)

SUID

直接运行。root

![image-20200612025759265](assets/hackme.assets/image-20200612025759265.png)