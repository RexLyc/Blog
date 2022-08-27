---
title: "嵌入式软件开发合集"
date: 2022-08-08T15:09:14+08:00
categories:
- 计算机科学与技术
- 嵌入式
tags:
- 持续施工
- 合集
- 嵌入式
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/embedded.jpg
draft: true

---
嵌入式软件开发是软件开发中比较特殊的一环，对于开发人员来说，往往需要具备对硬件的一定了解。
<!--more-->
## 串口通信

### 基本概念
- 位：
    - 起始位
    - 数据位
    - 停止位
    - 校验位

## Modbus协议
- 概述：一种应用层协议，即不限定所使用的物理层等底层设施。目前ModBus有ASCII、RTU、TCP三种形式。
    - ASCII
    - RTU
    - TCP
- 术语：
    - 线圈（Coil）：boolean状态值，来源是早期的设备控制都是磁线圈，只有上电和失电两种状态。
        - 在一些地方也被称为“位”，这个就是使用了现代计算机的Bit概念了
    - 寄存器（Register）：int值，这个就是通用的寄存器的含义了。

## ToDo