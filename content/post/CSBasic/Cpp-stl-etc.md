---
title: "C/C++：标准库篇"
date: 2022-03-14T14:21:34+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
- 开坑篇
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
---
本章统一记录实用的C/C++标准库接口。或者其他实用标准下的接口。未特殊说明，仅针对C++11标准。
<!--more-->
# C++
1. 容器
    1. map：可逆、AllocatorAware、关联容器，键唯一，通常实现为红黑树
        ```c++
        // C++11
        template<
            class Key,
            class T,
            class Compare = std::less<Key>,
            class Allocator = std::allocator<std::pair<const Key, T> >
        > class map;
        ```
        - 元素访存：[]、at
        - 迭代器：begin/end、cbegin/cend、rbegin/rend、crbegin/crend
        - 容量：empty、size、max_size
        - 修改：clear、erase、swap
        - 查找：count、find、equal_range、lower_bound、upper_bound
# C