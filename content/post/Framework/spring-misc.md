---
title: "Spring框架-疑难杂症"
date: 2023-05-18T11:20:54+08:00
categories:
- 计算机科学与技术
- Spring
tags:
- Spring
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/spring.jpg
draft: true
---
本文记录Spring在使用过程中，包括Maven等依赖库所遇到的各种疑难杂症。
<!--more-->
## 依赖库问题
1. 邮箱库：spring-boot-starter-mail
   - 问题：在使用nginx做代理的情况下（TCP代理），会发生每隔20秒进行一次自动登录的问题
   - 参考：
     - [Java Mail | Mail Properties详解](https://juejin.cn/post/6844904117018558477)
     - [廖雪峰-发送Email](https://www.liaoxuefeng.com/wiki/1252599548343744/1319099923693601)