---
title: "Java系列：注解篇"
date: 2021-12-13T14:44:54+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Java系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/java.jpg
---
注解是Java所提供的一种特别的编程方式。从Java5被引入，在Spring框架中应用尤为广泛。
<!--more-->
# 为什么要使用注解
1. 注解能够将一些元数据直接写入代码当中。而不需要另写文档。
1. 代码更加易于阅读，并且拥有了编译期类型检查的能力。
1. 可以用于控制编译期代码的生成。
> 总之，一旦你的代码中出现了重复性的工作，你就可以考虑使用注解来简化、自动化该过程。
# 内置注解
1. java.lang中：
    1. @Override：表示当前方法将覆盖父类中的方法。
    1. @Deprecated：表示当前方法已被废弃，如果被使用，编译器会发出警告。
    1. @SuppressWarnings：关闭不恰当的编译器警告信息
1. 
# 基本语法
1. 