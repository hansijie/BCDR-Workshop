---
title: "把本地数据中心的网络和容灾环境的网络打通"
chapter: false
weight: 32
---

## 把本地数据中心的网络和容灾环境的网络打通

真实的容灾环境中，通常会使用专线或者VPN的方式，把本地数据中心和云端的网络进行打通。
本次实验中，因为是通过北京region来模拟本地数据中心的环境，因此可以通过跨region的VPC peering来打通本地数据中心与宁夏region的容灾环境。

配置过程如下：

1.进入北京region的VPC界面：https://console.amazonaws.cn/vpc/home?region=cn-north-1#vpcs:sort=VpcId

找到名称为为：LocalVPC的VPC，并记录该VPC的ID。
![](/images/LandingZoneOfDRSite/idcVPCID.png)

2.进入宁夏region的VPC界面，在左边选择"对等连接"菜单，然后在右边点击【创建对等连接】按钮。
或者点击该链接进入创建对等连接的页面：https://cn-northwest-1.console.amazonaws.cn/vpc/home?region=cn-northwest-1#CreatePeeringConnection:

* 在"对等连接名称标签"中输入：dr-connection

* 在"VPC (请求方)"下拉列表中选择名为BCDRVPC的VPC。

* 在"选择要用作对等的另一个VPC"部分中：

  * "账号"部分保留缺省值
  
  * "区域"部分选择"另一个区域"，并在下拉列表中选择"中国(北京)(cn-north-1)"。
  
  * "VPC (接受方)"里填入你在第一步记录下的VPC ID。

点击【创建对等连接】按钮。

![](/images/LandingZoneOfDRSite/CreateVPCPeering1.png)
![](/images/LandingZoneOfDRSite/CreateVPCPeering2.png)

3.到北京region的对等连接界面：https://console.amazonaws.cn/vpc/home?region=cn-north-1#PeeringConnections:sort=vpcPeeringConnectionId

选中状态为"正在处理接受"的VPC peering，然后在"操作"下拉菜单里，选择"接受请求"选项。
![](/images/LandingZoneOfDRSite/CreateVPCPeering3.png)
在弹出的确认界面中，选择【是，接受】按钮。
回到宁夏region的对等连接界面：https://cn-northwest-1.console.amazonaws.cn/vpc/home?region=cn-northwest-1#PeeringConnections:sort=vpcPeeringConnectionId

确认名为"dr-connection"的对等连接的状态为"活动"。

4.到北京region的EC2界面：https://console.amazonaws.cn/ec2/v2/home?region=cn-north-1#Instances:tag:Name=Basion,APP;sort=launchTime

选择名称为"WordPress APP"的EC2，找到并点击其所在的子网ID的链接，进入VCP的子网界面。
![](/images/LandingZoneOfDRSite/wpserversubnet.png)

5.在VPC的子网界面上，选择"路由表"tab页，并点击路由表链接，进入路由表界面。
![](/images/LandingZoneOfDRSite/idcroutetable.png)

6.在路由表界面上，选择"路由"tab"页。并选择【编辑路由】按钮。
![](/images/LandingZoneOfDRSite/editroute.png)

7.在"编辑路由"界面上，点击【添加路由】按钮：

  * 在第一列"目标"输入：192.168.0.0/24
  
  * 在第二列"目标"下拉列表里选择"Peering Connection"，并选择我们刚才创建对等连接。
  
  * 最后，点击【保存路由】

![](/images/LandingZoneOfDRSite/addrouteentry.png)
  
8.到宁夏region的VPC界面上：https://cn-northwest-1.console.amazonaws.cn/vpc/home?region=cn-northwest-1#vpcs:sort=VpcId

找到并拷贝名称为BCDRVPC的VPC的ID。
![](/images/LandingZoneOfDRSite/getBCDRVPCID.png)

9.点击左侧"路由表"菜单，并在右侧的过滤栏里，输入上一步的VPC ID。找出属于该VPC的路由表ID。并选中"显式关联对象"一栏为"2个子网"的那个路由表。
![](/images/LandingZoneOfDRSite/selectDRSiteRouteTable.png)

10.选中关联了子网的路由表以后，点击"路由" tab 页，然后点击【编辑路由】按钮，进入路由编辑界面。在"编辑路由"界面上，点击【添加路由】按钮：

  * 在第一列"目标"输入：10.0.0.0/24
  
  * 在第二列"目标"下拉列表里选择"Peering Connection"，并选择我们刚才创建对等连接。
  
  * 最后，点击【保存路由】
  
