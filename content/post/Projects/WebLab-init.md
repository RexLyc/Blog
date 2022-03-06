---
title: "WebLab：开坑篇"
date: 2021-12-13T23:53:16+08:00
categories:
- 计算机科学与技术
- 全栈
tags:
- 系列开坑
- 网站建设
- 实验室
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/weblab.jpg
# 画一个齿轮 + 网络 + 101010的图标
---
本系列将会以搭建项目为导向，从实战角度出发学习网络相关技术。
<!--more-->
# 远程启动项目
1. 技术内容
    1. 支持身份验证，基于RBAC的控制
    1. 前端Vue3 + Typescript
    1. 后端Java + Spring
    1. 终端设备Python
    1. 必要时使用WebSocket长连接
    1. 其他中间件MySQL + Nginx
1. 项目需求
    1. 支持从外网登陆，并控制对应的终端设备开机。
    1. 终端设备和后端服务器保持长连接通信，前端和后端保持长连接通信。实时显示设备状态。
    1. ~~日志统计和分析。~~
