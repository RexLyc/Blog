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
## C++
1. 说在前面的一些重要经验
    1. 如果你发现程序抛出任何类似于：used after free、iterator can't dereference等问题。不要怀疑，你的迭代器一定是在使用时**早已非法化**了。由于STL中迭代器被广泛使用。请务必注意其可能导致非法化的任何场景。
    1. 优先使用emplace
    1. 优先使用数据结构自身的成员函数lower_bound、upper_bound、find，而不是使用通用的算法（受访问限制，性能可能会退化）。
1. 容器
    1. [map](https://zh.cppreference.com/w/cpp/container/map)：可逆、AllocatorAware、关联容器，键唯一，通常实现为红黑树
        ```c++
        // 定义于头文件<map>
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
    1. [set](https://zh.cppreference.com/w/cpp/container/set)：可逆、AllocatorAware、关联容器，键唯一，通常实现为红黑树
        ```cpp
        // 定义于头文件<set>
        template<
            class Key,
            class Compare = std::less<Key>,
            class Allocator = std::allocator<Key>
        > class set;
        ```
        - 常用构造函数：(first, last)、(compare)
        - 迭代器（不可修改元素内容）：begin/end、cbegin/cend、rbegin/rend、crbegin/crend
        - 容量：empty、size、max_size
        - 修改：clear、erase、insert、emplace、swap
        - 查找：count、find、equal_range、lower_bound、upper_bound
    1. multimap、multiset，允许重复的键。但也因此，不再支持```operator[]```。使用迭代器访问。
    2. [vector](https://zh.cppreference.com/w/cpp/container/vector)：可逆、AllocatorAware、序列、连续容器。扩容会调用拷贝构造。
        ```c++
        // 定义于头文件<vector>
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
    3. [list](https://zh.cppreference.com/w/cpp/container/list)：可逆、AllocatorAware、序列容器，通常实现为双向链表，支持O(1)的任意插入和删除，迭代器只有元素被删才失效
        ```c++
        // 定义于头文件<list>
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
    4. [forward_list](https://zh.cppreference.com/w/cpp/container/forward_list)：追求不输于C风格链表的极致效率，接口比list少了很多。单向链表。
    5. [array](https://zh.cppreference.com/w/cpp/container/array)：封装固定大小数组，不会退化为指针，推荐用于替代传统C数组。可逆，连续。
    6. [dequer](https://zh.cppreference.com/w/cpp/container/deque)：双端队列，由一系列连续的内存区域连接而成。有索引的序列容器。随机访问和首位插入删除是O(1)，中间插入删除是O(n)。
2. 容器适配器
    1. [stack](https://zh.cppreference.com/w/cpp/container/stack)
        ```cpp
        // 定义于头文件<stack>
        template<
            class T,
            class Container = std::deque<T>
        > class stack;
        ```
        - 常用构造函数：(first, last)
        - 元素访存：top
        - 容量：empty、size
        - 修改：push、pop、emplace、swap
    2. [priority_queue](https://zh.cppreference.com/w/cpp/container/priority_queue)：默认实现为大顶堆
        ```cpp
        // 定义于头文件<queue>
        template<
            class T,
            class Container = std::vector<T>,
            class Compare = std::less<typename Container::value_type>
        > class priority_queue;
        ```
        - 常用构造函数：(first, last)、(compare)
        - 元素访问：top（返回const_reference）
        - 容量：empty、size
        - 修改器：push、emplace、pop、swap
    1. [queue](https://zh.cppreference.com/w/cpp/container/queue)：容器适配器，队列
        ```cpp
        template<
            class T,
            class Container = std::deque<T>
        > class queue;
        ```
3. 算法\<algorithm\>
    1. [iter_swap](https://zh.cppreference.com/w/cpp/algorithm/iter_swap)：交换迭代器所指向元素的内容
        - 如果想交换两个迭代器，应该使用swap，而不是iter_swap
        - iter_swap可由swap实现
4. 工具\<utility\>
    1. [swap](https://zh.cppreference.com/w/cpp/algorithm/swap)：交换实例的内容
        - 存在大量特化
        - 右值是不能swap的！
    2. 生成器，例如```generate(xx.begin(), xx.end(), std::rand);```
    3. 容器间的拷贝，```copy```、```copy_if```。这个还支持输出流的迭代器，例如
        ```cpp
        std::vector<int> v(8);       
        std::generate(v.begin(), v.end(), std::rand);
        std::copy(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, "\n"));
        ```
    4. 
5. 迭代器\<iterator\>
    1. next、prev：返回下一个、上一个迭代器。
6. 字符串\<string\>
    1. to_string：重载函数，将int、long、long long、unsigned、unsigned long、unsigned long long、float、double、long double转为字符串
        - 转换后数字高位在字符串的0下标
        - 对浮点数并不保证完全正确（和cout输出可能不同）    
    2. [stoi/stol/stoll](https://zh.cppreference.com/w/cpp/string/basic_string/stol)、[stoul/stoull](https://zh.cppreference.com/w/cpp/string/basic_string/stoul)、[stof/stod/stold](https://zh.cppreference.com/w/cpp/string/basic_string/stof)：字符串转数字
        - 舍弃前缀空白字符，只取第一段数字
        - 无法转换或超出范围都会抛异常
7. IO
8. 并发
    1. 锁
    2. 线程
    3. 有锁数据结构
    4. 无锁数据结构
    5. promise
    6. future
    7. packaged_task

## Boost
Boost等实在是太过于重要，所以也有必要说一下

## C