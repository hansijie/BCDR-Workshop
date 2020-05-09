---
title: "备份数据到Stroage Gateway上"
chapter: false
weight: 32
---

## 本地数据中心使用AWS Storage Gateway作为NAS存储，进行文件备份

我们首先来了解一下本地数据中心里使用文件存储网关的调用流程。用户的应用服务器运行在本地数据中心内，在用户环境中部署文件存储网关（File Gateway）。
然后用户或应用服务器通过NFS客户端连接存储网关，利用网关将 S3 中的存储桶作为 NFS 云端存储，从而对文件进行写入和访问。
您可以在本地将 File Gateway 作为虚拟机 (VM) 设备运行，目前支持的虚拟化技术有VMware ESXi、Microsoft Hyper-V与Linux 内核的虚拟机 (KVM)。
更多详情可参考：https://docs.aws.amazon.com/zh_cn/storagegateway/latest/userguide/Requirements.html 与 http://docs.aws.amazon.com/zh_cn/storagegateway/latest/userguide/deploy-gateway-vmware.html

本次实验采用AWS EC2 实例 – 创建运行File Gateway 文件存储网关，期原理与在虚拟机 (VM) 设备中运行相同，只是创建细节稍有不同。具体操作过程如下：
1.创建Storage Gateway
打开https://console.amazonaws.cn/storagegateway/create/gateway?region=cn-north-1创建Storage Gateway, 选择” File Gateway”文件存储网关类型。
需要注意的是，本次实验将在cn-north-1 Beijing 区域创建文件存储网关，因为我们的本地数据中心部署在北京region。
![](/images/SetupStorageGW/createStorageGW.png)

2.选择Amazon EC2作为主机平台，按照设置说明配置VPC，主机类型，添加新EBS卷，安全组等。
注意：
* 如要求，EC2配置要求大于或等于xlarge，即4核/16G内存。建议使用m5.xlarge。
![](/images/SetupStorageGW/createStorageGW-EC2.png)
![](/images/SetupStorageGW/createStorageGW-EC2Type.png)
* 需要额外添加EBS卷，其容量设置不应小于150G；（存储网关至少需要添加一个磁盘以便进行缓存，客户端对文件的访问会首先在缓存中去获取文件信息，这样将大大减少通过网络去S3直接获取文件的延迟，文件存储网关缓存最大值为16TB）
![](/images/SetupStorageGW/createStorageGW-EBS.png)
* 在安全组上临时配置安全策略，允许VPC内其它机器通过80端口访问该机器（临时开放http 80端口）
![](/images/SetupStorageGW/createStorageGW-SG.png)
* 保证该EC2实例能够访问公网

3.获得Gateway私有IP
等待EC2创建完成，点击https://console.amazonaws.cn/ec2/v2/home?region=cn-north-1进入EC2详情页，记录下该机器的私有IP （下文称为 Gateway私有IP）
![](/images/SetupStorageGW/createStorageGW-IP.png)

4.链接并激活网关
SSH登录到跳板机EC2实例后，运行以下命令得到File Gateway实例的激活码：
```bash
curl -I [gateway私有IP]/?activationRegion=cn-north-1
```
记录下激活码activationKey的值
![](/images/SetupStorageGW/activateStorageGW.png)

5.运行以下命令激活File Gateway
```bash
aws --region cn-north-1 storagegateway activate-gateway --activation-key [activationKey] --gateway-type FILE_S3 --gateway-name [gateway name] --gateway-timezone GMT+8:00 --gateway-region cn-north-1
```
激活成功后命令返回GatewayARN
![](/images/SetupStorageGW/gatewayARN.png)

6.配置File Gateway缓存
打开https://console.amazonaws.cn/storagegateway/home/gateways?region=cn-north-1，检查File Gateway是否已经建立（Offine状态则需要等一会）。
点击File Gateway，编辑本地磁盘，将创建时添加的EBS卷配置成缓存，并且保存。
![](/images/SetupStorageGW/gatewayCache1.png)
![](/images/SetupStorageGW/gatewayCache2.png)

7.创建文件共享，关联S3 bucket和与File Gateway
打开https://console.amazonaws.cn/storagegateway/create/file-share?region=cn-north-1 后创建文件共享， S3 bucket为准备阶段创建的S3存储桶 “storagegateway-coldbackup-20200101xx”，网关选择” sg02_onprem-bjs-az1-dmz”，共享协议为NFS。
![](/images/SetupStorageGW/S3AndGateway1.png)
![](/images/SetupStorageGW/S3AndGateway2.png)

8.在EC2上使用file gateway上的NFS
登录已经创建的临时EC2实例。我们将在该机器上测试NFS的配置，运行以下命令：
```bash
sudo mount -t nfs -o nolock [gateway私有IP]:/[s3存储桶名] [MountPath]
```
![](/images/SetupStorageGW/mountFS.png)

9.成功mount后，您就可以在/nfs文件夹看到s3存储桶中的内容了。您可以添加,删除,或者查看这些内容。
```bash
cd [MountPath]
echo "hello, one" > hello1.txt 
echo "hello, two" > hello2.txt
echo "hello, three" > hello3.txt
```
![](/images/SetupStorageGW/backupFileToGW.png)

登录 AWS 管理控制台，并通过网址https://console.amazonaws.cn/s3/home?region=cn-north-1打开 Amazon S3 控制台，点击” storagegateway-coldbackup-20200101xx”存储桶。
![](/images/SetupStorageGW/verifyBackup.png)

{{% notice note %}}
注意： 您在/nfs文件夹下的任何操作会直接映射到File Gateway缓存上，为了提升效率，缓存会定期和S3进行同步工作。所以，当您添加了一个文件在/nfs中，可能需要过几秒在S3存储桶中看到它。
{{% /notice  %}}
