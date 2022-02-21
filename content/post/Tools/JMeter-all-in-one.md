---
title: "JMeter压测工具"
date: 2022-01-06T16:01:53+08:00
categories:
- 计算机科学与技术
- 实用工具
tags:
- 实用工具
- 后端
---
JMeter是Apache托管的一款用于进行接口测试的工具。功能比较丰富，还有插件可以进行拓展。
<!--more-->
# 安装
1. 前置安装JDK（配置好JAVA_HOME、CLASSPATH）
1. 去官网下载二进制包[传送门](https://jmeter.apache.org/download_jmeter.cgi)
1. 解压缩，windows下运行bin/jmeter.bat
# 创建测试计划
1. 创建线程组
1. 配置登录请求（可选）
1. 从登录请求中获取access_token（可选）
1. 发送待测试请求
1. 查看结果树
# 测试配置详解