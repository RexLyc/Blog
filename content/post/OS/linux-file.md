---
title: "边学边用linux-文件系统"
date: 2022-04-02T16:53:25+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg
---
文件系统支撑着操作系统的存储，外部可移动设备的读写。本文叙述安装系统时的一些术语，基本原理，目前常用的文件系统类型。
<!--more-->
## Linux的文件管理机制
- 术语：
    - inode
- 权限：
    - 用户和组：Linux的每一个用户都属于一个用户组
## 安装时の烦恼
- 术语：
    - 主分区
    - 逻辑分区
    - 扩展分区

## 分区表
1. BIOS-MBR
1. UEFI-GPT

## 文件系统
1. FAT32
1. exFAT
1. NTFS
1. EXT系列
    1. EXT2
    1. EXT3
    1. EXT4

## 目录内容约定
1. /opt：optional，安装额外软件的目录
1. /etc：系统管理所需的配置文件
1. /bin：一般是/usr/bin的链接
1. /usr：unix shared resourcecs，存放很多程序和文件，类似于Windows的Program Files
1. /run：存储系统开机以来的信息，该信息在下次启动后会被删掉
1. /sbin：系统管理员使用的程序，一般是/usr/sbin的链接
1. /tmp：临时文件
1. /var：variable，存放会变化的文件，如日志
1. /dev：外部设备
1. /mnt：用于挂载其他文件系统（如光驱、u盘）
1. /home、/root：普通用户主目录、root主目录
1. /lib：动态链接库
1. /proc：系统内存的部分映射