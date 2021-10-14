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
# 共享文件夹
1. 需要虚拟机内系统支持该特性（VMWare Tools）
2. 设置-选项-共享文件夹-选择主机内指定目录
3. 在虚拟机内/mnt/hgfs/下寻找共享目录即可
# 故障及解决方案
- 虚拟机上不去网
    - 情况：Ubuntu18.04 Desktop，配置桥接模式，ifconfig只有lo。有时虚拟机无法正常启动、关闭，强制关闭后开机会无法上网。
    - Ubuntu Desktop版本用的网络管理是NetworkManager，NetworkManager运行问题，尝试以下代码。
    ```bash
    sudo service NetworkManager stop
    sudo rm /var/lib/NetworkManager/NetworkManager.state #删之前可以看一下里面enable应该是false
    sudo service NetworkManager start
    ```
    - [参考CSDN](https://blog.csdn.net/leadingsci/article/details/80873542)