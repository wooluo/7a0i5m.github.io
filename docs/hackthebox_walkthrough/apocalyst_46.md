# Apocalyst\_46

![image-20200919173407792](assets/Apocalyst_46.assets/image-20200919173407792.png)

显示不完整。将apocalyst加入host解析

![image-20200919174402907](assets/Apocalyst_46.assets/image-20200919174402907.png)

正常了

![image-20200919175830422](assets/Apocalyst_46.assets/image-20200919175830422.png)

![image-20200919180151889](assets/Apocalyst_46.assets/image-20200919180151889.png)

首先这个盒子很扯淡。，正常字典打的目录会让你止步不前。应该先收集一下，我怀疑oscp的中的某个其实也有这个问题。所以还是做了。

看网页发现，how long we have 这个作者，怀疑是用户名`falaraki`

![image-20200919180308003](assets/Apocalyst_46.assets/image-20200919180308003.png)

写cewl收集一下网站，。然后。再附加到正常字典里再打目录

```bash
$ cewl 10.10.10.46 > wordlist.txt
```

![image-20200919180429377](assets/Apocalyst_46.assets/image-20200919180429377.png)

多个页面会频繁出现这个图片，下载。

![image-20200919180550486](assets/Apocalyst_46.assets/image-20200919180550486.png)

从这开始整个靶机就非常扯了其实 安装一个图片解码工具

```bash
$ apt-get install steghide
$ steghide extract -sf apocalyst.jpg
Enter passphrase:
wrote extracted data to "list.txt".
```

![image-20200919180724719](assets/Apocalyst_46.assets/image-20200919180724719.png)

提示输入的时候直接回车跳过

生成了一个新的文件list.txt，看文件内容应该是一个字典。

![image-20200919180825667](assets/Apocalyst_46.assets/image-20200919180825667.png)

我们已经看到 网站应该是另一个wp，现在有有一个用户名`falaraki`,一个字典。

扔到wpscan里尝试暴力一下

```bash
$ wpscan --url http://10.10.10.46 --wordlist /root/HackTheBox/Apocalyst_10.10.10.46/list.txt --username falaraki
```

![image-20200919180959968](assets/Apocalyst_46.assets/image-20200919180959968.png)

```bash
login: falaraki
password: Transclisiation
```

使用这个凭证从wp后台登陆。

![image-20200919181312068](assets/Apocalyst_46.assets/image-20200919181312068.png)

![image-20200919181425381](assets/Apocalyst_46.assets/image-20200919181425381.png)

老套路通过修改`single.php`文件为php reverse shell 文件内容

![image-20200919181530202](assets/Apocalyst_46.assets/image-20200919181530202.png)

修改为kali IP，并上传

![image-20200919181741494](assets/Apocalyst_46.assets/image-20200919181741494.png)

kali本地设立监听

![image-20200919181821613](assets/Apocalyst_46.assets/image-20200919181821613.png)

回到网页随便点一个文章题目就能得到shell

![image-20200919181912149](assets/Apocalyst_46.assets/image-20200919181912149.png)

先尝试获得更方便的bash shell

```bash
$ /bin/bash -i
```

![image-20200919182037066](assets/Apocalyst_46.assets/image-20200919182037066.png)

![image-20200919182115948](assets/Apocalyst_46.assets/image-20200919182115948.png)

获得user

![image-20200919182207913](assets/Apocalyst_46.assets/image-20200919182207913.png)

注意到一个隐藏文件`.secret`

![image-20200919182332949](assets/Apocalyst_46.assets/image-20200919182332949.png)

查看内容发现一串base64编码

![image-20200919182426741](assets/Apocalyst_46.assets/image-20200919182426741.png)

```bash
S2VlcCBmb3JnZXR0aW5nIHBhc3N3b3JkIHNvIHRoaXMgd2lsbCBrZWVwIGl0IHNhZmUhDQpZMHVBSU50RzM3VGlOZ1RIIXNVemVyc1A0c3M=
```

base64解码

![image-20200919182524254](assets/Apocalyst_46.assets/image-20200919182524254.png)

所以用户的凭证可能是

```bash
falaraki:Y0uAINtG37TiNgTH!sUzersP4ss
```

ssh登陆成功

![image-20200919201301296](assets/Apocalyst_46.assets/image-20200919201301296.png)

上传linEnum帮助枚举

LinEnum: [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)

枚举发现passwd可写

![image-20200919201530314](assets/Apocalyst_46.assets/image-20200919201530314.png)

直接在passwd文件最后追加一行新的用户及对应凭证。将最后面的用户部分改为root

![image-20200919201911626](assets/Apocalyst_46.assets/image-20200919201911626.png)

这相当于增加了新的root用户

su切换到新用户就获得了root shell

![image-20200919202046618](assets/Apocalyst_46.assets/image-20200919202046618.png)

