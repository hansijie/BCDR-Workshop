---
title: "创建本地数据中心运行的应用系统"
chapter: false
weight: 20
---

在本实验中，我们会在EC2实例里部署Wordpress应用系统，从而模拟本地数据中心里的业务系统。整个实验过程通过运行CloudFormation脚本完成。具体操作包括：

* 通过运行CloudFormation脚本，部署本地数据中心的应用系统

* 通过登录堡垒机，对本地数据中心的应用系统进行初始化

先进行如下的准备工作：

1.在北京region创建key pair。登录北京region的keypair控制台：https://console.amazonaws.cn/ec2/v2/home?region=cn-north-1#KeyPairs:
点击【Create key pair】
![](/images/CreateSourceEnv/createKeyPair1.png)
在创建key pair界面上，指定"Name"为：local-idc-key，"File Format"为：pem。并点击【Create key pair】。
![](/images/CreateSourceEnv/createKeyPair2.png)

在创建了key pair以后，会弹出下载key文件的窗口，把key pair文件下载到本地，并妥善保存。

{{% notice note %}}
key pair文件只有在第一次创建的时候才能下载，如果没有下载，则必须重新创建一个新的key paire文件。
{{% /notice  %}}

2.创建IAM用户。登录创建IAM用户的控制台：https://console.amazonaws.cn/iam/home?region=cn-north-1#/users$new?step=details

在"设置用户详细信息"页面上：
* 在"用户名"处输入：demouser
* 在"访问类型"处勾选"编程访问"
![](/images/CreateSourceEnv/createUser1.png)

点击【下一步:权限】按钮。

在"设置权限"页面，选择"直接附加现有策略"，并在策略列表里勾选"AdministratorAccess"。
![](/images/CreateSourceEnv/createUser2.png)

点击【下一步:标签】按钮。

在"添加标签 (可选)"页面，保留缺省值，点击【下一步:审核】按钮。
在"审核"页面，保留缺省值，点击【创建用户】按钮。
在用户创建完毕以后，点击【下载.csv】按钮，把该用户的access key和secret key下载下来。

{{% notice note %}}
包含IAM用户的access key和secret key的csv文件只有在第一次创建用户的时候才能下载，如果没有下载，则必须重新创建一个新的IAM用户。
{{% /notice  %}}


