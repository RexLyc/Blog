---
title: "C/C++：编译篇"
date: 2022-01-24T15:00:17+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
- 开坑篇
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
---
本文讲解编译构建过程中的一些进阶内容
<!--more-->
# 编译器版本
1. MSVC：微软的C++版本。
1. GNU/GCC：GNU系列的版本，在Linux系列操作系统中广泛使用
1. Clang：C系语言的LLVM前端。
    - LLVM：开源编译器后端。[官方网站](https://llvm.org/docs/tutorial/index.html)。
# 编译、链接、装载
1. 编译：
    1. 编译单元：一个可以被编译的文件，如.c和.cpp。而.h文件不算。
        - .hpp也是一个编译单元。
# CMakeLists
1. 概述：CMake是一个跨平台的C/C++构建文件生成工具，也支持一些其他语言。但是最主要的功能还是给C/C++语言项目使用。其在Windows上一般生成Visual Studio工程，在Linux下一般生成Makefile。
1. 用法：
    1. 编写CMakeLists.txt
    2. 编写对应的工程
    3. 惯用方式
    ```bash
    mkdir build
    cd build
    cmake ../
    make
    ```
1. 以一个典型的CMakeLists.txt为例
```bash
cmake_minimum_required(VERSION 3.10)

# set the project name
project(Tutorial)

# add the executable
add_executable(Tutorial tutorial.cxx)
```
1. 一些坑
    - SET(CMAKE_INSTALL_PREFIX)无效：和SET本身有关，可以使用SET(CMAKE_INSTALL_PREFIX "/your/path" CACHE PATH "注释" FORCE)。[方法来源](https://stackoverflow.com/questions/39481958/setting-cmake-install-prefix-from-cmakelists-txt-file)
        - 实际上应当尽量不使用FORCE，应该用链接中的方法。
1. 参考资料
    - [官方文档](https://cmake.org/cmake/help/latest/index.html)
    - [CMake入门实战](https://www.hahack.com/codes/cmake/)
# Makefile
# 其他构建工具
## 测试工具
1. Google Test
    - 一个Google开源C++单元测试框架
    - 参考：[玩转GoogleTest](https://www.cnblogs.com/coderzh/archive/2009/04/06/1426755.html)
# 散点
1. 内部链接对象、外部链接对象
    1. 内部链接对象：不同编译单元引用同名同类型的内容，但其实是不同的实例，则是内部链接对象
    1. 外部链接对象：如果不同编译单元同时引用一个实例，则是外部链接对象。
    1. inline：当头文件内出现全局函数时，避免出现重复定义，可使用inline标记为内部链接对象。同时也建议该函数直接嵌入在调用处。
    1. static：多种语义，在这里代表声明当前内容的作用域为当前文件。对其他文件不可见。
# 一些坑
1. Linux/Ubuntu
    1. 使用cmake时，可能会出现错误：check for working C compiler: /usr/bin/cc -- broken
        ```bash
        # 尝试安装必要的包
        sudo apt install build-essential
        # apt安装可能会失败，提示有无法满足的依赖
        # 如上述失败，可以换用aptitude，更智能的包管理
        sudo apt install aptitude
        sudo aptitude install build-essential
        # 接下来会提示一些可行的解决方案，升级、降级等谨慎选择
        ```
# 参考资料
1. 《程序员的自我修养：链接、装载与库》
