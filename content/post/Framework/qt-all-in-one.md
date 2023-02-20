---
title: "Qt合集"
date: 2022-02-09T15:28:10+08:00
categories:
- 计算机科学与技术
- Qt
tags:
- 滚动更新
- Qt
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/qt-logo.svg
---
Qt是一个非常好用的C++框架，最主要的用途是用来开发跨平台GUI程序。
<!--more-->
本文主要记录一些开发过程中遇到的经典坑和关键理解。以高版本（不低于5.4）为例。
## 最佳实践
## 核心机制关键词
1. 信号和槽
    1. 观察者模式：信号和槽本质上就是观察者模式的一种实现
    1. 转换：在moc过程中转换为标准的C++代码
    1. QObject：只有QObject派生类，且添加了Q_OBJECT宏才可以使用该机制
    1. public：signals不能添加限定符，默认就是public的，信号只能声明，不能定义，返回值必须为void
    1. 参数：信号的参数量不能小于槽函数的参数量
    1. 级联：信号可以级联，信号将会连接顺序进行传递
1. MOC
    1. Meta-Object Compiler：元对象编译器
1. UIC
1. QMake
1. 国际化

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
- Linux下并没有自带部署工具，但由于Linux的特殊性，只需实用ldd查看依赖项并进行拷贝即可，拷贝步骤可以放在CMake种进行

## 参考资料
[Qt5官方文档](https://doc.qt.io/qt-5/classes.html)