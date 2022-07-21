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
math: true
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
## 基本原理
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
| .rel.text | 重定位条目，存储.text节中需要重定位的代码位置，即调用了外部函数和全局变量的位置 |
| .rel.data | 重定位条目，存储.data节中需要重定位的变量的信息 |
| .debug | 一个调试符号表，其条目存储了调试所需的类型信息，源代码等 |
| .line | 源代码和.text之间的映射关系 |
| .strtab | 字符串表，包括symtab、debug节中的符号表中符号的名称，以及各节的节名 |
> 并不是每个ELF文件都拥有全部的段，不使用-g时则没有debug/line段，可执行文件中一般则不包含重定位信息
### 链接
```bash
ld -o hello [依赖库] [各选项] hello.o 
```
1. 符号和符号表：
    1. 符号：对于可重定位文件，其符号表内存储着当前模块定义或者引用的符号的信息。主要有有三种符号
        - 全局符号：非静态的C函数和全局变量，C++中的类函数（包括静态）、静态成员变量，这些符号能被其他模块引用
        - 外部符号：由其他模块定义并被当前模块引用
        - 局部符号：当前模块定义并使用的符号，如静态的C函数、静态全局变量、静态局部变量
    1. 符号表项具体内容：符号表是由汇编器构造的，通常来说具有以下结构
        ```c
        typedef struct {
            int name;       // 在strtab表中的位置
            char type:4,    // 类型，函数还是数据
                bind:4;     // 局部还是全局
            char reserved;  // 保留
            short section;  // 所在节索引
            long value;     // 节中位置
            long size;      // 符号对象大小（字节）
        } Elf64_Symbol;
        ```
    1. 符号重整：由于C++允许函数重载，因此很多方法具有相同的名字，为了链接器能够有效区分，编译器将会把所有的类、函数名、变量都进行重整，基本的方式是名称长度+原始名字+参数缩写（v，i，l），例如6Person7getNamev，注意重整名称并不包括返回类型，也正因如此，返回类型不能作为重载条件。
1. 内部链接对象、外部链接对象
    1. 内部链接对象（Internal Linkage）：例如使用static标记的变量，此时各个编译单元（即各个.o），各自使用该变量的不同副本。该副本对其他编译单元不可见。
    1. 外部链接对象（External Linkage）：如果不同编译单元同时引用一个实例，即共享同一个地址的变量/函数，则是外部链接对象。
    > inline的一个用法：当头文件内出现全局函数时，避免出现重复定义，可使用inline标记为内部链接对象。同时也建议编译器将该函数直接嵌入在调用处。
1. 静态链接：
    1. 地址和空间分配（Address and Storage Allocation）
    1. 符号解析（Symbol Resolution）
        1. 局部符号的解析是显而易见的，只需要保证定义且唯一即可
        1. 外部符号的解析相对复杂。一般的符号将会被划分为强弱两种类型：
            - 强符号：函数和已初始化的全局变量
            - 弱符号：未初始化的全局变量
            - 一般的匹配规则：不允许多个同名强符号、强符号和强符号同名选强符号、多个弱符号同名则任选其一
            > 可以通过选项控制这些默认规则的行为，例如将这些编译、链接警告提升为错误
        1. 解析过程：链接器维护一个可重定位目标文件集合E，一个未解析符号集合U，一个已定位符号集合D。对于输入处理的目标文件（.o）和存档文件（.a），将会按顺序添加、更新E、U、D三个集合
            > 目标文件和存档文件的顺序可能非常重要，如果解析一个存档文件时，其符号均不在U集合内，则该存档文件将会被直接跳过，即使后面有需要，也会产生“无法解析的符号”的错误
    1. 重定位（Relocation）
        1. 重定位条目数据结构：
            ```c
            typedef struct {
                long offset;        //需要重定位的引用在所在节中的偏移量
                long type:32,       //重定位类型
                     symbol:32;     //需要重定位的符号在符号表中的索引
                long addend;        //重定位地址偏移量
            } Elf64_Rela;
            ```
            > 编译阶段，每当汇编器发现一个无法确定位置的引用时，就会产生一个重定位条目。重定位条目和其引用所在的节是对应的。
        1. 重定位节和符号定义：链接器将来自所有输入文件的相同类型的节，合并到输出文件的该类型节中。例如各个文件的.data合并为一个大的.data，所有.rodata合并为一个.rodata。从这一步开始，函数、变量都具有自己真正的地址。
        1. 重定位节中的符号引用：链接器处理每一个重定位条目，修改其指向的代码节或数据节中各种符号的实际引用值，让他们指向正确的地址。典型的重定位地址计算包括相对地址、绝对地址两种。
        > 可以理解代码就是一个字节流，其中指定位置的字节代表了符号引用的实际值，需要在链接期进行修改
        > 强烈推荐阅读《深入理解计算机系统》480-482页
1. 动态链接
### 装载
1. 概念
    1. 进程管理、虚拟内存系统：
    1. 加载器：系统调用来启动加载器，加载器将可执行目标文件从磁盘复制到内存中，然后跳转到程序的第一指令或入口点。该复制并不是真正的复制，只有当CPU开始引用一个虚拟内存页时，才会真正从磁盘复制。
    1. 程序头部表：加载器需要使用的数据结构。ELF的连续的片段（chunk）将会被映射到连续的内存段。这种映射关系由程序头部表给出，可以通过objdump查看，主要就是标记每一个片段的大小，目标文件中的位置，映射到内存的地址，类型等
    1. 内存结构：以Linux x86-64为例。内核占用高地址$2^{64}-1$至$2^{48}$。之后是栈、堆、数据、代码、空白页。（保留0x400000大小是为了尽量保证空指针能触发空页异常，虽然通用的页是4KB，但仍设置为大页的4MB大小）
    
## 常用指令
### GCC & G++
1. 实用编译选项：
    1. -fno-common：禁止多重定义符号，提示错误
    1. -Werror：将所有警告提升为错误
### GDB
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
## 常用项目级工具
### CMakeLists
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

### Makefile
### Visual Studio

## 其他构建工具
### 测试工具
1. Google Test
    - 一个Google开源C++单元测试框架
    - 参考：[玩转GoogleTest](https://www.cnblogs.com/coderzh/archive/2009/04/06/1426755.html)
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
1. CMake
    1. SET(CMAKE_INSTALL_PREFIX)无效：和SET本身有关，可以使用SET(CMAKE_INSTALL_PREFIX "/your/path" CACHE PATH "注释" FORCE)。[方法来源](https://stackoverflow.com/questions/39481958/setting-cmake-install-prefix-from-cmakelists-txt-file)
        - 实际上应当尽量不使用FORCE，应该用链接中的方法。    
## 其他常用指令
- objdump
| 选项 | 含义 |
| --- | --- |
| -h | 查看简略的各段的头部信息 |
| -x | 查看各段较详细信息，符号表单独显示 |
| -s | 以十六进制打印 |
| -d | 输出反汇编代码 |
- readelf：看符号表更好用一些
| 选项 | 含义 |
| --- | --- |
| -s | 查看符号表 |
| -e | 查看节头信息 |
- ar：从目标文件(.o)创建静态库(.a)，ar \[options\] archive_file object_files ...
| 选项 | 含义 |
| --- | --- |
| r | 创建、替换已有.a或向其插入新内容 |

## 参考资料
1. 《程序员的自我修养：链接、装载与库》
1. 《深入理解计算机系统（第三版）》第七章链接
1. [CMake官方文档](https://cmake.org/cmake/help/latest/index.html)
1. [CMake入门实战](https://www.hahack.com/codes/cmake/)