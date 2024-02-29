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
## 资格证书
1. PAT考试
2. PMP、ACP证书

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

## 网络
1. IPV4网络地址类型
    - A类：0XXXXXXX-YYYYYYYY-YYYYYYYY-YYYYYYYY，7位网络号，24位主机号
        - 私有号段：10.0.0.0 ~ 10.255.255.255
    - B类：10XXXXXX-XXXXXXXX-YYYYYYYY-YYYYYYYY，14位网络号，16位主机号
        - 私有号段：172.6.0.0 ~ 172.31.255.255
    - C类：110XXXXX-XXXXXXXX-XXXXXXXX-YYYYYYYY，21位网络号，8位主机号
        - 私有号段：192.168.0.0 ~ 192.168.255.255
    - D类：11110开头，用于多播协议
    - E类：11111开头，保留地址
    - 特别的：
        - 本机：0.0.0.0、127.0.0.1
            > 实际上127开头都用于回路测试，不能用于公网
        - 任何全0、全1的主机号，都是不能作为ip使用的，分别是主机本身和子网广播的意思

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

## 关键词
1. 蜜罐技术：蜜罐技术是一种主动防御技术，通过部署没有真实业务数据的系统来诱骗攻击者实施攻击，记录其攻击行为从而学习攻击者的攻击目的和攻击手段，以此不断提升真实业务系统的安全防护能力。[蜜罐WIKI](https://zh.wikipedia.org/zh-hans/%E8%9C%9C%E7%BD%90_(%E9%9B%BB%E8%85%A6%E7%A7%91%E5%AD%B8))
1. N-S图：也被称为盒图或者NS图，是结构化编程（即以while、if等取代goto）种的一种可视化模型
1. SQL注入：永远不要信任用户，具体有以下办法
    - 以正则表达式等方式对用户输入进行校验，限制长度，对单引号、双引号等特殊字符进行转义或限制
    - 不动态拼装sql
    - 不使用管理员权限处理业务，为专门的服务配置专门的角色
    - 不直接存放机密信息，加密、hash、脱敏
    - 异常信息给出尽可能少的提示
    - 使用注入检测工具进行检测
1. 测试技术：
    1. 基本路径测试（独立路径测试）：在程序控制流图的基础上（仅包含单一控制条件节点的流程图），分析控制结构的环路复杂性（统计所有可达的不同路径），导出独立路径设计响应的测试用例
        - 程序控制流图：Control-Flow Graph，表示程序执行中所经过的所有路径，是许多编译器优化和静态分析的核心技术。[控制流图WIKI](https://zh.wikipedia.org/zh-hans/%E6%8E%A7%E5%88%B6%E6%B5%81%E5%9C%96)
        - 线性独立路径：一个路径是连接起点到终点的一系列边，线性独立路径是包含至少一个在其他线性无关路径中从未出现过的边的路径
        - 圈复杂度(Cyclomatic Complexity)：覆盖所有可能情况的最小测试用例个数，有点边计算法和节点及算法两种。[圈复杂度WIKI](https://zh.wikipedia.org/zh-hans/%E5%BE%AA%E7%92%B0%E8%A4%87%E9%9B%9C%E5%BA%A6)
    1. 白盒测试：测试应用程序的内部结构和运作，测试者了解程序的内部结构和算法等信息。[白盒测试WIKI](https://zh.wikipedia.org/zh-cn/%E7%99%BD%E7%9B%92%E6%B5%8B%E8%AF%95)。主要包括以下测试：
        - 控制流、数据流测试
        - 分支测试、语句覆盖、判定覆盖
        - 路径测试
    1. 灰盒测试：多用于集成测试阶段，不仅关注输出、输入的正确性，同时也关注程序内部的情况。灰盒测试不像白盒那样详细、完整，但又比黑盒测试更关注程序的内部逻辑，常常是通过一些表征性的现象、事件、标志来判断内部的运行状态
    1. 黑盒测试：功能测试，它是通过测试来检测每个功能是否都能正常使用，从用户角度出发，不看代码
    1. 验收测试：
        - α测试：由一个用户在开发环境下进行的测试，也可以是公司内部的用户在模拟实际操作环境下进行的测试。α测试的目的是评价软件产品的FLURPS(即功能、局域化、可用性、可靠性、性能和支持)。尤其注重产品的界面和特色。
        - β测试：验收测试，是软件产品完成了功能测试和系统测试之后，在产品发布之前所进行的软件测试活动，它是技术测试的最后一个阶段，通过了验收测试，产品就会进入发布阶段。验收测试一般根据产品规格说明书严格检查产品，逐行逐字地对照说明书上对软件产品所做出的各方面要求， 确保所开发的软件产品符合用户的各项要求。 通过综合测试之后，软件已完全组装起来，接口方面的错误也已排除，软件测试的最后一步——验收测试即可开始。验收测试应检查软件能否按合同要求进行工作，即是否满足软件需求说明书中的确认标准。
        - 正式验收
1. XSS漏洞：类似SQL注入，攻击者向Web页面内插入恶意Script代码，当用户浏览时将会执行。扩展阅读：
    - [Web安全头号大敌XSS漏洞解决最佳实践](https://cloud.tencent.com/developer/article/1790802)
    - [浅谈XSS攻击的那些事（附常用绕过姿势）](https://zhuanlan.zhihu.com/p/26177815)
1. ER图转关系模式的计算：ER图是由实体、实体属性、实体联系三种要素组成的图，连接中的数字代表联系中的关系(如1:1、1:n、m:n)，将其转为关系模式是非常基本的开发操作。参考[ER图转换关系模型](https://www.cnblogs.com/vvlj/p/12750853.html)
1. Servlet、Cookie、Session：
    - Servlet：一套Java语言下的Web开发规范，避免了使用原生JavaAPI编写服务器程序的繁琐步骤，更细一些的概念
        - Servlet：实现业务的类，这些实现必须遵守Servlet接口
        - Servlet容器：负责创建、使用、维护Servlet示例
        - Web容器：HTTP服务器+Servlet容器（如Tomcat、Jetty）
    - Session：在HTTP协议中使用唯一ID标识用户身份的机制
    - Cookie：由服务器创建并由浏览器存储在本地的小文本文件，其存在时间长度可能超过一次会话（会话cookie、持久性cookie、安全cookie）
1. 线性、非线性：在计算机学科中，线性用来代指集合一定存在第一个和最后一个的情况（并且在部分语境中，除了这两者，每一个元素都有唯一的前驱和后继）。比如在数据结构中，线性结构指：线性表、栈、队列，非线性结构：多维数组、树、图。
1. RAID技术要点：
    - 镜像（Mirroring）
    - 数据条带（Data Stripping）
    - 数据校验（Data Parity）
1. 死锁：
    - 死锁的发生条件：互斥、请求与保持、不剥夺、循环等待
    - 死锁避免算法：银行家算法，只能避免死锁，即不允许死锁发生。通过记录四种属性（Available、Max、Allocation、Need）
    - 死锁检测算法：
    - 死锁恢复算法：
1. NoSQL：Not Only SQL