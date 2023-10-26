---
title: "信息安全专栏"
date: 2023-10-26T16:17:36+08:00
categories:
- 计算机科学与技术
- 信息安全
tags:
- 信息安全
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/security.jpg
---
本文记录所有的信息安全中的加解密知识，库，实用工具。
<!--more-->

## OpenSSL
- Windows下的编译：
    - 步骤：
        1. 安装Perl
        2. ```mkdir your-build ; cd your-build```
        3. ```perl Configure```，生成makefile等
        4. 调用Visual Studio的Native Tools Command Prompt，执行```nmake```
    - 参考：[win10 x64 visual studio 2019 编译 OpenSSL-1.1.1](https://blog.csdn.net/qiuxue110/article/details/115560240)