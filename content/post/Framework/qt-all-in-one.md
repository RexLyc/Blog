---
title: "Qt合集"
date: 2022-02-09T15:28:10+08:00
categories:
- 计算机科学与技术
- Qt
tags:
- Qt
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/qt-logo.svg
---
Qt是一个非常好用的C++框架，最主要的用途是用来开发跨平台GUI程序。
<!--more-->
本文主要记录一些开发过程中遇到的经典坑和关键理解。以高版本（不低于5.4）为例。
## 核心机制关键词
1. 信号和槽
    1. 观察者模式：信号和槽本质上就是观察者模式的一种实现
    1. 转换：在moc过程中转换为标准的C++代码
    1. QObject：只有QObject派生类，且添加了Q_OBJECT宏才可以使用该机制
    1. public：signals不能添加限定符，默认就是public的，信号只能声明，不能定义，返回值必须为void
    1. 参数：信号的参数量不能小于槽函数的参数量
    1. 级联：信号可以级联（信号连接信号），信号将会连接顺序进行传递
1. 元对象系统
    - 一个基于标准C++的扩展，提供对象间通信的信号槽机制，实施类型信息和动态属性等功能
    - 实现基于QObject类、Q_OBJECT宏、元对象编译器moc
1. 事件模型
    - 主事件循环从事件队列中获取本地窗口系统事件，分发给对应的接收对象
    - 用户也可以由QEventLoop创建本地事件循环，管理事件并分发
1. MOC
    - Meta-Object Compiler：元对象编译器
1. UIC
    - User Interface Compiler：图形界面编译器
1. QMake
1. 国际化，QtLinguist
    - 对用户可见的文本信息全部使用tr函数进行封装，ts文件报错了所有待翻译内容，qm保存了所有翻译结果
## 扩展
### 绘图
1. QGraphics：Qt自带的一系列绘图用的类型，也是大部分第三方绘图库的实际底层
    - [PyQt5 实现图元（QGraphicsItem）的建立、操作和连接](https://blog.csdn.net/qq_25000387/article/details/106025439)
1. QCustomPlot：短小精悍的第三方库，主要用于制作美观的数据可视化图标
1. PyQtGraph：功能丰富的第三方库，从2D、3D图标，到树状数据列表，该库提供了非常丰富的数据绘制方式
    - [PyQtGraph](https://pyqtgraph.readthedocs.io/en/latest/)
### QWebEngine
1. 概述：从Qt5.4.0版本后，Chromium替代了Webkit成为了Qt未来更新Web控件的内核，只要系统允许（win7及以上），都应当使用QWebEngine
1. 一些坑
    - [QtWebEngine图形性能问题](https://cloud.tencent.com/developer/article/1995597)
## 32位编译
1. 由于高版本后Qt仅提供64位的安装程序，32位需要自行编译。在此记录Linux(Debian 11.2 i386)下编译的流程和注意实现。
1. 基本流程
```bash
sudo apt install git
git clone git://code.qt.io/qt/qt5.git
cd qt5
# 以5.13.0版本为例
git checkout 5.13.0
perl init-repository
# 必要前置
sudo apt install build-essential libgl1-mesa-dev
sudo apt install python # >2.7.5
cd ../
mkdir qt5130build
cd qt5130build
# 可以根据选项自行选择
../qt5/configure –nomake examples –nomake tests
# 可以跳过一些用不到的模块
../qt5/configure -skip qtlocation
# 指定安装路径
../qt5/configure --prefix=/path/to/your/install/directory
```
1. 更多配置内容参考[官方网站-ConfigOption](https://doc.qt.io/qt-5/configure-options.html),[官方网站-编译总结篇](https://wiki.qt.io/Building_Qt_5_from_Git#Getting_the_source_code)。

## 打包
- 因为是Qt系列，所以此处用Qt官方的Qt Install Framework作为打包方式，不过该模块的中文资料相对较少，主要还是得去看官方文档
- 目录结构
    ```bash
    # /my-app/
    #      /config
    #             /config.xml  # 整体的一些配置、信息
    #             /install.js  # 整体安装的执行脚本
    #      /packages
    #             /包1
    #                 /data    # 放你的程序文件
    #                 /meta
    #                      /install.qs  #  包1的安装脚本
    #                      /package.xml #  包1的配置、信息
    ```
- 核心命令
    ```bash
    # -c为配置文件路径 -p为打包目录 打包名为afc2xxx的安装包
    binarycreator.exe -c .\config\config.xml -p packages afc2browswer_install.exe -v
    ```
- 参考
    - [Qt Install Framework官方文档](https://doc.qt.io/qtinstallerframework/)
    - [Qt Install Framework应用总结](https://blog.csdn.net/youzai2017/article/details/124728929)
### win
- 使用官方提供的windeployqt.exe
- 一些坑
    - [Qt打包发布程序，解决找不到msvcp140.dll等动态库问题正确方案](https://blog.csdn.net/no_say_you_know/article/details/126360830)
    - [OpenGL环境检测和设置](https://blog.csdn.net/mvmmvm/article/details/122177404)
### linux
- Linux下并没有自带部署工具，但由于Linux的特殊性，只需实用ldd查看依赖项并进行拷贝即可，拷贝步骤可以放在CMake中进行

## Qt-Android
1. 首先，不推荐用Qt做这个开发，还是用原生的开发，资料多，坑少。其次，非用不可的情况下，Qt作为跨平台开发框架，也可以用于Android开发，在这里记录一下。
1. 环境搭建：
    - 先安装好Android Studio，Java JDK，Android SDK Commandline Tool这三样东西。注意SDK CMD Tools和JDK版本有依赖，如果使用较低版本的JDK，则最新的工具可能会报错。前两者作用自不必说，第三个的工具作用是让Qt Creator有下载、安装、检查安卓各项依赖的能力
    - 安装Qt，注意一定要安装其Android依赖，如果不确定，就全安装了吧
1. 开发
    - 引入第三方so、jar
        - 在Qt开始编译后，会生成一个Android构建目录，在那下面有libs文件夹，直接拷进去就行
    - 自定义AndroidMenifest.xml
        - 在Qt的CMakeLists中，添加配置，然后在该配置项指向的目录中编写一个AndroidMenifest，将优先使用该配置
            ```cmake
            set_target_properties(PalmRegistry PROPERTIES
                QT_ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android"
            )
            ```
1. 坑：
    1. Qt安装路径下的：.\Qt\Tools\QtCreator\share\qtcreator\android\sdk_definitions.json，文件内记载了对安卓依赖的一些版本要求，这个要求很死板，必要的时候可以进行修改。要不然总会提示更新依赖包，实际上更新反而会导致错误。（比如升级了SDK，和JDK版本不兼容）
    1. 创建工程后，可能会发生Gradle与JDK版本不兼容，这里参照Android Studio，对构建目录下的gradle.properties进行修改，添加如下项
        ```properties
        org.gradle.java.home=C:/Program Files/Android/Android Studio/jre
        ```
    1. 提示qmlimportscanner不存在。**尚未解决**
## 参考资料
1. [Qt5官方文档](https://doc.qt.io/qt-5/classes.html)
1. [Qt事件机制](https://www.cnblogs.com/Braveliu/p/7417476.html)