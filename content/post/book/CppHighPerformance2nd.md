---
title: "C++高性能编程读书笔记"
date: 2024-05-10T14:40:42+08:00
categories:
- 计算机科学与技术
- C++
tags:
- C++
- 暂停施工
- 读书笔记
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpphighperf.jpg
draft: true
---
本笔记记录的是（瑞典）比约恩·安德里斯特版的，C++ High Performance，第二版。侧重在多种场景（数据结构、并行算法、协程）下的高性能代码编写。
<!--more-->

## 基础补充
1. auto的各种用法：返回值自动推导，类型自动推导，模板参数推导```decltype(auto)```
1. 成员函数的const重载和&&重载。例如```void func() const {}```，```void func() && {}```。
1. 可以适当使用值传参。在接口有性能问题的时候，再进行优化。
    ```cpp
    // 将字符串小写化的接口，这种写法可以同时兼容左值和右值
    // 而且本来都要返回新字符串，传值是更好的设计
    auto string_to_lower(std::string s){
        for(auto &c:s) c = std::tolower(c);
        return s;
    }
    ```
1. 异常处理的原则
    1. C/C++缺少对契约的形式化方式。需要编码者自行明确编写代码的契约：调用者要保证先验条件，被调用者要满足后验条件，调用过程中满足不变式。

## 参考资料
[协程 by Lewis Baker](https://lewissbaker.github.io/)