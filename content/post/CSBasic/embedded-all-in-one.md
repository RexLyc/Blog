---
title: "嵌入式软件开发合集"
date: 2022-08-08T15:09:14+08:00
categories:
- 计算机科学与技术
- 嵌入式
tags:
- 合集
- 嵌入式
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/embedded.jpg
math: true
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
    - RTU（Remote Terminal Unit）：该模式使用帧间隔来区分帧，两个帧之间至少间隔3.5个字符发送时间。
    - ASCII：该模式是使用特殊字符来标记帧起始和结束，同时规定ADU内只能使用ASCII编码下的'0'~'f'这16种数值。需要用两个字符才能表示RTU的ADU中的一个字符
    - TCP：用于在TCP上传输时的报文形式，替换了ADU头部，同时由于TCP具备校验能力，去除了校验部分
- 术语：
    - 线圈（Coil）：boolean状态值，来源是早期的设备控制都是磁线圈，只有上电和失电两种状态。
        - 在一些地方也被称为“位”，这个就是使用了现代计算机的Bit概念了
    - 寄存器（Register）：int值，这个就是通用的寄存器的含义了。
    - ADU（Application Data Unit） & PDU（Protocal Data Unit）：
        <center> <img src="/images/embedded/adu-modbus.png" </center>
    - 功能码：用于表明该指令的业务目的，取状态值/设置状态值
- 坑：
    - 在使用Modbus协议发送数据时，如果底层采用RTU，需要注意间隔发送，否则粘包。即使是使用上层Modbus TCP时，如果最终转为Modbus RTU，也需要注意这个问题。
    - 使用Modbus TCP转RTU时，需要注意TCP格式中的内容。
        > [参考：工业以太网杂谈（一） Modbus TCP/IP](https://zhuanlan.zhihu.com/p/568309501)：关于设备识别号，对于Modbus TCP/IP协议该项默认255，但是如果该协议为Modbus Plus或者Modbus RTU等串口协议，通过串口服务器等转换设备转换后变为了Modbus RTU over TCP，则该项为Modbus 串口从站的设备地址。
- 工具：
    - Modbus Poll/Slave: Poll用来模拟一个主站设备，便于调试从站。Slave则用来模拟从站设备，便于调试主站。
- 参考：
    - [Modbus Poll/Slave 模拟器使用教程](https://blog.csdn.net/qq_35029061/article/details/125865898)
    - [知乎：Modbus协议详解与案例演示](https://zhuanlan.zhihu.com/p/537762158)