---
title: "Among Us私服教程"
date: 2021-11-13T20:12:25+08:00
categories:
- 娱乐
- 电子游戏
tags:
- 电子游戏
- 教程
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/among-us.jpg
---
among us是一个非常好玩的，多人在线扯皮、互演游戏。但是他的服务器实在是太差了。本文的目标就是解决这个问题。
<!--more-->
## 服务器端步骤
1. 去Github上下载对应的服务器端程序
2. 在服务器上安装.NET
```sh
# 添加包来源
wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb \
       -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

#安装sdk
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-5.0

#安装运行时
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y aspnetcore-runtime-5.0
```
3. 修改服务器配置config.json
    - 主要是修改公共ip，可参考[官方教程](https://github.com/Impostor/Impostor/blob/master/docs/Running-the-server.md)
4. 运行
```sh
./Impostor.Server
```
## 客户端步骤
1. 去Github上下载客户端配置生成工具
    - 或者去这里[在线生成](https://impostor.github.io/Impostor/)
2. 复制到本地游戏的配置位置
    - 例如C:\Users\liyicheng\AppData\LocalLow\Innersloth\Among Us
## 已知问题
1. 游戏升级到2021.11.9之后，无法使用目前的Impostorv1.9.0
    - 游戏使用了新的验证机制，等待Github仓库更新。
    - 20220427 更新
      1. 下载仓库的CI Build最新版
      1. 在服务器的配置中，添加
        ```json
        "AuthServer": {
          "Enabled": false
        }
        ```
        > 暂时关闭了验证用的服务，有时间的读者可以研究研究
## 注意事项
1. 安装.NET框架，一定注意版本。
## 参考资料
- [服务器、客户端Github仓库](https://github.com/Impostor/Impostor)
- [搭建教程](https://www.bilibili.com/read/cv13573966)
- [微软.NET官网安装导航](https://dotnet.microsoft.com/download/dotnet)
