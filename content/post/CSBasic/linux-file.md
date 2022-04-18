---
title: "边学边用linux-文件系统"
date: 2022-04-02T16:53:25+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 开坑篇
- 施工中
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