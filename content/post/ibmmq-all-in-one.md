---
title: "IBM WebSphere MQ"
date: 2021-07-27T16:43:16+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
#thumbnailImage: //example.com/image.jpg
---
由于工作原因，需要使用到IBMMQ，由于相关资料较少，所以在此特地整理一篇。尽力涵盖安装、配置、使用。
<!--more-->
# 安装
最离谱的事情就是安装，我到现在都不知道从哪里能下载到Linux的服务器安装包，以及Windows的Explorer客户端的安装包。
- Linux
    - 找到压缩包之后（名称如：WS_MQ_V8.0.0.2_LINUX_ON_X86_64_RELEASE.tar.gz），解压。
    - server目录下，./mqlicense.sh -accept
    - 