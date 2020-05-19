---
title: "从Stroage Gateway上恢复数据"
chapter: false
weight: 73
---

## 对AWS Storage Gateway备份的文件进行恢复

使用AWS Storage Gateway作为NAS存储，恢复文件的过程与备份类似，把文件网关挂载到目标服务器上即可。具体实验过程如下：

1.登录到跳板机，并打开xshell，双击"remote-aws-env"，从而登录到容灾端的WordPress APP服务器里，并运行以下命令，其中的"/mnt"为本地的挂载点。
```bash
sudo mount -t nfs -o nolock [gateway私有IP]:/[s3存储桶名] /mnt
ls /mnt
```

即可看到在/mnt目录下，您在本地数据中心环境里输入的3个文件了。
![](/images/SetupStorageGW/restoreFromStorageGW.png)

