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
本文讲解编译构建过程中的一些进阶内容。以GNU/GCC为主，也会包含部分其他编译器内容。
<!--more-->
## 编译器版本
1. MSVC：微软的C++版本。
1. GNU/GCC：GNU系列的版本，在Linux系列操作系统中广泛使用
    - gcc和g++都是compiler driver，分别负责调用真正的编译器cc1和cc1plus、汇编器as、链接器ld。
    - gcc曾经是单指c语言编译器，但现在，它指的是GNU Compiler Collection，一系列编译器。
1. Clang：C系语言的LLVM前端。
    - LLVM：开源编译器后端。[官方网站](https://llvm.org/docs/tutorial/index.html)。
## 编译、链接、装载
### 预编译
```bash
# -E为只进行预编译
gcc -E hello.c -o hello.i
```
1. 输入：.c/cpp/cxx/hpp，输出一般写作.i/.ii。
1. 处理内容：
    - 删除注释
    - #include：包含头文件，有<>和\"\"两种，前者顺序为先搜索编译器选项-I目录、后搜索默认的标准库目录，后者在这两个前面再优先搜索当前文件所在目录。预编译会将头文件在源文件中展开。
    - #define：宏定义
    - #if/ifdef/ifndef/elif/else/endif：条件编译
    - 添加行号和文件名标识
    - **保留#pragma等**编译器需要的指令
### 编译
```bash
# -S为只进行编译
gcc -S hello.i -o hello.s
```
1. 输入：编译单元，即一个可以被编译的文件，如.c、.cpp、.hpp。.h文件不算。实际上进入编译阶段的，已经是由预处理器处理过的源文件了。现阶段预编译和编译一般已经合并为一个步骤。输出一般写作.s。
1. 处理内容：词法分析、语法分析、语义分析、中间语言生成、优化、生成汇编代码
### 汇编
```bash
# 直接调用汇编器as
as hello.s -o hello.o
# 或者由gcc代劳
gcc -c hello.s -o hello.o
# -c也支持直接从源码到目标文件
gcc -c hello.c -o hello.o
```
1. 输入：汇编文件。输出目标文件。
1. 处理内容：就是把汇编文件，转换成当前平台的机器码。输入的汇编文件实际上仍然是文本文件，但目标文件就是二进制文件了。
### 目标文件
1. 编译后的文件类型：PC平台上的可执行文件大致分为Windows的PE（Portable Executable），和Linux的ELF（Executable Linkable Format）。目标文件（.obj/.o）只是尚未进行链接的文件，但内容上和可执行文件是高度类似的。
1. ELF标准下的文件划分
| 文件类型 | 含义 | 例子 |
| --- | --- | --- |
| 可重定位文件 | 包含代码和数据，只在编译期链接 | 目标文件.o、静态链接库.a |
| 可执行文件 | 可以直接执行 | bash |
| 共享目标文件 | 包含代码和数据，编译期、运行期都可以进行链接 | 动态链接库.so |
| 核心转储文件 | 保存的进程内存数据 | core dump |
> PE和ELF都是基于Unix System V Release 3提出的COFF格式发展出来的。
1. 核心概念：段（Segment）
| 重要的段的名称 | 含义 |
| --- | --- |
| .text | 代码段 |
| .rodata | 只读数据（如字符串常量、条件语句的跳转表） |
| .data | 已初始化的全局变量和静态C变量 |
| .bss | 未初始化的全局变量和静态C变量 |
| .symtab | 符号表，存放程序中定义和引用的函数及全局变量 |
| .rel.text | 存储.text节中需要重定位的代码位置，即调用了外部函数和全局变量的位置 |
| .rel.data | 存储.data节中需要重定位的变量的信息 |
| .debug | 一个调试符号表，其条目存储了调试所需的类型信息，源代码等 |
| .line | 源代码和.text之间的映射关系 |
> 并不是每个ELF文件都拥有全部的段，不使用-g时则没有debug/line段，可执行文件中一般则不包含重定位信息
### 链接
```bash
ld -o hello [依赖库] [各选项] hello.o 
```
1. 静态链接：
    1. 地址和空间分配（Address and Storage Allocation）
    1. 符号解析（Symbol Resolution）
    1. 重定位（Relocation）
1. 动态链接

    
## CMakeLists
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
1. 一些指令指路：
    1. PROJECT()
    1. ADD_EXECUTABLE()
    1. FIND_PACKAGE()
    1. INSTALL
    1. IF()
    1. MESSAGE()
    1. SET()
    1. ...
1. 一些坑
    - SET(CMAKE_INSTALL_PREFIX)无效：和SET本身有关，可以使用SET(CMAKE_INSTALL_PREFIX "/your/path" CACHE PATH "注释" FORCE)。[方法来源](https://stackoverflow.com/questions/39481958/setting-cmake-install-prefix-from-cmakelists-txt-file)
        - 实际上应当尽量不使用FORCE，应该用链接中的方法。
1. 参考资料
    - [官方文档](https://cmake.org/cmake/help/latest/index.html)
    - [CMake入门实战](https://www.hahack.com/codes/cmake/)
## Makefile
## GDB
1. 启用：gcc -g
1. 常用指令：
    1. break
    1. delete
    1. clear
    1. condition
    1. commands
    1. run、next、stop
    1. bt
    1. print
    1. info
    1. reg
## 其他构建工具
### 测试工具
1. Google Test
    - 一个Google开源C++单元测试框架
    - 参考：[玩转GoogleTest](https://www.cnblogs.com/coderzh/archive/2009/04/06/1426755.html)
## 散点
1. 内部链接对象、外部链接对象
    1. 内部链接对象：不同编译单元引用同名同类型的内容，但其实是不同的实例，则是内部链接对象
    1. 外部链接对象：如果不同编译单元同时引用一个实例，则是外部链接对象。
    1. inline：当头文件内出现全局函数时，避免出现重复定义，可使用inline标记为内部链接对象。同时也建议该函数直接嵌入在调用处。
    1. static：多种语义，在这里代表声明当前内容的作用域为当前文件。对其他文件不可见。
## 一些坑
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
## 其他常用指令
- objdump
| 选项 | 含义 |
| --- | --- |
| -h | 查看简略的各段的头部信息 |
| -s | 以十六进制打印 |
| -d | 输出反汇编代码 |
- 
## 参考资料
1. 《程序员的自我修养：链接、装载与库》
1. 《深入理解计算机系统（第三版）》第七章链接