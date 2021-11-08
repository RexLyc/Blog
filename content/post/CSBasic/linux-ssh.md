---
title: "边学边用linux-SSH篇"
date: 2021-11-03T11:17:33+08:00
categories:
- 计算机科学与技术
- 操作系统
tags:
- 操作系统系列
- 命令行
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
1. ssh跳板与端口转发（forwarding port）：
    1. 需求：有一个公网ip，有一群分布在不同内网的内网机器，希望内网机器之间能ssh互连
    2. 设备：
        - 公网主机用户名userA，地址ipA，对外网登录主机端口portA，对内网被登陆主机端口portB
        - 内网登录主机用户名userSrc，地址ipSrc，端口portSrc
        - 内网被登录主机用户名userDst，地址ipDst，端口portDst
    3. 公网机器端搭建步骤
        ```sh
        # 本地代理，将portA内容转发给portB
        # *号表示支持任意ip访问portA，不局限于本公网机器
        ssh -fCNL "*:portA:localhost:portB" localhost 
        # 也是本地代理，但不支持外网访问portA
        ssh -fCNL portA:localhost:portB localhost 
        ```
    4. 内网机器端搭建步骤
        ```sh
        # 远程代理，将代理机上portB转发给本地22
        ssh -fCNR portB:localhost:22 userA@ipA
        ```
    5. 免密：
        - 方法一：将公钥互相赋给对方的authorized_keys
            ```sh
            # 将本地公钥拷给远端机器
            ssh-copy-id -i .ssh/id_rsa.pub remoteUser@remoteHost
            ```
        - 方法二：用spawn、expect、send指令完成自动发送密码（密码在脚本中明文存储了，不太安全）
            ```sh
            # 安装必须内容
            sudo apt install expect
            sudo apt install tcl tk # 可能需要安装
            # 编写脚本如下（注意第一行的内容是必须的，即该脚本由expect执行）
            #!/usr/bin/expect
            spawn ssh -fCNR portB:localhost:22 userA@ipA # spawn包裹下的原指令
            expect "*password" # 匹配模式
            send "你的密码" # 危险
            expect eof # 结束
            ```
    6. autossh和开机自启动
        - 经过以上步骤，已经能够进行基础的ssh转发了，但是这种方式并不够稳定，开机也未自动化。接下来的步骤将会解决这个问题
            ```sh
            # 使用crontab，支持各种linux版本
            crontab -e
            # 内网被登陆机器侧，进入编辑页面，在存储着公钥的用户账号下填入以下开机计划
            @reboot autossh -M 监听端口 -fCNR portA:localhost:22 userA@ipA
            # 也可以为了方便修改，将内容存放到另外的脚本，并填写开机计划如
            @reboot /home/user/my-autossh.sh > /home/user/crontab.log 2>&1
            ```
        - my-autossh.sh脚本（需要考虑网络问题）
            ```sh
            #!/usr/bin/zsh
            while true
            do
                    # 检测到指定主机的连通，避免crontab执行时网络尚未连通
                    ping -c 8 -w 100 82.157.175.91
                    if [[ $? != 0 ]];then
                            echo " ping fail "
                            sleep 5
                    else
                            echo " ping ok"
                            break
                    fi
            done
            echo "try to open autossh"
            killall ssh # 清理ssh
            killall autossh
            autossh -M 5678 -fCNR 1234:localhost:22 ubuntu@82.157.175.91
            echo "finish open autossh"
            ```
        - 外网主机侧修改配置/etc/sshd/sshd_config
            ```sh
            # 避免ssh连接意外中断后外网服务器不及时回收端口
            # 30s进行一次探测
            ClientAliveInterval 30
            # 3次失败则断开
            ClientAliveCountMax 3
            ```
    7. 其他实用
        ```sh
        # 清理全部ssh代理配置（杀死对应的服务进程）
        killall ssh
        # 若公网机器使用了"*:portA..."，可从任意机器登录内网机器
        ssh -p portA userDst@ipA
        # 若未使用"*:portA...",只能从公网机器上执行此命令跳转到内网
        ssh -p portA userDst@localhost
        knock # 检测远端端口是否开放的命令
        ```
    8. 注意事项
        - 需打开所有涉及到的端口的防火墙
        - 本教程的方法不仅仅支持ssh协议，这种端口转发实际上构建了一个安全的隧道，稍加改造即可用于其他协议的使用
        - 本文环境中，内网被登录机器机器为RaspberryPi，有一些和教程有出入的地方
            - autossh使用-f时才能在后台运行，而此时无法使用spawn，只能使用上传公钥的免密方式
            - crontab在不同用户下的执行任务是区分的，需要在指定用户的命令行环境下进行设置
# 参考资料
- 核心协议：RFC 4251（协议架构）、RFC 4253（传输层协议）、RFC 4252（鉴权协议）、RFC 4254（连接协议）
- [SSH Academy](https://www.ssh.com/academy/ssh/protocol) 
- [SSH 端口转发](https://wangdoc.com/ssh/port-forwarding.html)
- [SSH反向连接及Autossh](https://www.cnblogs.com/eshizhan/archive/2012/07/16/2592902.html)
- [利用SSH端口转发登陆远程内网服务器](https://blog.csdn.net/u010412858/article/details/81270078)
- [云服务器通过内网穿透的方式ssh访问内网服务器](https://www.cnblogs.com/schips/p/using_pubilc_server_config_ssh_for_nat_in_ubuntu.html)
- [sshd_config配置详解](https://www.cnblogs.com/wangliangblog/p/6226488.html)