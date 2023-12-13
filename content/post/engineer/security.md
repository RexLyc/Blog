---
title: "信息安全-工程实施专栏"
date: 2023-12-13T10:18:02+08:00
categories:
- 信息安全
- 工程实现
tags:
- 工程实现
- 滚动更新
- 信息安全
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/security.jpg
draft: true

---
本文从实际的工程实施中的应用角度，记录一些信息安全工具的常见用法。
<!--more-->

## Linux系统
1. 配置审计日志
    ```bash
    # 记录所有对/etc/passwd文件的读取行为，以pass_audit做key插入审计日志
    auditctl -w /etc/passwd -p r -k pass_audit
    ```
1. 配置更详细的访问控制ACL
    ```bash
    # 对用户userA，设置允许对/opt/test.txt具备读写权限
    setfacl -m u:userA:rw /opt/test.txt
    ```
1. 安全的删除文件
    ```bash
    # 使用伪随机数据填充/opt/secret.txt（默认三次），并删除文件
    shred /opt/secret.txt && rm /opt/secret.txt
    ```
1. 数据完整性和摘要
    ```bash
    md5sum xxxfile
    # 不同的sha算法生成的摘要长度不同
    sha512sum xxxfile
    ```
## 数据库
1. 设置密码策略
    ```mysql
    
    ```
1. 从数据库历史中查看可疑命令
    ```mysql
    // 查看是否有写入了文件的注入行为，例如
    select "<?php eval...?" into outfile xxx
    ```
