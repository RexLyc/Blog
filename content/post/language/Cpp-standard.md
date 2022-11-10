---
title: "C/C++：标准篇"
date: 2022-11-08T23:07:29+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
- 持续施工
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
math: true
---
语言标准是语言发展的重要基础，这一点在C/C++上尤为明显，从C++11开始，C++标准委员基本以3年为一个周期快速更新C/C++，这门语言也终于焕发越来越强的活力。
<!--more-->
## 概述
&emsp;&emsp; 标准是繁杂且专业的内容，本文仅记录一些我认为比较有学习价值的标准内容。
## C标准
1. K&R C：大佬Dennis Ritchie和Brian Kernighan合作的书《The C Programming Language》的参考指南中提到的C语言完整定义，成为当时（1978年）的事实标准。
1. ANSI C和GNU C
    - GNU C是GNU计划在开发GNU项目GCC、Linux过程中制定的一个C标准
    - ANSI C则是美国国家标准协会（ANSI），在1989年提出的C标准：C89。该标准随后在1990年被ISO修订后接纳为标准，即C90。因此C89、ANSI C、C90几乎相同
    - 一些对比
        | 特性 | 代码示例 | GNU C | ANSI C |
        | --- | --- | --- | --- |
        | 零长度和可变长度数组 | int x[n],y[0] | 支持 | 不支持 |
        | 范围case | case 0 ... 10 : /\*...\*/ | 支持 | 不支持 |
        | 括号语句表达式 | #define min(type,x,y) ({/\*...\*/}) | 支持 | 不支持 |
        | typeof关键字 | const typeof(x) _x=(x); | 支持 | 不支持 |
        | 可变参数宏 | #define myPrint(fmt,arg...)\\</br>&emsp;printf(fmt,##arg) | 支持 | 不支持 |
        | 标号元素 | int a[5]={[2 ... 4]=-1,[1]=0,[0]=1} | 支持 | 不支持 |
        | 函数、变量、类型特殊属性 | \_\_attribute\_\_((各种属性)) | 支持 | 不支持 |
        | 内建函数 | - | - | - |
        >  注意&emsp;...&emsp;前后必须有空格
    - GNU C和后续的标准C：一般来说GNU会扩展一些用法，也会出现一些不完全支持的情况。
1. C89、C95、C99
    - C89的主要目标是建立一个“无歧义、平台无关”的C语言定义
    - C95是一些印刷更正和小的技术勘误
    - C99提供了更多的技术改进，如
        - 可变长数组、单行注释、for语句内的变量声明、结构体变量初始化
        - printf&scanf增强
        - 新增了复数/浮点/字符方面的头文件和库、\_\_func\_\_预定义宏
        - 放宽/加强了一些技术上的编译限制、禁止非void函数不写return
        - 引进long long int、规范整型向上转型规则
1. C11：[标准变更](https://zh.cppreference.com/w/c/11)，如
    - 弃用gets
    - 更好的多线程支持
    - 增强的内存对齐支持
    - 匿名union、struct成员
    - 细粒度的求值顺序
    - 可分析性
    - unicode支持
1. C17：主要是解决一些标准缺陷
1. C23：还没出呢
## C++标准
### 重要语法特性
1. C++98
1. C++03
1. C++11
1. C++14
1. C++17
1. C++20
1. C++23
1. 编译器支持情况（截止2022年底）

## 参考
1. [cppreference](https://zh.cppreference.com/w/)
1. [C89和C99区别](https://www.cnblogs.com/xiaoyoucai/p/6146784.html)