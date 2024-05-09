---
title: "各平台工程问题实录"
date: 2024-03-25T15:07:36+08:00
categories:
- 计算机
- 工程部署
tags:
- 部署
- 工程
# thumbnailImagePosition: left
# thumbnailImage: //example.com/image.jpg
---

<!--more-->
## 网络
1. windows下作为服务器时，可能遇到的无法通信的网络情况
    1. 未开放防火墙权限。需要修改，允许应用通过防火墙。
    2. 允许通过防火墙后，仍然无法访问，运行```secpol.msc```，可能需要专门放行某些ip端口策略。
1. 通过windows，为网络之间提供共享（网桥）
    1. 选择打算分享的网络A，并在其适配器设置中，选择共享，共享到目标网络B。这里的共享是指，目标网络B上的主机，可通过本Windows提供的网桥功能，访问到网络A。不要配反了。
    1. 参考[用Windows通过网线共享网络给其他电脑（Windows、Ubuntu）](https://blog.csdn.net/iamjingong/article/details/119379129)
1. NFS配置（Linux）
    1. 在跨设备进行开发时很有用的文件共享，其配置方式的主要流程是：安装nfs-kernel-server、rpcbind。并修改/etc/exports文件，再用```exportfs -a```导出，用systemctl重启nfs-server和rpcbind。
    2. 文件示例
        ```bash
        # 如果不需要控制ip，就用*，否则这里是ip，例如192.168.1.*，后面是一些具体配置，根据需要修改
        /path/to/your/share/directory *(rw,sync,insecure,no_subtree_check)
        ```
    3. 注意在mount的时候有一些说法，如果网络不好，建议使用nolock，避免出现connection refused
        ```bash
        # 不添加nolock，对网络稳定性有一些要求
        mount -t nfs -o nolock 192.168.1.1:/path/to/your/share/directory /mnt/directory
        ```