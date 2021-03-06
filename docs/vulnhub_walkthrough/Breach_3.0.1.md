# Breach 3.0.1

![image-20200630215536009](assets/Breach_3.0.1.assets/image-20200630215536009.png)

反复检查确实没有网络上的问题。只是所有tcp端口都被过滤掉了。

扫描UDP

![image-20200630215840889](assets/Breach_3.0.1.assets/image-20200630215840889.png)

![image-20200630215640438](assets/Breach_3.0.1.assets/image-20200630215640438.png)

snmp

![image-20200630220426950](assets/Breach_3.0.1.assets/image-20200630220426950.png)

![image-20200630220752919](assets/Breach_3.0.1.assets/image-20200630220752919.png)



我们似乎只能得到一个域名`breach.local`以及一个用户名`milton`

这台机器只有一个端口打开了，如果要继续下去，我只能想到有可能需要敲端口，

在唯一的这些信息中，看哪些信息像端口。

![image-20200630220850403](assets/Breach_3.0.1.assets/image-20200630220850403.png)

看上去只有这个。`545\232\1876`

尝试敲端口

![image-20200701014546407](assets/Breach_3.0.1.assets/image-20200701014546407.png)

重新扫描

![image-20200701014605675](assets/Breach_3.0.1.assets/image-20200701014605675.png)

再次对端口进行nmap扫描

![image-20200701015422778](assets/Breach_3.0.1.assets/image-20200701015422778.png)

![image-20200701015443579](assets/Breach_3.0.1.assets/image-20200701015443579.png)

![image-20200701015524482](assets/Breach_3.0.1.assets/image-20200701015524482.png)

再繁琐的尝试之后在ssh看到了新的提示

![image-20200701015725306](assets/Breach_3.0.1.assets/image-20200701015725306.png)

再次敲端口

![image-20200701015839357](assets/Breach_3.0.1.assets/image-20200701015839357.png)

![image-20200701020421659](assets/Breach_3.0.1.assets/image-20200701020421659.png)

![image-20200701020628277](assets/Breach_3.0.1.assets/image-20200701020628277.png)

![image-20200701020654819](assets/Breach_3.0.1.assets/image-20200701020654819.png)

milton的密码在breach1中得到过

```bash
milton：thelaststraw
```



![image-20200701021309450](assets/Breach_3.0.1.assets/image-20200701021309450.png)





指向了一个新的链接

![image-20200701021508809](assets/Breach_3.0.1.assets/image-20200701021508809.png)

登录页面，抓包测试，可以被sql绕过

![image-20200701021829841](assets/Breach_3.0.1.assets/image-20200701021829841.png)

绕过后跳转的页面没有任何意义。

![image-20200701022214743](assets/Breach_3.0.1.assets/image-20200701022214743.png)

使用抓包中的验证凭证扫目录

![image-20200701025541069](assets/Breach_3.0.1.assets/image-20200701025541069.png)

![image-20200701025639900](assets/Breach_3.0.1.assets/image-20200701025639900.png)

![image-20200701025830620](assets/Breach_3.0.1.assets/image-20200701025830620.png)

转了很久，注意到这条线索

![image-20200701030505278](assets/Breach_3.0.1.assets/image-20200701030505278.png)

或许应该从数据库下手

重新把登录时的包存为文件，sqlmap跑一下

```bash
sqlmap -r login_pack.txt --dbs --batch --dump
```

![image-20200701032400208](assets/Breach_3.0.1.assets/image-20200701032400208.png)

```bash
sqlmap -r login_pack.txt --dbs --batch --dump --level=5 --risk=3 --tamper=equaltolike
```

![image-20200701033432782](assets/Breach_3.0.1.assets/image-20200701033432782.png)

![image-20200701033401185](assets/Breach_3.0.1.assets/image-20200701033401185.png)

然后检查必查项mysql.user

```bash
sqlmap -r login_pack.txt -D mysql -T user --batch --dump --level=5 --risk=3 --tamper=equaltolike
```

