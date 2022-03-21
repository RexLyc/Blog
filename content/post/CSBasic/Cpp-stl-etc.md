---
title: "C/C++：标准库篇"
date: 2022-03-14T14:21:34+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
---
本章统一记录实用的C/C++标准库接口。或者其他实用标准下的接口。未特殊说明，仅针对C++11标准。
<!--more-->
# C++
1. 容器
    1. [map](https://zh.cppreference.com/w/cpp/container/map)：可逆、AllocatorAware、关联容器，键唯一，通常实现为红黑树
        ```c++
        #include <map>
        
        template<
            class Key,
            class T,
            class Compare = std::less<Key>,
            class Allocator = std::allocator<std::pair<const Key, T> >
        > class map;
        ```
        - 常用构造函数：(compare)
        - 元素访存：[]、at
        - 迭代器：begin/end、cbegin/cend、rbegin/rend、crbegin/crend
        - 容量：empty、size、max_size
        - 修改：clear、erase、swap
        - 查找：count、find、equal_range、lower_bound、upper_bound
    1. [vector](https://zh.cppreference.com/w/cpp/container/vector)：可逆、AllocatorAware、序列、连续容器
        ```c++
        #include <vector>

        template<
            class T,
            class Allocator = std::allocator<T>
        > class vector;
        ```
        - 常用构造函数：(count,value)、(count)、(otherVector)、(first, last)
        - 元素访存：[]、at、front、back、data
        - 迭代器：begin/end、cbegin/cend、rbegin/rend、crbegin/crend
        - 容量：empty、size、max_size、reserve、capacity、shrink_to_fit
        - 修改（均可能造成部分迭代器无效）：clear、insert、emplace、erase、push_back、emplace_back、pop_back、resize、swap
    1. [list](https://zh.cppreference.com/w/cpp/container/list)：可逆、AllocatorAware、序列容器，通常实现为双向链表
        ```c++
        template<
            class T,
            class Allocator = std::allocator<T>
        > class list;
        ```
        - 常用构造函数：(count,value)、(first, last)
        - 元素访存：front、back
        - 迭代器：begin/end、cbegin/cend、rbegin/rend、crbegin/crend
        - 容量：empty、size、max_size
        - 修改（至多造成被删除位置迭代器失效）：clear、insert、emplace、erase、push_back、emplace_back、pop_back、push_front、emplace_front、pop_front、resize、swap
        - 其他操作：merge合并两个有序列表、splice从另一列表移动元素、reverse、unique删除连续重复元素、sort、remove、remove_if
1. 容器适配器
    1. [stack](https://zh.cppreference.com/w/cpp/container/stack)
        ```cpp
        template<
            class T,
            class Container = std::deque<T>
        > class stack;
        ```
        - 常用构造函数：(first, last)
        - 元素访存：top
        - 容量：empty、size
        - 修改：push、pop、emplace、swap
1. 算法\<algorithm\>
    1. [iter_swap](https://zh.cppreference.com/w/cpp/algorithm/iter_swap)：交换迭代器所指向元素的内容
        - 如果想交换两个迭代器，应该使用swap，而不是iter_swap
        - iter_swap可由swap实现
1. 工具\<utility\>
    1. [swap](https://zh.cppreference.com/w/cpp/algorithm/swap)：交换实例的内容
        - 存在大量特化
        - 右值是不能swap的！
1. 迭代器\<iterator\>
    1. next、prev：返回下一个、上一个迭代器。
# C