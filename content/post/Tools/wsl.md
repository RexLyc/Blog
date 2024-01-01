---
title: "WSL：微软Linux子系统"
date: 2023-12-05T21:58:33+08:00
categories:
- 实用工具
- 子系统
tags:
- VMWare
- 子系统
- 暂时废弃
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/wsl.png
draft: true

---
WSL在经过一段时间的发展后，已经具备了一定的可用性。相对VMWare来说，有性能和易用性上的优点。
<!--more-->

> 暂时废弃，WSL2仍然不够稳定。

## 基础搭建
1. 基本步骤：
    1. 在“启用或关闭Windows功能”中，打开虚拟机平台、Linux子系统两个选项即可。
    2. 重启
    3. 不想安装在C盘，可以参考[旧版 WSL 的手动安装步骤](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual)，下载所需镜像，移动到喜欢的路径下，更改后缀名（从AppxBundle到zip），解压缩并运行其中的ubuntu.exe，此后Linux将存储在exe同级位置
    4. 安装过程中可能遇到问题，参考
        - [WSL2出现“参考的对象类型不支持尝试的操作”的解决方法](https://cloud.tencent.com/developer/article/1986728)
        - [wsl安装到非C盘解决方案](https://zhuanlan.zhihu.com/p/419242528)
2. 注意事项
   1. 安装后，WSL2会在Windows内创建一个ext4.vhdx文件，作为提供给Linux使用的虚拟磁盘，磁盘内存放所有Linux文件（根目录）
   2. ext4.vhdx可能会被Windows磁盘管理识别并挂载，此时将无法启动子系统，需要去磁盘管理中分离该VHD

## 开发环境
### 基本开发
1. 更好用的命令行：推荐安装微软的全新[终端](https://learn.microsoft.com/zh-cn/windows/terminal/install)
2. docker：查看[参考](#参考)中的相关链接。可能会遇到的问题
   1. 图形界面创建可能有问题，建议还是在命令行里用```docker run```
   2. ubuntu22.04官方镜像无法进行```apt-get update```。**宿主机一侧的网桥并没有工作**。

### 交叉编译

### 显卡

## 参考
1. [官方文档目录：适用于 Linux 的 Windows 子系统文档](https://learn.microsoft.com/zh-cn/windows/wsl/)
2. [安装并开始设置 Windows 终端](https://learn.microsoft.com/zh-cn/windows/terminal/install)
3. [WSL 2 上的 Docker 远程容器入门](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-containers)
4. [Docker Desktop WSL 2 backend on Windows](https://docs.docker.com/desktop/wsl/#download)