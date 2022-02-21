---
title: "技术杂项"
date: 2021-11-01T15:20:01+08:00
categories:
- 计算机科学与技术
- 杂项
tags:
- 杂项
- 滚动更新
---
一些尚未形成体系，但是可能很有用的技术杂项整理。
<!--more-->
# Web
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

1. WebSocket并发问题
    - 当并发量较大时，调用session.getAsyncRemote/getBasicRemote.send()，均会报错误“TEXT_FULL_WRITING”。
    - [一个解决方案，尚未确认](https://blog.csdn.net/wy_xing/article/details/82744559)

1. MyBatis，使用缓存时需要将所有查询结果实体类进行可序列化声明
    - 为什么不是仅声明需要缓存的实体为可序列化的类型？

# 树莓派
## 网络配置
- 树莓派还是推荐使用Raspbian系统，而且最好选择无桌面版，功耗低
- 配置网络推荐使用自带工具：raspi-config，可以对大部分功能、硬件进行设置。
# 命令行工具
## ZSH美化
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