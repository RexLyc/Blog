---
title: "UNIX环境高级编程：其一"
date: 2023-01-19T13:09:30+08:00
categories:
- 读书笔记
- 技术书
tags:
- 暂停施工
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/book/apue.png
---
本节为《UNIX环境高级编程（第三版）》读书笔记第一部分，包含第1到第6章，内容以文件相关内容为主。
<!--more-->
## UNIX基础
### 概述
&emsp;&emsp;操作系统可以理解为是一种控制计算机硬件资源、提供程序运行环境的软件。也可以称为**内核（Kernel）**。内核的接口就是**系统调用**。内核向外依次是：系统调用、公用函数库/Shell、应用程序。
>注：Linux其实只是GNU系统的内核
### 标准
- ISO C：1989年ANSI C被基本确定为初版ISO C，此后ISO C一直致力于将C发展为标准化语言，而且是UNIX无关，即可以在非UNIX系统之间进行移植
- IEEE POSIX：从IEEE标准1003.1-1988（操作系统接口）发展而来，1003.1-1988被提交给ISO，稍加修订被确定为ISO/IEC 9945-1:1990，该标准通常也被称为POSIX.1。此后的更新可以被称为POSIX.1 某某年版。
    - IEEE提交的标准经常被ISO批准作为ISO标准
    - 后续增加的标准有：实时扩展（1993）、pthreads（1995）、高级实时扩展（1999）、跟踪（2000）、shell、系统接口、命令和使用程序、网络服务
- SUS（Single UNIX Specification）：单一UNIX规范，是POSIX.1标准的超集。该规范是Open Group的出版物，Open Group是两个工业社团X/Open和开放系统软件基金会（OSF）在1996年合并而来。
    - Open Group拥有UNIX商标，该组织提出的标准代表的是UNIX的标准要求。合并前的标准名为X/Open，合并后就是SUS了。
    - POSIX.1中的X/Open系统接口选项，描述了一些可选的接口，定义了满足XSI（X/Open System Interfce）要求的系统必须实现的接口，只有遵循XSI的，才可用称之为UNIX系统。
> ISO、IEEE、SUS是三个基本独立的组织，各自出于自己的目的在做标准化

### 实现
- UNIX分时系统v6（1976）、v7（1979）
    - AT&T分支：系统III、系统V（商用化）
    - 加州大学伯克利分校分支：4.xBSD实现
    - AT&T贝尔实验室的计算科学研究中分支：UNIX分时系统v8、v9、v10

## I/O
1. 实用函数：
    - open、read、write、lseek、close：不带缓冲的I/O

## 文件和目录
1. 目录是包含目录项的一种文件
1. 实用函数：
    - opendir、readdir、closedir：打开一个目录、读取目录内容、关闭目录
    - stat、fstat：读取文件属性
    - getcwd/getwd、chdir：查询、修改当前程序的工作目录

## 系统文件
### 用户信息
1. 口令文件：通常是/etc/passwd，加密内容已经移动到其他文件，包含：登录名、加密口令（已空）、用户ID、用户组ID、注释字段、起始目录、默认shell程序
1. 组名文件：通常是/etc/group，包含：组名、组名id