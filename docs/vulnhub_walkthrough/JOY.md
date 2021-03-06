# JOY

> 反复利用ProFTPD < 1.3.5的任意用户复制替换漏洞

![image-20200609085426330](assets/JOY.assets/image-20200609085426330.png)

```bash
sudo nmap -O -A -Pn -T4 -v -p21,22,25,80,110,139,143,445,465,587,993,995 192.168.1.110

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.2.10
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxr-x   2 ftp      ftp          4096 Jan  6  2019 download
|_drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 upload
22/tcp  open  ssh         Dropbear sshd 0.34 (protocol 2.0)
25/tcp  open  smtp        Postfix smtpd
|_smtp-commands: JOY.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=JOY
| Subject Alternative Name: DNS:JOY
| Issuer: commonName=JOY
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-12-23T14:29:24
| Not valid after:  2028-12-20T14:29:24
| MD5:   9a80 5234 0ef3 1fdd 8f77 16fe 09ee 5b7b
|_SHA-1: 4f02 9a1c 1f41 2ec9 c0df 4523 b1f4 a480 25f9 0165
|_ssl-date: TLS randomness does not represent time
80/tcp  open  http        Apache httpd 2.4.25
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2016-07-19 20:03  ossec/
|_
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Index of /
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: PIPELINING CAPA UIDL TOP RESP-CODES STLS AUTH-RESP-CODE SASL
|_ssl-date: TLS randomness does not represent time
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: more IMAP4rev1 have capabilities ENABLE listed post-login Pre-login SASL-IR IDLE ID OK LOGINDISABLEDA0001 LOGIN-REFERRALS LITERAL+ STARTTLS
|_ssl-date: TLS randomness does not represent time
445/tcp open  netbios-ssn Samba smbd 4.5.12-Debian (workgroup: WORKGROUP)
465/tcp open  smtp        Postfix smtpd
|_smtp-commands: JOY.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=JOY
| Subject Alternative Name: DNS:JOY
| Issuer: commonName=JOY
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-12-23T14:29:24
| Not valid after:  2028-12-20T14:29:24
| MD5:   9a80 5234 0ef3 1fdd 8f77 16fe 09ee 5b7b
|_SHA-1: 4f02 9a1c 1f41 2ec9 c0df 4523 b1f4 a480 25f9 0165
|_ssl-date: TLS randomness does not represent time
587/tcp open  smtp        Postfix smtpd
|_smtp-commands: JOY.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=JOY
| Subject Alternative Name: DNS:JOY
| Issuer: commonName=JOY
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-12-23T14:29:24
| Not valid after:  2028-12-20T14:29:24
| MD5:   9a80 5234 0ef3 1fdd 8f77 16fe 09ee 5b7b
|_SHA-1: 4f02 9a1c 1f41 2ec9 c0df 4523 b1f4 a480 25f9 0165
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imaps?
|_ssl-date: TLS randomness does not represent time
995/tcp open  ssl/pop3s?
|_ssl-date: TLS randomness does not represent time

```

21/tcp  open  ftp         ProFTPD 1.2.10
| ftp-anon: Anonymous FTP login allowed (FTP code 230)

使用匿名登录

![image-20200609221903458](assets/JOY.assets/image-20200609221903458.png)

download为空

upload很多文件

![image-20200609221952034](assets/JOY.assets/image-20200609221952034.png)

使用`mget *`下载全部到本地。逐个查看

只有directory有意义

看上去是用户patrick的用户目录

![image-20200609222134860](assets/JOY.assets/image-20200609222134860.png)

根据之前扫描得到的ProFTPD版本可以知道存在免验证读写文件的漏洞

https://www.freebuf.com/column/209238.html

![image-20200609222236595](assets/JOY.assets/image-20200609222236595.png)

![image-20200609221608317](assets/JOY.assets/image-20200609221608317.png)

通过这个漏洞我们可以使用telnet登录ftp读写patrick的用户目录中的文件

![image-20200609222644097](assets/JOY.assets/image-20200609222644097.png)

然后我们可以将任意文件通过`SITE CPFR`和`SITE CPTO`传输到ftp目录进行下载。这样就可以读取到文件内容



![image-20200609222545177](assets/JOY.assets/image-20200609222545177.png)

![image-20200609223031884](assets/JOY.assets/image-20200609223031884.png)

![image-20200609223144695](assets/JOY.assets/image-20200609223144695.png)

![image-20200609223250955](assets/JOY.assets/image-20200609223250955.png)

通过查看得到了一些新的信息

![image-20200609223402590](assets/JOY.assets/image-20200609223402590.png)

```bash
searchsploit -m 36803
```

![image-20200609223655652](assets/JOY.assets/image-20200609223655652.png)

备注：如果环境是python3,而脚本是python2的，可以先使用2to3转换

![image-20200609225630339](assets/JOY.assets/image-20200609225630339.png)

```
python 36803.py 192.168.43.142 /var/www/tryingharderisjoy whoami
```

多次调试后无法成功，msf也能利用，这里不谈

直接手动

再次利用之前的ftp漏洞。将php-reverse-shell.php上传到ftp服务器，并将其写入www目录，从而得到shell。

将kali本地的php rshell利用ftp匿名可写的设置上传到ftp的upload目录

![image-20200610000656087](assets/JOY.assets/image-20200610000656087.png)

利用已经知道的漏洞，和之前得到的http目录的信息，将rshell文件复制到网页服务目录下

![image-20200610000921663](assets/JOY.assets/image-20200610000921663.png)

监听。浏览器执行。得到shell

![image-20200610001020885](assets/JOY.assets/image-20200610001020885.png)

![image-20200610001150341](assets/JOY.assets/image-20200610001150341.png)

做一些基础的搜索

![image-20200610001804739](assets/JOY.assets/image-20200610001804739.png)

`admin:$apr1$3Jv2Ok6H$4BMdXenVBmD2E3kXe8RVL.`

![image-20200610001928725](assets/JOY.assets/image-20200610001928725.png)

```bash
patrick:apollo098765
root:howtheheckdoiknowwhattherootpasswordis
```

发现明文密钥，尝试切换用户

![image-20200610002112757](assets/JOY.assets/image-20200610002112757.png)

得到patrick用户凭证

![image-20200610002249574](assets/JOY.assets/image-20200610002249574.png)

看起来直接一波了，但是没有权限继续操作。

![image-20200610002409487](assets/JOY.assets/image-20200610002409487.png)

![image-20200610002817381](assets/JOY.assets/image-20200610002817381.png)

但是我们已知目录，并且掌握可以任意替换文件的漏洞。

新建一个test文件

![image-20200610003714115](assets/JOY.assets/image-20200610003714115.png)

再次利用漏洞，替换掉我们无法执行的文件

![image-20200610003801739](assets/JOY.assets/image-20200610003801739.png)

利用nopasswd

![image-20200610003857064](assets/JOY.assets/image-20200610003857064.png)