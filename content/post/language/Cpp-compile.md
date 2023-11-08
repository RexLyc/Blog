---
title: "C/C++：编译篇"
date: 2022-01-24T15:00:17+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
math: true
---
本文讲解编译构建过程中的一些进阶内容。以Linux GNU/GCC为主，也会包含部分其他编译器内容。
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
1. 输入：.c/cpp/cxx/hpp，输出一般写作.i/.ii。
1. 处理内容：
    - 删除注释
    - #include：包含头文件，有<>和\"\"两种，前者顺序为先搜索编译器选项-I目录、后搜索默认的标准库目录，后者在这两个前面再优先搜索当前文件所在目录。预编译会将头文件在源文件中展开。
    - #define：宏定义
    - #if/ifdef/ifndef/elif/else/endif：条件编译
    - 添加行号和文件名标识
    - **保留#pragma等**编译器需要的指令
### 编译
1. 输入：编译单元，即一个可以被编译的文件，如.c、.cpp、.hpp。.h文件不算。实际上进入编译阶段的，已经是由预处理器处理过的源文件了。现阶段预编译和编译一般已经合并为一个步骤。输出一般写作.s。
1. 处理内容：词法分析、语法分析、语义分析、中间语言生成、优化、生成汇编代码
### 汇编
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
    | .interp | 动态链接器路径 |
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
1. 动态链接：
    1. 静态链接的问题：将会增大程序的存储体积，也会增大程序运行时的内存占用。静态库升级必须重新编译。
    1. 共享库（shared library/object）：一个目标文件，在运行或加载时可以被加载到任意的内存地址，并和一个内存中的程序链接起来。这个过程就是动态链接（dynamic linking）。
        - linux下为so、windowsxia为dll
    1. 动态链接流程：
        1. 编译期指定，随程序加载：
            - 创建动态链接库：编译阶段，对于库创建动态链接库
            - 创建可执行目标文件：指定依赖的动态链接库，编译器将在可执行文件内记录这些动态链接库，包括重定位信息、符号表信息等
            - 程序加载：加载器检查到存在.interp节，加载并执行动态链接器，动态链接器重定位所有依赖的动态链接库到内存，并将程序中的相关符号引用进行重定位。
        1. 运行期动态加载：
            - 根据需求，调用系统提供的动态链接器接口，如Linux平台为函数dlopen
    1. 位置无关代码（Position Independent Code，PIC）：
        - 问题：为了能让不同进程共享同一份内存中的动态库，需要做一些额外的工作。首先只有代码段和只读数据是可以共享的，每个使用到的进程仍然需要独立的读/写数据块。这就需要编译、加载期做一些事情。
        - 编译选项：动态库本身是可以随意加载而不需要（也不应该）进行重定位的。这样就必须保证寻址仍能正确。
        - 寻址方式：（仅描述基本原理）
            1. 动态库的目标模块代码段内部，可以很方便的通过相对地址进行寻址
            1. 引用全局数据：利用目标文件加载后相对距离不变的特性，构建全局偏移量表（Global Offset Table，GOT），编译时使用偏移量寻址，每一个引用了共享模块的进程都有自己的GOT，偏移值在共享模块加载时填充到GOT中。
            1. 引用外部函数：类似的方式来构造GOT，但同时还构造过程链接表（Procedure Linkage Table，PLT）。对外部函数的寻址处理使用延迟绑定来提高性能，GOT中外部函数的初始值是其PLT条目的第二个指令。每一个能被动态链接的外部库中的函数，都会有一个自己的PLT条目，第一次调用时该条目中的代码将会负责将自己在运行期的实际地址修改GOT中该函数的对应条目，非首次调用时PLT内会直接调用该GOT地址。
            1. GOT是数据段的一部分，PLT是代码段的一部分
            > PIC设计巧妙，建议阅读《深入理解计算机系统》491-492页
### 装载
1. 概念
    1. 加载器：系统调用来启动加载器，加载器将可执行目标文件从磁盘复制到内存中，然后跳转到程序的第一指令或入口点。
        - 进程管理：加载器的工作和进程管理相关，子进程是父进程的复制，加载器会把子进程的内存删除干净，并将可执行程序的内容拷贝到子进程的内存中。
        - 虚拟内存系统：加载器加载可执行文件并非真正的加载，只有当CPU开始引用一个虚拟内存页时，才会真正从磁盘复制。
    1. 程序头部表：加载器需要使用的数据结构。ELF的连续的片段（chunk）将会被映射到连续的内存段。这种映射关系由程序头部表给出，可以通过objdump查看，主要就是标记每一个片段的大小，目标文件中的位置，映射到内存的地址，类型等
    1. 内存结构：以Linux x86-64为例。内核占用高地址$2^{64}-1$至$2^{48}$。之后是栈、堆、数据、代码、空白页。（空白页大小是0x400000，为了尽量保证空指针能触发空页异常，虽然通用的页是4KB，但仍设置为大页的4MB大小）
    <center><img src="/images/postImage/MemoryChunk.png" width="256" height="512" ></center>
### 库打桩
1. 库打桩（library interpositioning）：用来截获对共享库函数的调用
1. 经典用法：
    - 替换目标函数
    - 包装目标函数
    - 统计性能、追踪调用细节
1. 打桩方式：
    - 编译时：包装目标函数，并以#define等形式进行函数替换。用编译选项-I优先使用本地包装的头文件。
    - 链接时：指定编译选项--wrap，提示链接器变更符号引用的解析方式。
    - 运行时：修改LD_PRELOAD环境变量，指定共享库路径，优先加载这里的运行库。一般会由包装程序内部再以dlsym来动态加载原始目标函数。包装程序需要编译为动态链接库。
## C和C++互操作
1. C++中调用C代码：因为C++和C在链接层面上，会出现符号不一致的情况，因此如果C++想要调用C，必须进行一定的处理。才能使C语言生成的符号链接在C++的链接过程中可见。
   - 在C语言的头文件中
        ```c
        // MSVC官方示例
        // MyHeader.h 

        // 必须限定只在C++内调用C的时候才通过条件编译
        // C中没有extern "C"这种写法
        #ifdef __cplusplus
        extern "C" {  
        #endif

        #ifdef DLL_EXPORT
            // dllexport用于说明该函数，将要导出到dll为其他程序使用
            // 可以通过编译器选项，控制函数默认不导出
            __descspec(dllexport) void YourFunc();
        #else
            // dllimport用于说明该函数，在链接使用的时候是由外部引入
            // dllimport用于使用时
            // 推荐添加，但实际上不添加也可能通过
            __declspec(dllimport) void YourFunc();
        #endif

        #ifdef __cplusplus
        }
        #endif
        ```
    - 在C++中使用C代码
        ```cpp
        // 真正重要的，这一步指使该编译单元是由C语言产生
        extern "C" {
        #include "MyHeader.h"
        }
        ```
2. C中调用C++代码：比较麻烦，因为C++中存在类和重载等特性，需要对这些内容进行一定的包装。
    - C++中
        ```cpp
        
        ```
    - C中
        ```c
        ```
## 常用指令
### GCC & G++
1. 概述：gcc和g++分别是GNU旗下用于c和c++的编译器
2. 常用命令、选项示例：
    ```bash
    # 用-o指定输出文件的名字
    gcc main.c -o main
    # -E为只进行预编译
    gcc -E hello.c -o hello.i
    # -S为只进行编译
    gcc -S hello.i -o hello.s
    # -C（大写），生成文件保留注释，用于调试
    gcc -C hello.c -o hello.o
    # 直接调用汇编器as
    as hello.s -o hello.o
    # 或者由gcc代劳
    gcc -c hello.s -o hello.o
    # -c也支持直接从源码到目标文件
    gcc -c hello.c -o hello.o
    # -pipe，使用管道避免产生临时文件
    gcc -pipe -o hello.o hello.c
    # --warp指定链接时打桩
    gcc --wrap malloc --warp free -o hello hello.c myMalloc.o
    # 等价于
    gcc -Wl,--wrap,malloc -Wl,--wrap,free -o hello hello.c myMalloc.o
    ```
3. 其他实用编译选项：
    | 选项 | 功能 |
    | --- | --- |
    | -fno-common | 禁止多重定义符号，提示错误 |
    |-Werror | 将所有警告提升为错误 |
    | -w | 不生成任何警告信息 |
    | -Wall | 生成所有警告信息 |
    | -Wl | 将在此选项后面的字符串传递给**链接器**，每个逗号替换为一个空格 |
    | -Wa | 将在此选项后面的字符串传递给**汇编器**，规则同上 |
    | -fpic | 指示编译器生成位置无关代码（动态链接库） |
    | --wrap xxx | 将__warp_xxx的函数作为包装函数，在链接时绑定到xxx的符号引用上 |
    | -lxxx | 指定链接所需要的库 |
    | -ldl | 需要运行期实用动态链接器的程序，用此指示编译器链接dlfcn相关依赖库 |
    | -ansi | 关闭gnu c中和ansi c不兼容的部分。用来专门支持ansi c的特性。 |
    | -include | 添加指定的头文件 |
    | -Dxxx | 为代码添加#define xxx |
    | -Dxxx=yyy | 为代码添加#define xxx=yyy |
    | -Uxxx | 为代码添加#undef xxx |
    | -undef | 取消任何非标准宏的定义 |
    | -I路径 | 指定一个寻找头文件的路径 |
    | -L路径 | 指定库的寻找路径 |
    | -M、-MM、-MD | 将所依赖的所有源代码包含到目标文件的一系列不同效果的选项 |
    | -O0/1/2/3 | 四个优化级别，默认1。0为不优化，3最高。 |
    | -static | 禁止使用动态库，打包成全静态链接程序 |
    | -shared | 尽可能使用动态库 |
### GDB
1. gdb是一个基于命令行的，交互式的debug工具。对于没有gui界面的情况，可以使用。但现在已经是2022年啦，如果有gui，还是建议用gui吧。
2. 启用：
    ```sh
    # 编译时需要至少打开-g选项
    gcc -g main.c -o main
    # 使用gdb启动
    gdb main
    # 可以使用gdb调试一个正在运行的服务
    gdb 程序名 程序id
    ```
3. gdb常用指令：
    | 命令 | 简写 | 功能 |
    | --- | --- | --- |
    | 回车 |  |  重复执行上一条命令 |
    | list 行数/函数名 | l | 按顺序展示代码，每一次10行 |
    | start |  | 开始执行，停留在main的第一条语句 |
    | run | r | 连续执行，直到遇到断点或结束 |
    | continue | c | 继续执行，直到断点或结束 |
    | next | n | 执行下一句 |
    | step | s | 进入正在执行的函数内部（step in） |
    | until | | 运行直到跳出循环体 |
    | finish | | 一直执行直到跳出当前函数（step out） |
    | info | i | info有大量选项，观察变量、堆栈、寄存器等 |
    | info 变量名 | i | 查看变量的值 |
    | set var 变量名=变量值 | | 修改变量的值 |
    | print 表达式 | | 打印一个表达式 |
    | display 变量名 | | 程序每次停止都会显示该变量的值（监视）|
    | x/nbx 变量名 | | 查看变量名开始之后的n个字节（指针、数组） |
    | backtrace | bt | 查看调用栈 |
    | frame n | f | 查看调用栈中的第n个栈帧 |
    | break 行数/函数名 [if 条件表达式]| b | 设置断点 |
    | watch 变量名/表达式 | | 当程序访问指定变量的内存、表达式有变时触发断点 |
    | rwatch 变量名 | | 变量被读时断点 |
    | awatch 变量名 | | 变量被读写断点 |
    | ignore 断点号 n | | 忽略指定断点n次 |
    | info watchpoints/breakpoints | | 查看当前观察点、断点 |
    | condition 断点编号 断点条件 | | 为断点添加条件 |
    | catch catch/throw 类型 | | C++专用，发现一个catch或throw时触发 |
    | commands 断点号 </br> 指令列表 </br> end |  | 设置在指定断点时执行的一系列指令 |
    | clear 行数/函数名/文件名 | | 删除断点 |
    | delete breakpoints 断点编号 | | 删除断点 |
    | enable/disable breakpoints 断点编号 | | 启用/禁用断点 |
    | quit | q | 推出gdb |
4. gdb命令行选项：
    | 选项 | 简写 | 功能 |
    | -tui | - | 用tui展示调试页面 |
5. 条件断点的使用说明
    ```sh
    # 创建普通断点
    (gdb) break 10
    # 注意gdb返回的断点编号为1
    breakpoints 1 at 0x....  file ..., line 10
    # 添加断点1条件，value不小于0
    (gdb) condition 1 value>=0

    # 类似的，观察断点
    (gdb) watch value
    breakpoints 2 at ....
    (gdb) condition 2 value>=10

    # 类似的，异常断点
    (gdb) catch throw int
    break points 3 at ...
    (gdb) condition 3 value>100
    ```
## 常用项目级工具
### CMakeLists
1. 概述：CMake是一个跨平台的C/C++构建文件生成工具，也支持一些其他语言。但是最主要的功能还是给C/C++语言项目使用。其在Windows上一般生成Visual Studio工程，在Linux下一般生成Makefile。
2. 用法：
    1. 编写CMakeLists.txt
    2. 编写对应的工程
    3. 惯用方式
        ```bash
        mkdir build
        cd build
        cmake ../
        make
        ```
3. 以一个典型的CMakeLists.txt为例
    ```sh
    # 指定所需要的cmake的最低版本
    cmake_minimum_required(VERSION 3.10)
    # 设置C++编译环境需要支持支持标准11
    SET(CMAKE_CXX_STANDARD 11)
    SET(CMAKE_CXX_STANDARD_REQUIRED True)
    # 设置项目名称
    PROJECT(Tutorial VERSION 1.0)

    FIND_PACKAGE(Qt5 )

    # 系统内省，判断是否能满足项目对平台或语言特性的需要
    CHECK_CXX_SOURCE_COMPILES("
        #include<cmath>
        int main(){
            std::exp(1.0);
            return 0;
        }
    " HAVE_EXP)

    IF(HAVE_EXP)
        MESSAGE("EXP CHECK PASS")
    ENDIF()

    # 编译选项
    OPTION(USE_MYDEP "使用自定义dep" ON)

    IF(USE_MYDEP)
        # 包含子文件夹dep的构建步骤
        ADD_SUBDIRECTORY(dep)
        # 将头文件和依赖库添加进来
        LIST(APPEND EXTRA_LIBS dep)
        LIST(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/dep")
    ENDIF()

    # 根据需要引入到代码中
    # CONFIGURE_FILE(...)

    # 添加生成的目标文件
    ADD_EXECUTABLE(Tutorial tutorial.cpp)

    # 指定目标文件所需的链接库
    target_link_libraries(Tutorial ${EXTRA_LIBS})

    # 指定头文件的搜索路径
    target_include_directories(Tutorial "${PROJECT_BINARY_DIR}" ${EXTRA_INCLUDES})

    # 会被安装的文件
    SET(export_targets Tutorial)
    SET(exports_includes Tutorial.h)
    # 安装类型，目的地
    INSTALL(TARGETS ${export_targets} DESTINATION bin)
    INSTALL(FILES ${exports_includes} DESTINATION include)
    ```
4. 如果需要将宏从CMake引入到代码中，则需要配合专门指令
    ```sh
    # CMake文件内
    # .in文件需要一定的编写，.h则会由cmake自动生成
    CONFIGURE_FILE(BuildConfig.h.in BuildConfig.h)
    ```
    ```c
    // BuildConfig.h.in内
    // 获取版本
    #define VERSION_MAJOR @Tutorial_VERSION_MAJOR@
    #define VERSION_MINOR @Tutorial_VERSION_MINOR@
    // 获取选项宏
    #cmakedefine USE_MYDEP
    ```
5. 一些基础指令指路：
    | 名称 | 功能 |
    | --- | --- |
    | PROJECT | 设定项目名称、版本 |
    | ADD_EXECUTABLE | 设定输出的目标可执行文件 |
    | ADD_LIBRARY | 设定输出的目标库文件(可配置库类型) |
    | TARGET_COMPILE_FEATURES | 指定目标文件的编译特性 |
    | FIND_PACKAGE | 查找必要的依赖包 |
    | INSTALL | 安装配置 |
    | IF | 条件编译语句 |
    | MESSAGE | 打印信息 |
    | SET | 设置普通、用户可修改、不可修改、环境变量 |
    | ADD_CUSTOM_COMMAND | 执行用户自定义脚本 |
    | EXPORT | 将本项目导出（生成.cmake配置），给其他项目使用 |
6. cmake中的各种内置变量、命令行选项非常重要，可以多看多了解
### Makefile
&emsp;&emsp;2022年了，目前还是建议使用CMake等跨平台生成方案。在Linux系平台上，CMake将会生成Makefile。
### Visual Studio
&emsp;&emsp;2022年了，目前还是建议使用CMake等跨平台生成方案。在Windows系平台上，CMake将会生成vxcproj/sln等文件。
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
2. CMake
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
- strings：列出目标文件中**所有**可以打印的字符串
    > 包括用户的常量字符串，以及各个节的名字，函数名、文件名等等
- nm：列出目标文件的符号表中的符号
- size：列出目标文件中节的名字和大小
- strip：删除目标文件中的非必要符号表信息，如debug等辅助信息
- ldd：列出所有该可执行文件运行所需要的共享库
- windows下：
    - cl：msvc编译器
    - link：msvc链接器
    - dumpbin：msvc的COFF/PE文件查看器
## 参考资料
1. 《程序员的自我修养：链接、装载与库》
2. 《深入理解计算机系统（第三版）》第七章链接
3. [CMake官方文档](https://cmake.org/cmake/help/latest/index.html)
4. [CMake入门实战](https://www.hahack.com/codes/cmake/)
5. [CMake官方教程](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)