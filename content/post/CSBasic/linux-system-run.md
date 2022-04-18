---
title: "边学边用linux-系统运行篇"
date: 2022-02-22T21:47:16+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg
---
了解系统运行是更好的使用Linux的基础。本文讲述Linux的系统运行原理。
<!--more-->
## 启动
1. BIOS
1. GRUB
1. 内核启动
1. 服务启动：
    - systemctl是systemd风格的服务启动方式。用于替换init风格的服务启动方式。
1. 参考
    - [init systemd](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
    - [init service systemctl区别](https://blog.csdn.net/lineuman/article/details/52578399)