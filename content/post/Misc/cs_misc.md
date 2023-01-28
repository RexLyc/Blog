---
title: "技术杂项"
date: 2021-11-01T15:20:01+08:00
categories:
- 计算机科学与技术
- 杂项
tags:
- 杂项
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/misc.jpg
---
一些尚未形成体系，但是可能很有用的技术杂项整理。
<!--more-->
## 开发工具
### Python：
- Mock服务器搭建
    - 参考：[用flask搭建Python Mock服务器](http://t.zoukankan.com/xiaobaibailongma-p-12992802.html)、[Postman搭建Mock服务器](https://www.cnblogs.com/quchunhui/p/11881185.html)
### IDEA
- 自动添加注释
    - 参考: [IDEA类和方法注释模板设置（非常详细）](https://blog.csdn.net/xiaoliulang0324/article/details/79030752)

## Web
1. PixiJS：纹理重复加载问题
    - 问题描述：使用Pixi在加载纹理的时候，经常出现重复加载的警告。显然可以优化。
    - 基础写法
        ```javascript
        textureList = ['a.jpg','b.jpg'];
        loader.add(textureList);
        loader.load((loaders, resources) => {
            //balabala
        });
        ```
    - 问题原因：加载的时候，会将纹理加载进缓存，警告也是提示这一点，缓存中已经有重名的纹理了。
    - 修改写法：使用默认的loader，并使用PIXI.utils获取缓存，加载前进行去重判断
        ```javascript
        app.loader = PIXI.Loader.shared;
        for (let texture of textureSet) { // textureSet是存储纹理加载路径的集合
            if (PIXI.utils.BaseTextureCache[texture]) {
                textureSet.delete(texture);
            }
        }
        let textureList = Array.from(iconSet);
        if (textureList.length != 0) {
            app.loader.add(textureList).loader((loader, resources) => {
                // balabala
            });
        }
        ```
    - 参考:[精灵加载去缓存](https://segmentfault.com/a/1190000022280843)
1. CSS：
    - 国家哀悼日等情况时，设置全站灰色的办法，在顶层添加
        ```css
        -webkit-filter: grayscale(.95);
        ```

1. WebSocket并发问题
    - 当并发量较大时，调用session.getAsyncRemote/getBasicRemote.send()，均会报错误“TEXT_FULL_WRITING”。
    - [一个解决方案，尚未确认](https://blog.csdn.net/wy_xing/article/details/82744559)

1. MyBatis，使用缓存时需要将所有查询结果实体类进行可序列化声明
    - 为什么不是仅声明需要缓存的实体为可序列化的类型？

## 树莓派
### 网络配置
- 树莓派还是推荐使用Raspbian系统，而且最好选择无桌面版，功耗低
- 配置网络推荐使用自带工具：raspi-config，可以对大部分功能、硬件进行设置。
- 一些坑
    - 启动时需要连接HDMI，可以通过配置支持热拔插
    ```bash
    # /boot/config.txt
    hdmi_force_hotplug=1
    ```
## 命令行工具
### ZSH美化
1. 环境：MobaXterm（Windows）、Raspbian
2. 美化脚本
    1. 在目标机器上安装zsh并启用
        ```sh
        # 安装zsh
        sudo apt install zsh
        # 设置为当前用户的默认shell
        sudo usermod -s `whereis zsh` $(whoami)
        sudo reboot
        ```
    2. 在本机上安装需要的字体（MobaXterm一般不需要再安装了）
    3. 继续目标机器上的配置
        ```sh
        # 安装powerlevel10k
        git clone --depth=1 https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
        p10k configure
        # 安装zsh高亮
        sudo apt install zsh-syntax-highlighting
        echo "source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ~/.zshrc
        # 当然也可以实用oh-my-zsh来完成主题的配置
        sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
        ```
3. 参考：[Ubuntu18.04安装并美化zsh](https://www.sysgeek.cn/install-zsh-shell-ubuntu-18-04/)

### Powershell
1. 更换当前终端的字符编码（从GBK变为UTF8）
    ```bash
    chcp 65001
    ```

## 小飞机
### hostwinds
- 比较靠谱的供货商，但是其无管理（就是需要用户自己管理）的套餐，管理起来确实有些坑，在这里记录一下。
    1. 封锁问题：目前没有很好的办法，只能用其提供的fix功能
    1. 解决封锁后ipv6丢失问题：
        ```sh
        # /etc/network/interfaces
        # 这个文件是networking服务用于控制网络的配置
        # 一般来说，缺少了ipv6时需要自己在这里做如下内容
        # Interface lo
        auto lo
        iface lo inet loopback

        # Interface ens3
        auto ens3
        iface ens3 inet static
            address 你的静态ipv4
            netmask 你的掩码
            gateway 你的网关
        iface ens3 inet6 static
            address 你的静态ipv6
            netmask 你的掩码
            gateway 你的网关

        dns-nameservers 你喜欢的dns（可以不写ipv6的）
            ```
    1. 仍然留着的坑：
        1. 重启之后ipv6会再次丢失
        1. 截止到目前无法恢复
## Windows
1. vc_redist那些事儿
    - 可以命令行静默安装
    ```powershell
    # 查看帮助
    ./vc_redist.x64.exe /?
    # 静默安装
    ./vc_redist.x64.exe /install /quiet
    ```


## Ubuntu
1. 系统没有网络配置中的有线设置
    - network相关的配置问题，需要修改NetworkManager.conf、10-globally-managed-devices.conf文件，删除NetworkManager服务state文件，重新启动系统及服务
    - 参考：[ubuntu18.04没有网络，网络中或者右上角没有有线设置](https://blog.csdn.net/lylg_ban/article/details/121657952)