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
    - 服务器发送公钥（非对称加密算法）、客户端发送加密的密钥（对称加密算法）、生成会话ID等
3. 加密认证：
    - 客户端向服务器端发送认证请求（携带用户名和密码）
4. 授权后开展加密通信
> 如果采用其他认证方式，步骤2、3可能有变化
# 基本脚本
1. 登录
    ```sh
    ssh user@host
    ssh -v user@host #调试模式verbose
    ssh -b ip user@host #用于本机有多个ip时，指定实用的ip
    ssh user@host -p port #指定端口
    ```
2. 挂载远程文件系统
    ```sh
    sshfs user@host:path local_path
    sudo echo "sshfs user@host:path local_path" >> /etc/fstab #将挂载命令添加到开机挂载流程中
    ```
3. 远程拷贝
    ```sh
    scp user@src_host:/path/to/your/file user@dest_host:/path/to/your/newfile
    scp -r user@src:/path/to/your/dir user@dest_host:/path/to/your/newdir #目录整体拷贝
    ```
# 实用进阶
1. ssh端口转发（forwarding port）：
    1. 需求：有一个公网ip，有一群分布在不同内网的内网机器，希望内网机器之间能ssh互连
    2. 设备：
        - 公网主机用户名userA，地址ipA，端口portA
        - 内网登录主机用户名userSrc，地址ipSrc，端口portSrc
        - 内网被登录主机用户名userDst，地址ipDst，端口portDst
    3. 公网机器端搭建步骤
        ```sh
        ssh -fCNL 1235:localhost:1234 localhost
        # 打开防火墙 1235 1234端口
        ```
    4. 内网机器端搭建步骤
        ```sh
        ssh -fCNR 1234:localhost:22 ubuntu@82.157.175.91
        ```
    5. 其他实用
        ```sh
        killall ssh #清理全部ssh配置规则（杀死对应的服务进程）
        ssh -p1235 localhost #配置完成后，从公网机器上执行此命令跳转到内网
        ```
# 参考资料
- 核心协议：RFC 4251（协议架构）、RFC 4253（传输层协议）、RFC 4252（鉴权协议）、RFC 4254（连接协议）
- [SSH Academy](https://www.ssh.com/academy/ssh/protocol) 
- [SSH 端口转发](https://wangdoc.com/ssh/port-forwarding.html)
- [SSH反向连接及Autossh](https://www.cnblogs.com/eshizhan/archive/2012/07/16/2592902.html)
- [利用SSH端口转发登陆远程内网服务器](https://blog.csdn.net/u010412858/article/details/81270078)
- [云服务器通过内网穿透的方式ssh访问内网服务器](https://www.cnblogs.com/schips/p/using_pubilc_server_config_ssh_for_nat_in_ubuntu.html)
- 