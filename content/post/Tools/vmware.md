---
title: "实用虚拟化技术"
date: 2021-08-20T15:16:18+08:00
categories:
- 实用工具
tags:
- 实用工具
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/VMware.jpg
---
虚拟化和虚拟机是开发中常见的两个概念。本文重新整理之前的学习内容。
<!--more-->
## 什么时候在用
VMWare在大多数情况下是够用的。尤其是提供了良好的GUI。

但是如果需要跨体系架构模拟，比如x86上运行arm。则需要Qemu。此时Qemu作为第二类虚拟机（指通过软件模拟），来提供虚拟机的运行环境。

这里的KVM则是指Linux提供的KVM模块，为Linux引入了类似第一类虚拟机的能力（指令直通硬件）。

而纯粹的第一类虚拟机则是直接运行Hypervisor，上面可以执行不同系统。这类技术例如VMWare ESXi、Xen (独立)、Microsoft Hyper-V Server等。他们提供操作系统级别的隔离，负责调度虚拟机在cpu、网络设备上的使用。


![现代操作系统](/images/tools/vm-archs.png)

## Qemu
推荐直接用fedora，官方提供来qcow2格式的server镜像。可以直接运行。

如果你在windows上运行，其命令类似于。

```sh
./qemu-system-x86_64.exe -accel whpx -m 2048  -drive file="E:\VMs\qemu\Fedora-Server-Guest-Generic-43-1.6.x86_64.qcow2",format=qcow2 -netdev user,id=net0 -device virtio-net-pci,netdev=net0 -boot c
```


## KVM
暂时还没用过


## VMWare

### 高版本的坑
1. vmware tools

在workstation较高版本之后，vmware tools只提供windows虚拟机使用了。如果安装ubuntu虚拟机。请实用open-vm-tools。
```bash
sudo apt update
sudo apt install open-vm-tools
```

### 共享文件夹
1. 需要虚拟机内系统支持该特性（VMWare Tools）
2. 设置-选项-共享文件夹-选择主机内指定目录
3. 在虚拟机内/mnt/hgfs/下寻找共享目录即可
### 故障及解决方案
- 虚拟机上不去网
    - 情况1：Ubuntu18.04 Desktop，配置桥接模式，ifconfig只有lo。有时虚拟机无法正常启动、关闭，强制关闭后开机会无法上网。
        - Ubuntu Desktop版本用的网络管理是NetworkManager，NetworkManager运行问题，尝试以下代码。
        ```bash
        sudo service NetworkManager stop
        sudo rm /var/lib/NetworkManager/NetworkManager.state #删之前可以看一下里面enable应该是false
        sudo service NetworkManager start
        ```
        - [参考CSDN](https://blog.csdn.net/leadingsci/article/details/80873542)
    - 情况2：虚拟机开机后使用NAT模式，但是ens33未启动
        - 修改/etc/network/interfaces文件
        ```bash
        sudo echo 'auto ens33' > /etc/network/interfaces
        sudo echo 'iface ens33 inet dhcp' > /net/network/interfaces
        sudo systemctl restart networking
        ```


## 参考文档
1. [BUAA-OS实验文档](https://wokron.github.io/posts/qemu-introduction/)