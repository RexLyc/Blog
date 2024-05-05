---
title: "C/C++：最佳实践"
date: 2023-03-03T11:00:31+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
---
本篇记录一些C/Cpp开发中的一些最佳实践。
<!--more-->

## 写法推荐
1. 在内置类型（int、float），以及一些小型非基本类型（pair、complex）的传递中，使用```const auto```，而非```const auto &```。这是出于简化生命周期、编译器优化潜力、防止引用悬挂等问题的考虑。


## 资源获取和五法则
五法则是指：拷贝构造、拷贝赋值运算符、析构函数、移动构造、移动赋值运算符。其中后两者是在C++11之后提出的。在此之前，只有三法则。

从C++11开始，C++为一系列情况，提供了移动语义：无法拷贝、不必要的拷贝。
```cpp
class MyClass {
public:
    MyClass(MyClass&&) noexcept {/* ... */}

    MyClass& operator=(MyClass&&) noexcept {/* ... */}
}
```

移动语义只应该在有必要的时候才进行使用。如果类型中，不涉及到资源管理，那么移动就不一定有价值。比如类型中都是内置类型（int、float），对他们的移动和拷贝没有性能上的区别。

## noexcept
在移动构造、移动赋值运算符、析构函数中，鼓励添加noexcept，以提示编译器进行优化。当然如果你的程序里确实需要抛出异常，那就还是抛出来。


## STL
1. 智能指针：
    - weak_ptr：A、B类内各用weak_ptr保存对方，在其他地方使用A、B时可以随意使用shared_ptr、unique_ptr

## 实用工具
1. 一款能够查看C/C++编译的汇编代码和原始代码对比情况的工具：[godbolt.org](https://godbolt.org/)，也有[github仓库](https://github.com/compiler-explorer/compiler-explorer)可以自行部署

## 参考
1. [360 安全规则集合](https://github.com/Qihoo360/safe-rules)