---
title: "对本地数据中心的应用系统进行初始化"
chapter: false
weight: 22
---

在本实验中，我们会通过登录堡垒机，对本地数据中心的应用系统进行初始化。具体步骤如下：

1.在EC2的控制台上找到该Cloudformation创建的EC2实例，包括一台堡垒机，和一台Wordpress服务器（其中，Wordpress服务器里同时部署了数据库和应用系统）：
https://console.amazonaws.cn/ec2/v2/home?region=cn-north-1#Instances:tag:Name=Basion,APP;sort=launchTime

选中Wordpress服务器，并记录下该服务器的私有IP地址。
![](/images/CreateSourceEnv/getIDCWPIP.png)

2.选中堡垒机，并记录下该服务器的公网IP地址：
![](/images/CreateSourceEnv/getBastionIP.png)

在【操作】下拉菜单中，选择"获取Windows密码"菜单，如果遇到下面窗口，则等待5分钟以后再试。
![](/images/CreateSourceEnv/waitFourMins.png)

点击【Browse...】按钮，把之前下载的密钥pem文件打开，再点击【解密密码】，从而获取登录Windows的密码。
![](/images/CreateSourceEnv/getBastionPassword1.png)
![](/images/CreateSourceEnv/getBastionPassword2.png)

点击复制图标，把该密码拷贝下来
![](/images/CreateSourceEnv/getBastionPassword3.png)

打开本地Windows的remote desktop，输入堡垒机的公网ip地址，用户名输入administrator，回车进行登录。
![](/images/CreateSourceEnv/remotedesktop.png)
在提示密码的弹出窗口中，输入上一步记录的密码，回车进行确认。如果有弹出安全证书的告警信息，则点击【Yes】按钮即可。

3.登录到堡垒机以后，会多次看到下面的提示，点击【OK】按钮，忽略即可。
![](/images/CreateSourceEnv/ignoreMySQLError1.png)

打开桌面的WorkshopTools文件夹，找到并双击"MySQL Workbench 8.0 CE"。
在MySQL Workbench的界面上找到并右键点击"local-idc-env"图标，在弹出菜单中选择"Edit Connection..."菜单选项。
![](/images/CreateSourceEnv/editDBConnection1.png)

在Hostname字段里输入第一步中记录的Wordpress服务器的私有IP地址

点击"Store in Vault..."按钮，在弹出的窗口的"Password"栏位中输入：Initial-1
如果发现弹出如下报错窗口，则点击【Ignore】按钮忽略即可。
![](/images/CreateSourceEnv/ignoreMySQLError2.png)

点击【Test Connection】按钮确认连接成功以后，点击【Close】按钮退出。
![](/images/CreateSourceEnv/editDBConnection2.png)

双击"local-idc-env"，然后在Query 1里输入下面的SQL，注意把下面的"Wordpress服务器的私有IP地址"改为你在第1步中记录的私有IP地址。
```bash
UPDATE wp_options SET option_value = REPLACE(option_value, 'localhost', 'Wordpress服务器的私有IP地址') WHERE option_name = 'home' OR option_name = 'siteurl';
UPDATE wp_posts SET post_content = REPLACE(post_content, 'localhost', 'Wordpress服务器的私有IP地址');
UPDATE wp_posts SET guid = REPLACE(guid, 'localhost', 'Wordpress服务器的私有IP地址');
```

然后在Query下拉菜单里选择"Execute (All or Selection)"选项，执行这3条SQL语句：
![](/images/CreateSourceEnv/updateMetadata1.png)

执行后的结果如下：
![](/images/CreateSourceEnv/updateMetadata2.png)

4.在桌面的WorkshopTools文件夹，找到并双击"Google Chrome"，在浏览器地址栏中输入：Wordpress服务器的私有IP地址/wordpress，从而打开Wordpress应用。







 

