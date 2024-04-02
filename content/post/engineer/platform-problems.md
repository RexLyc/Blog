---
title: "各平台工程问题实录"
date: 2024-03-25T15:07:36+08:00
categories:
- 计算机
- 工程部署
tags:
- 部署
- 工程
# thumbnailImagePosition: left
# thumbnailImage: //example.com/image.jpg
draft: true
---

<!--more-->
## 网络
1. windows下作为服务器时，可能遇到的无法通信的网络情况
    1. 未开放防火墙权限。需要修改，允许应用通过防火墙。
    2. 允许通过防火墙后，仍然无法访问，运行```secpol.msc```，可能需要专门放行某些ip端口策略。