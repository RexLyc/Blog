---
title: "Qt合集"
date: 2022-02-09T15:28:10+08:00
categories:
- 计算机科学与技术
- Qt
tags:
- 持续施工
- Qt
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/qt-logo.svg
---
Qt是一个非常好用的C++框架，最主要的用途是用来开发跨平台GUI程序。
<!--more-->
本文主要记录一些开发过程中遇到的经典坑和关键理解。以高版本（不低于5.4）为例。
## 最佳实践
## 核心机制
1. 信号和槽
1. MOC
1. UIC
1. QMake
1. 国际化
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
### win
- 使用官方提供的windeployqt.exe
- 使用Qt Install Framework
- 一些坑
    - [Qt打包发布程序，解决找不到msvcp140.dll等动态库问题正确方案](https://blog.csdn.net/no_say_you_know/article/details/126360830)
    - [OpenGL环境检测和设置](https://blog.csdn.net/mvmmvm/article/details/122177404)
### linux

## 参考资料
[Qt5官方文档](https://doc.qt.io/qt-5/classes.html)