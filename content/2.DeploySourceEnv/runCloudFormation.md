---
title: "运行CloudFormation脚本从而部署本地应用系统"
chapter: false
weight: 21
---

在本实验中，我们会通过运行CloudFormation脚本，部署本地数据中心的应用系统。具体步骤如下：

1.下载CloudFormation脚本到本地电脑。
{{%attachments title="下载链接:" /%}}

2.打开北京region的CloudFormation console：https://console.amazonaws.cn/cloudformation/home?region=cn-north-1#/stacks/create/template
在模板源里选择"上传模板文件"
在【选择文件】按钮里，选择在之前下载的模板文件：workshop_local.yaml。
![](/images/CreateSourceEnv/createStack1.png)

3.在堆栈详细信息部分：

* 堆栈名称输入：local-idc-env

* InstanceType输入：c5.large

* KeyName选择之前创建的key：local-idc-key

其他保留缺省值，选择【下一步】按钮。
![](/images/CreateSourceEnv/createStack2.png)

4.在"配置堆栈选项"页面上，保留缺省值，选择【下一步】

5.在第四步"审核"中，选择【创建堆栈】按钮，开始部署CloudFormation stack，等待该过程结束。

