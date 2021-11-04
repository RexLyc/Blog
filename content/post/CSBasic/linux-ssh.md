---
title: "边学边用linux-SSH篇"
date: 2021-11-03T11:17:33+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 开坑篇
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/linux.jpg
---
SSH（Secure Shell）安全外壳协议，是常用的登录到远程服务器的协议。在日常开发中具有不可撼动的基础地位。
<!--more-->
# 基本流程
![一次基本的SSH协议流程](/images/Linux/SSH_simplified_protocol_diagram-2.webp)
1. 版本号协商：
    - 客户端发起连接请求，双方协商SSH版本号
2. 密钥和算法协商：
    - 双方协商加密算法、消息验证算法、压缩算法等
    - 交换公钥、生成会话ID
3. 加密认证：
    - 客户端向服务器端发送认证请求
4. 加密通信
# 基本脚本
1. 登录
    ```sh
    ssh user@host
    ssh -v user@host #调试模式verbose
    ssh -b ip user@host #用于本机有多个ip时，指定实用的ip
    ```
2. 挂载远程文件系统
    ```sh
    sshfs user@host:path local_path
    echo "sshfs user@host:path local_path"
    ```
# 实用进阶
# 参考资料
- 核心协议：RFC 4251（协议架构）、RFC 4253（传输层协议）、RFC 4252（鉴权协议）、RFC 4254（连接协议）
- [SSH Academy](https://www.ssh.com/academy/ssh/protocol) 