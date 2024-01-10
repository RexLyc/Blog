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

> 谨慎使用，WSL2仍然不够稳定。

## 基础搭建
1. 基本步骤：
    1. 在“启用或关闭Windows功能”中，打开虚拟机平台、Linux子系统两个选项即可。
    2. 重启
    3. 不想安装在C盘，可以参考[旧版 WSL 的手动安装步骤](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual)，更新内核，下载所需镜像，移动到喜欢的路径下，更改后缀名（从AppxBundle到zip），解压缩并运行其中的ubuntu.exe，此后Linux将存储在exe同级位置
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
   1. ~~图形界面创建可能有问题，建议还是在命令行里用```docker run```~~
   2. ~~ubuntu22.04官方镜像无法进行```apt-get update```。**宿主机一侧的网桥并没有工作**。~~
      > 尚未有结论，但是看起来wsl中的docker工作方式略有不同，其网桥无法通过```brctl```、```ip addr```观测到，可以用```docker network ls```看一下，参考[坑](#坑)
   4. 在尝试安装elasticsearch的过程中，使用图形化界面绑定端口，发现该端口是容器映射到wsl2，需要在做一步从wsl2映射到windows，[参考](https://blog.csdn.net/keyiis_sh/article/details/113819244)，powershell执行如下指令
      ```powershell
      # 先查看wsl的地址
      wsl -- ifconfig
      # listenXXX是windows侧，connectXXX是wsl侧
      netsh interface portproxy add v4tov4 listenport=7777 listenaddress=0.0.0.0 connectport=7777 connectaddress=172.22.153.228
      # 查看映射配置
      netsh interface portproxy show all
      ```



### ElasticSearch
参考[命令来源](https://ion-utale.medium.com/how-to-install-elasticsearch-with-kibana-on-wsl-2-docker-engine-90d6335a07c0)，主要需要考虑将ElasticSearch和Kibana连接起来
```bash
# wsl2内
docker network create es-stack-network
# 根据你所使用的版本
docker run -d --name elasticsearchdb --net es-stack-network -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:8.11.3

docker run -d --name kibana-es-ui --net es-stack-network -e "ELASTICSEARCH_URL=http://elasticsearchdb:9200"  -p 5601:5601 kibana:8.11.3

# ====== windows ======
netsh interface portproxy add v4tov4 listenport=5601 listenaddress=0.0.0.0 connectport=5601 connectaddress=172.22.153.228
```
注意
1. 初次启动需要进行一次token的验证，先去es下的bin中生成，再去kibana下查看验证码。
2. 初次启动kibana需要密码，也可以从es下的bin重置
3. 非初次启动记得先启动ES，确定ES日志中显示OK，再启动Kibana。否则Kibana会永远显示ES Server未就绪。

### 交叉编译

### 显卡

## 坑
1. Windows下，WSL2内的Ubuntu容器无法访问外网，**仍未解决**
   - Docker Desktop在Windows上确实不支持Host模式，具体的使用[参考微软文章](https://learn.microsoft.com/en-us/virtualization/windowscontainers/container-networking/network-drivers-topologies)
   - 关于网络部分的其他限制，参考[官网](https://dockerdocs.cn/docker-for-windows/networking/)
   - 关于WSL2的网络拓扑，参考[WSL2设置桥接网络及高级设置 _](http://www.ronnyz.top/2023/11/18/WSL2%E8%AE%BE%E7%BD%AE%E6%A1%A5%E6%8E%A5%E7%BD%91%E7%BB%9C%E5%8F%8A%E9%AB%98%E7%BA%A7%E8%AE%BE%E7%BD%AE/)

## 参考
1. [官方文档目录：适用于 Linux 的 Windows 子系统文档](https://learn.microsoft.com/zh-cn/windows/wsl/)
2. [安装并开始设置 Windows 终端](https://learn.microsoft.com/zh-cn/windows/terminal/install)
3. [WSL 2 上的 Docker 远程容器入门](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-containers)
4. [Docker Desktop WSL 2 backend on Windows](https://docs.docker.com/desktop/wsl/#download)