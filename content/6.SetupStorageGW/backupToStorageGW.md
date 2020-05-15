---
title: "备份数据到Stroage Gateway上"
chapter: false
weight: 62
---

## 本地数据中心使用AWS Storage Gateway作为NAS存储，进行文件备份

我们首先来了解一下本地数据中心里使用文件存储网关的调用流程。用户的应用服务器运行在本地数据中心内，在用户环境中部署文件存储网关（File Gateway）。
然后用户或应用服务器通过NFS客户端连接存储网关，利用网关将 S3 中的存储桶作为 NFS 云端存储，从而对文件进行写入和访问。
您可以在本地将 File Gateway 作为虚拟机 (VM) 设备运行，目前支持的虚拟化技术有VMware ESXi、Microsoft Hyper-V与Linux 内核的虚拟机 (KVM)。
更多详情可参考：https://docs.aws.amazon.com/zh_cn/storagegateway/latest/userguide/Requirements.html 与 http://docs.aws.amazon.com/zh_cn/storagegateway/latest/userguide/deploy-gateway-vmware.html

本次实验采用AWS EC2 实例 – 创建运行File Gateway 文件存储网关，期原理与在虚拟机 (VM) 设备中运行相同，只是创建细节稍有不同。具体操作过程如下：
1.创建Storage Gateway
打开https://console.amazonaws.cn/storagegateway/create/gateway?region=cn-north-1

创建Storage Gateway, 选择"文件网关"作为存储网关类型。点击【下一步】。
需要注意的是，本次实验将在cn-north-1 北京区域创建文件存储网关，因为我们的本地数据中心部署在北京region。
![](/images/SetupStorageGW/createStorageGW.png)

2.选择Amazon EC2作为主机平台
![](/images/SetupStorageGW/createStorageGW-EC2.png)

并点击【启动实例】按钮。在弹出的界面上，按照设置说明配置VPC，主机类型，添加新EBS卷，安全组等。
注意：
* 如要求，EC2配置要求大于或等于xlarge，即4核/16G内存。建议使用m5.xlarge。
![](/images/SetupStorageGW/createStorageGW-EC2Type.png)

* 在"步骤 3: 配置实例详细信息"中，"网络"一栏选择名称为"LocalVPC"的子网，"自动分配公有IP"下拉列表里选择"启用"。点击【下一步:添加存储】按钮。
![](/images/SetupStorageGW/createStorageGW-VPC.png)

* 需要额外添加EBS卷，其容量设置不应小于150G；（存储网关至少需要添加一个磁盘以便进行缓存，客户端对文件的访问会首先在缓存中去获取文件信息，这样将大大减少通过网络去S3直接获取文件的延迟，文件存储网关缓存最大值为16TB）。点击【下一步:添加标签】按钮。
![](/images/SetupStorageGW/createStorageGW-EBS.png)

* 在"步骤 5: 添加标签"处，选择【添加其他标签】按钮，并在"键"列输入"Name"，在"值"列输入"Filegateway"。
![](/images/SetupStorageGW/createStorageGW-Tag.png)

* 在安全组上临时配置安全策略，允许VPC内其它机器通过80端口访问该机器（临时开放http 80端口）。点击【审核和启动】按钮。
![](/images/SetupStorageGW/createStorageGW-SG.png)

* 点击【启动】按钮。在弹出的窗口中，在"选择一个m密钥对"下拉列表里，选择"local-idc-key"。并选中"我确认我有权访问所选的私有密钥文件(local-idc-key.pem)，并且如果没有此文件，我将无法登录我的实例。"复选框。点击【启动实例】按钮。
![](/images/SetupStorageGW/createStorageGW-LaunchEC2.png)

3.获得Gateway私有IP。进入EC2的控制台：https://console.amazonaws.cn/ec2/v2/home?region=cn-north-1#Instances:search=Filegateway;sort=launchTime
等待EC2创建完成，进入EC2详情页，记录下该机器的公网IP （下文称为 Gateway公网IP）以及私有IP（下文称为 Gateway私网IP）
![](/images/SetupStorageGW/createStorageGW-IP.png)

4. 回到"创建网关"界面，点击【下一步】，"终端节点类型"选择"公有"，点击【下一步】。
![](/images/SetupStorageGW/createStorageGW-endpoint.png)

5.在"IP地址"输入第3步记录的公网IP地址，点击【连接到网关】按钮。

6.在"网关名称"一栏输入：backup-filegw，然后点击【激活网关】按钮。
![](/images/SetupStorageGW/createStorageGW-activate.png)

7.在"配置本地磁盘"页面上，等待网关激活，然后点击【配置日志记录】按钮。
![](/images/SetupStorageGW/createStorageGW-localdisk.png)

8.在"网关运行状况日志组"页面上，选择"创建新的日志组"，点击【保存并继续】按钮。
![](/images/SetupStorageGW/createStorageGW-loggroup.png)
![](/images/SetupStorageGW/createStorageGW-finish.png)

9.创建文件共享，关联S3 bucket和与File Gateway。打开stroage gateway控制台：https://console.amazonaws.cn/storagegateway/create/file-share?region=cn-north-1 

创建文件共享， S3 bucket为准备阶段创建的S3存储桶 "storagegateway-coldbackup-xxx"，网关选择"backup-filegw"，共享协议为NFS。
![](/images/SetupStorageGW/S3AndGateway1.png)
点击【下一步】，在"配置文件在 Amazon S3 中的存储方式"页面上，保留缺省值，点击【下一步】。在"审核"页面上，点击【创建文件共享】按钮。

10.在服务器里挂载file gateway。

登录到跳板机，并打开xshell，双击"local-idc-env"，从而登录到WordPress APP服务器里，并运行以下命令，其中的"/mnt"为本地的挂载点。
```bash
sudo mount -t nfs -o nolock [gateway私有IP]:/[s3存储桶名] /mnt
```
![](/images/SetupStorageGW/mountFS.png)

10.成功mount后，您就可以在/nfs文件夹看到s3存储桶中的内容了。您可以添加,删除,或者查看这些内容。
```bash
cd [MountPath]
echo "hello, one" > hello1.txt 
echo "hello, two" > hello2.txt
echo "hello, three" > hello3.txt
```
![](/images/SetupStorageGW/backupFileToGW.png)

打开 Amazon S3 控制台：https://console.amazonaws.cn/s3/home?region=cn-north-1#

点击"storagegateway-coldbackup-xxx"存储桶，可以看到我们刚才创建的3个文件。
![](/images/SetupStorageGW/verifyBackup.png)

{{% notice note %}}
注意： 您在/nfs文件夹下的任何操作会直接映射到File Gateway缓存上，为了提升效率，缓存会定期和S3进行同步工作。所以，当您添加了一个文件在/nfs中，可能需要过几秒在S3存储桶中看到它。
{{% /notice  %}}
