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
draft: true
---

<!--more-->
## 语言特性
1. 序列化：
    - 一般写法：继承Serializable接口，提供一个serialVersionUID
    ```java
    public class MyEntity implements java.io.Serializable {
        // 该UID可以用于序列化内容的版本控制
        private static final long serialVersionUID=1L;
    }
    ```