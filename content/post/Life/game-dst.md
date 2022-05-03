---
title: "饥荒Linux服务器搭建教程"
date: 2021-08-21T21:39:30+08:00
categories:
- 娱乐
- 电子游戏
tags:
- 电子游戏
- 教程
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/dst.jpg
---
饥荒官方的服务器往往不够稳定，自己搭一个可以大幅度提高多人游戏的稳定性。并不算很复杂。步骤已经弄好啦，进来看一下就会~
<!--more-->
## 概述
本文安装于Ubuntu18.04环境，处理器2核，内存4G，带宽6M。验证有效。
## 基本步骤
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
echo "./dontstarve_dedicated_server_nullrenderer -console\
 -persistent_storage_root /home/ubuntu/Game/dstsave -conf_dir\
  dst -cluster World1 -shard Master" > master_create.sh
echo "./dontstarve_dedicated_server_nullrenderer -console\
 -persistent_storage_root /home/ubuntu/Game/dstsave -conf_dir\
  dst -cluster World1 -shard Caves" > cave_create.sh
chmod +x master_create.sh cave_create.sh
./master_create.sh #运行一下看到WILL NOT START就退出即可
./cave_create.sh #运行一下看到WILL NOT START就退出即可
# 上两句的目的是生成一下默认配置
```
2. 在本地电脑上创建一个饥荒服务器
3. 复制本地游戏目录（包含Caves、Master）到Linux服务器下指定的存储位置（包含Caves、Master）的位置
4. 用master_creat、cave_create创建游戏，记得配置防火墙
5. 配置服务器mod
    - windows上创建时，可以看到Master、Caves下面都有一个mod***.lua的文件，里面有带有workshop开头的数字串，代表mod的ID。
    - 将这一系列id以ServerModSetup("你的id")形式，分多行写入到path/to/your/dst/mods/dedicated_server_mods_setup.lua 
6. 配置管理员
    - 在指定的饥荒存档路径，如~/dstsave/World1/adminlist.txt里按行添加用户Klei ID（游戏内左下角账户信息查看）。
## 可能的问题
1. 缺少一些32位的库，并且由于系统是64位的，需要显式指定32位。(:i386)
    - 可以用ldd查看一下依赖库，避免手动尝试的尴尬。
    - [libcurl-gnutls.so.4缺少](https://wuter.cn/2282.html/)
2. steamcmd中，force_install_dir指定的不好容易忘记，建议不要指定，用默认的就行。非要用记得用绝对路径。如果你修改了，那么每一次登录steamcmd都需要重新指定，不然是会安装到默认位置的。
3. 只要客户端有更新，就要去更新服务器，重新执行一次app_update 343050 validate等指令即可。
<!-- 快乐老家 密码#Friend4Ever -->
## 一些实用远程指令
1. c_rollback()：输入参数为回退的天数，如c_rollback(2)代表回到2天前。
## 参考资料
- [主要参考：官方论坛关于SteamCMD的相关内容](https://developer.valvesoftware.com/wiki/SteamCMD)
- [centos7搭建饥荒服务器](https://blog.csdn.net/zhang41228/article/details/103106298)
- [SteamCMD良好支持的服务器游戏列表](https://developer.valvesoftware.com/wiki/Dedicated_Servers_List)