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
## 基本原理
## 安装使用实战
1. Hugo博客自动生成
    1. 在Debian系电脑上安装（2022年4月，ubuntu18.04，jenkins 2.332.2）
    ```bash
    # apt 安装的最大问题是，插件等下载地址配置容易被墙
    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt-get update
    # 安装将自动完成各类配置，并启动
    sudo apt-get install jenkins
    ```
    1. (强烈推荐)修改jenkins在服务器上的启动用户
        - 配置文件：/etc/default/jenkins，配置项JENKINS_USER / JENKINS_GROUP
    1. 用网页连接（默认8080端口），进行用户配置，初次插件安装
        - 如果存在安装失败，仍然可以在后续页面中，安装可用插件，进行重新安装
        - 重启的方式是在网页URL中直接访问/restart，如http://xxx/restart。或者后台systemctl restart jenkins。
    1. Github添加webhook
        - 默认就是jenkins地址+github-webhook，例如http://your.web.com/github-webhook
    1. jenkins添加流水线
        1. 配置git惨苦地址
        1. 配置shell构建
            - 用户就是jenkins的启动用户
            - 填写构建的shell脚本
        1. 设置邮箱
            - 不用安装邮件扩展，可以在系统管理-系统配置中，直接配置最后一项，邮件通知
            - 配置内容形如：
                - smtp服务器：smtp.qq.com
                - 默认后缀：@qq.com
                - 使用smtp：用户名（qq号），密码（qq邮箱smtp授权码）
                - 使用ssl协议
            - 在流水线中配置构建后动作，发送邮件
    1. 配置CDN刷新程序（调用运营商API）
    1. 参考：
        - [利用Jenkins+Github自动部署hugo博客](https://zhuanlan.zhihu.com/p/129069420)
## 经典问题
1. 插件安装失败
    - 修改镜像和其他
        ```bash
        # /var/lib/jenkins/hudson.model.UpdateCenter.xml
        http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
        # /var/lib/jenkins/update/default.json
        sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
        ```
    - 也可以手动安装
        - [官网插件地址](https://updates.jenkins.io/download/plugins/)
        - 插件管理->高级
1. 长时间无法启动
    - 也是网络问题，配置镜像
1. Shell构建中的权限问题
    - 描述：由于jenkins会在安装过程中，创建名为jenkins的用户，并以此为基础运行。因此很容易出现权限问题。常见的就是无法创建文件夹，无法删除等。[细节参考文件权限博文](/2022/04/边学边用linux-文件系统/)
    - 解决办法：
        - 如果是docker中，建议修改启动用户为root
        - 否则修改为常用的具有普通权限的用户
1. 设置邮箱
    - 推荐安装插件：Email Extension Plugin，配置SMTP，注意此处密码应当是授权码
        - 可以用自带的Email notification里的Test功能测试，测试无误，直接将相关内容填到插件的配置中
    - 安装推荐插件后可以在构建后事件中，创建一个Editable Email Notification。并自定义邮件内容。
1. 拉取git失败
    - 提示错误：gnutls_handshake() failed: The TLS connection was non-properly terminated. 
    - apt安装curl

## K8s部署
参考[官方教程](https://www.jenkins.io/doc/book/installing/kubernetes/)。

里面有几个要点别忘记了：
1. 修改volume种配置的```worker-node```，这个要根据具体的集群节点进行配置，如果使用的是```minikube```的话，就填写```minikube```
2. 最后的网络转发阶段不太明确，其实只需要对pod进行转发即可，不需要非得配置那个```service.yaml```，方法形如
    ```bash
    # 8080是jenkins在容器上暴露的端口，32000是宿主机向外部所有ip暴露的端口
    kubectl -n devops-tools port-forward jenkins-658f9489bf-928l7 32000:8080 --address=0.0.0.0
    ```

## 参考
1. [官方中文网站](https://www.jenkins.io/zh/doc/book/installing/#setup-wizard)