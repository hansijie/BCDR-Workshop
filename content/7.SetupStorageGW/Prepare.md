---
title: "准备工作"
chapter: false
weight: 71
---

## 配置Storage Gateway之前的准备工作

1.创建一个S3 Bucket
登录 AWS 管理控制台，并通过网址https://console.amazonaws.cn/s3/bucket/create?region=cn-north-1

打开 Amazon S3 控制台，并开始创建一个新的 S3存储桶 
Create bucket (创建存储桶)，在 Bucket name 字段中，为新存储桶键入一个符合 DNS 标准的唯一名称，此处名称为"storagegateway-coldbackup-xx"，"xx"为你的姓名拼音，以确保该bucket的名称唯一。
对于 Region，选择作为要将存储桶放置到的区域，选择"中国(北京) cn-north-1"，其他保留缺省值，然后点击【创建存储桶】按钮。
![](/images/SetupStorageGW/createS3Bucekt.png)


