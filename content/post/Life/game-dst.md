---
title: "饥荒Linux服务器搭建教程"
date: 2021-08-21T21:39:30+08:00
categories:
- category
- subcategory
tags:
- tag1
- tag2
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
# 概述
本文安装于Ubuntu18.04环境，处理器2核，内存4G，带宽6M。验证有效。
# 基本步骤
1. 安装steamCMD、登录、安装饥荒服务器
```bash
sudo apt install steamcmd
# 可能需要的依赖 sudo apt install lib32gcc1 libsdl2-2.0.0:i386
steamcmd
Steam> login anonymous
  系统输出一堆balabala
Steam> app_update 343050 validate
  系统输出一堆balabala
Steam> quit
cd /path/to/your/dst/bin
echo "./dontstarve_dedicated_server_nullrenderer -console -persistent_storage_root /home/ubuntu/Game/dstsave -conf_dir dst -cluster World1 -shard Master" > master_create.sh
echo "./dontstarve_dedicated_server_nullrenderer -console -persistent_storage_root /home/ubuntu/Game/dstsave -conf_dir dst -cluster World1 -shard Caves" > cave_create.sh
chmod +x master_create.sh cave_create.sh
./master_create.sh #运行一下看到WILL NOT START就退出即可
./cave_create.sh #运行一下看到WILL NOT START就退出即可
# 上两句的目的是生成一下默认配置
```
2. 在本地电脑上创建一个饥荒服务器
3. 复制本地游戏目录（包含Caves、Master）到Linux服务器下指定的存储位置（包含Caves、Master）的位置
4. 用master_creat、cave_create创建游戏，记得配置防火墙
5. 配置服务器mod
    - TODO
6. 配置管理员
    - TODO

# 可能的问题
1. 缺少一些32位的库，并且由于系统是64位的，需要显式指定32位。(:i386)
    - 可以用ldd查看一下依赖库，避免手动尝试的尴尬。
    - [libcurl-gnutls.so.4缺少](https://wuter.cn/2282.html/)
2. force_install_dir指定的不好，建议用绝对路径。这里的当前路径是steamCmd的安装位置，ubuntu的apt安装后大概是~/.steam下面的某一级。我现在这个就弄乱了。
<!-- 快乐老家 密码#Friend4Ever -->
# 参考资料
[主要参考：官方论坛关于SteamCMD的相关内容](https://developer.valvesoftware.com/wiki/SteamCMD)
[centos7搭建饥荒服务器](https://blog.csdn.net/zhang41228/article/details/103106298)
[SteamCMD良好支持的服务器游戏列表](https://developer.valvesoftware.com/wiki/Dedicated_Servers_List)