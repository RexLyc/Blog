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
thumbnailImage: /images/thumbnail/jenkins.png
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
        - 默认就是jenkins地址+github-webhook，例如http://your.web.com/github-webhook
    1. jenkins添加流水线
    1. 配置CDN刷新程序（调用运营商API）
    1. 参考：
        - [利用Jenkins+Github自动部署hugo博客](https://zhuanlan.zhihu.com/p/129069420)
        <!-- https://console.cloud.tencent.com/api/explorer?Product=cdn&Version=2018-06-06&Action=PurgePathCache&SignVersion= -->

        <!-- https://console.cloud.tencent.com/api/explorer?Product=cdn&Version=2018-06-06&Action=PurgePathCache&SignVersion= -->
# 经典问题
1. Shell构建中的权限问题
    - 描述：由于jenkins会在安装过程中，创建名为jenkins的用户，并以此为基础运行。因此很容易出现权限问题。常见的就是无法创建文件夹，无法删除等。[细节参考文件权限博文](/2022/04/边学边用linux-文件系统/)
    - 解决办法：
1. 设置邮箱
    - 推荐安装插件：Email Extension Plugin，配置SMTP，注意此处密码应当是授权码
    - **未成功**
# 参考
1. [官方中文网站](https://www.jenkins.io/zh/doc/book/installing/#setup-wizard)