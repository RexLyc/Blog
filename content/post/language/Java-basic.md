---
title: "Java系列：基础难点总结篇"
date: 2022-08-23T11:17:29+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Java系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/java.jpg
---
本文总结一些常见的较为特别的Java基础要点
<!--more-->
## 类型大小
1. 基本类型不再复述，记得char是两个字节
1. 类类型的大小分为四个部分：
    - 对象头：包含jvm信息（如gc、锁），和计算机字长相等
    - oop指针：指向类型信息：开启指针压缩是4个字节，否则是8个
    - 实际数据成员：
    - 对齐区域：默认对齐到8个字节的倍数
## 语言特性
1. 序列化：
    - 一般写法：继承Serializable接口，提供一个serialVersionUID
    ```java
    public class MyEntity implements java.io.Serializable {
        // 该UID可以用于序列化内容的版本控制
        private static final long serialVersionUID=1L;
    }
    ```