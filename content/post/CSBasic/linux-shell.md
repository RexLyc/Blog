---
title: "边学边用linux-命令行篇"
date: 2022-02-10T14:54:43+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 命令行
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg
---
需要熟练掌握命令行是Linux系列系统最大的特点之一。本文从实用角度出发编写。
<!--more-->
# 核心指令
1. vi/vim
1. awk
1. sed
1. grep
1. find
# 其他常用指令
1. 网络工具
    ```bash
    # 嗅探指定端口
    nmap ip -p port
    # 嗅探全部端口
    nmap ip
    ```
# Shell编程
1. 基本概念
    1. 标准输入输出
    1. 重定向
    1. 管道
    1. 前后台
    1. 系统变量
1. 核心语法
1. 经典例子
1. Shell环境
    - .bashrc
    - .bash_profile
1. 启动环境
    - init.d
    - rcX.d
# 其他指令收集
- 查看系统发行版：
```bash
# Red Hat系 & Ubuntu系
cat /etc/os-release
# Ubuntu系
cat /etc/issue
```