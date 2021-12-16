---
title: "Java系列：开坑篇"
date: 2021-12-13T14:21:41+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Java系列
- 开坑篇
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/java.jpg
---
本系列主要提供Java相关开发的一些难点的讲解。基础的语法尚需使用其他资料学习。
<!--more-->
# 历史
- 由Sun公司为家电设备的嵌入式应用开发而设计。目标是一处编译，处处运行。以C++为参考语言，去除不实用的部分和不安全的部分。
- 在寻求硬件厂商支持无果后。Java发现其设计目标对于网页开发也适用，从此搭上了互联网开发的东风。
- Sun公司目前已被甲骨文收购。
# 系列计划内容
1. 注解
1. JVM内存模型
1. JVM垃圾回收
1. 内部类
1. 反射
1. 泛型
# 配置时需要注意的内容
1. 环境变量
    - 由于部分环境会对空格敏感，因此在windows上建议安装至无空格，无特殊字符的目录。
    - 也可以通过创建链接的方式，将JAVA_HOME设置为链接目录，如
    ```bash
    # cmd（而非Powershell）
    mklink /J yourJavaLink "C:\Program File\Java\Jdk8\"
    set JAVA_HOME=X:\Path\To\yourJavaLink
    CLASSPATH=...
    PATH=...;%JAVA_HOME%\bin
    ```
# 参考文献
1. 《Java编程思想（第四版）》
1. 《Java核心技术》