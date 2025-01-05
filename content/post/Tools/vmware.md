---
title: "VMWare实用知识"
date: 2021-08-20T15:16:18+08:00
categories:
- 实用工具
- VMWare
tags:
- VMWare
- 实用工具
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/VMware.jpg
---
VMWare作为Windows上最常用的虚拟机。是跨平台开发必不可少的帮手。本文记录一些实用的VMWare知识
<!--more-->

## vmware tools
在workstation较高版本之后，vmware tools只提供windows虚拟机实用了。如果安装ubuntu虚拟机。请实用open-vm-tools。
```bash
sudo apt update
sudo apt install open-vm-tools
```

## 共享文件夹
1. 需要虚拟机内系统支持该特性（VMWare Tools）
2. 设置-选项-共享文件夹-选择主机内指定目录
3. 在虚拟机内/mnt/hgfs/下寻找共享目录即可
## 故障及解决方案
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