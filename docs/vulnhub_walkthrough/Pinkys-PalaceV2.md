# Pinkys-PalaceV2

![image-20200715212232042](assets/Pinkys-PalaceV2.assets/image-20200715212232042.png)

![image-20200715212419777](assets/Pinkys-PalaceV2.assets/image-20200715212419777.png)

![image-20200715214055500](assets/Pinkys-PalaceV2.assets/image-20200715214055500.png)

![image-20200715214138941](assets/Pinkys-PalaceV2.assets/image-20200715214138941.png)

显然31337是代理

![image-20200715214330412](assets/Pinkys-PalaceV2.assets/image-20200715214330412.png)

加入代理，重新浏览127.0.0.1:8080

因为已经代理了，所以ip应该使用127.0.0.1

![image-20200715214445176](assets/Pinkys-PalaceV2.assets/image-20200715214445176.png)

现在可以正常浏览了，就可以利用代理对目标网址进行测试

目录

```bash
dirb http://127.0.0.1:8080/ /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -p http://192.168.43.37:31337/
```



```bash

```





![image-20200715222744993](assets/Pinkys-PalaceV2.assets/image-20200715222744993.png)