---
title: "CI/CD系列：Jenkins"
date: 2022-04-14T15:14:51+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/IBM.jpg
---
Jenkins是CI/CD工具中非常重要的一个持续集成工具。本文进行简要介绍总结。
<!--more-->
# 基本原理
# 安装使用实战
1. Hugo博客自动生成
    1. 在Debian系电脑上安装（2022年4月）
    ```bash
    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt-get update
    # 安装将自动完成各类配置，并启动
    sudo apt-get install jenkins
    ```
    1. 用网页连接（默认8080端口），进行用户配置，初次插件安装
        - 如果存在安装失败，仍然可以在后续页面中，安装可用插件，进行重新安装
        - 重启的方式是在网页URL中直接访问/restart，如http://xxx/restart
    1. Github添加webhook
    1. jenkins添加流水线
    1. 参考：[利用Jenkins+Github自动部署hugo博客](https://zhuanlan.zhihu.com/p/129069420)
# 参考
1. [官方中文网站](https://www.jenkins.io/zh/doc/book/installing/#setup-wizard)