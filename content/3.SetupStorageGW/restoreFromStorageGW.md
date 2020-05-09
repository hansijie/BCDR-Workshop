---
title: "从Stroage Gateway上恢复数据"
chapter: false
weight: 33
---

## 从AWS Storage Gateway进行文件恢复

使用AWS Storage Gateway作为NAS存储，恢复文件的过程与备份类似，都是需要先创建文件网关并激活。然后创建文件共享，将文件网关与完成备份的S3 bucket进行关联。

{{% notice note %}}
唯一不同的是，需要对文件网关显示调用“刷新缓存”，才能完整的获取Amazon S3 存储桶中的对象。
{{% /notice  %}}

* 因为在 NFS客户端执行文件系统操作时，您的网关会在与文件共享关联的 Amazon S3 存储桶中维护一个对象清单，网关使用此缓存清单来减小 S3 请求的延迟和频率。
* 一个新的文件网关关联到Amazon S3 存储桶，需要通过显示刷新以完成对象清单的更新，从而能在NFS客户端查询到完整的文件清单。
* 刷新过程所需的时间取决于在网关上缓存的对象数以及在 S3 存储桶中添加或删除的对象数。

要刷新用于文件共享的 S3 存储桶，您可以使用 AWS Storage Gateway 控制台或 AWS Storage Gateway API 中的 RefreshCache 操作。具体操作过程如下：
1.创建新的文件网关，关联S3 bucket存储桶后，发现 NFS客户端 无法查询到S3 bucket存储桶已有的文件，如下图。
![](/images/SetupStorageGW/restoreFromStorageGW1.png)
![](/images/SetupStorageGW/restoreFromStorageGW2.png)

2.刷新文件网关从而完整的获取Amazon S3 存储桶中的对象，运行以下命令。
```bash
aws storagegateway list-file-shares #获得file share ARN
aws storagegateway refresh-cache -–file-share-arn <fileshare ARN> #调用RefreshCache
```
通过调用运行RefreshCache命令， NFS客户端成功查询到S3 bucket存储桶已有的文件。
![](/images/SetupStorageGW/listFileShare.png)

