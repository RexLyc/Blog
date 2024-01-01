---
title: "《现代C++教程》：读书笔记"
date: 2023-12-27T12:25:30+08:00
categories:
- 读书笔记
- 技术书
tags:
- C/C++
- 框架
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
# draft: true
mermaid: true
---
从C++11开始，C++开始加速向一门更现代的语言进化。很多需求都有了更优秀的写法来替代原来的技巧。
<!--more-->
## 标准更迭
1. 实际上C++并不是C的超级，从一开始的标准就无法完全做到。在代码中应当使用宏和```extern```来严格标记混合使用的位置。参考[C和C++互操作]({{<relref "/content/post/language/Cpp-compile.md#C和C++互操作">}})。


## 可用性优化
```constexpr```：指示编译器该表达式、函数，能够在编译期成为常量表达式，应该尝试去求解其数值。有以下几点值得注意
- ```constexpr```和```const```在语义上的区别，前者说明是一个常量表达式，后者说明是一个不可修改的常量（但其值则可能是运行期给出）。
- 在C++11初次引入时，常量表达式内可以递归，但仍不能使用循环、分支、定义局部变量等简单语句。从C++14开始允许使用。

允许在```if/switch```语句作用域内定义临时变量，例如
```cpp
// 先定义，以 ; 分割
// if(Type t; boolean statement)
if (const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 3);
    itr != vec.end()) {
    *itr = 4;
}
```

初始化列表```std::initializer_list<>```，从C++11开始，为了给类初始化提供近似于POD类型的写法，提出的初始化列表。这使得具有支持初始化列表构造器的类型，可以和POD类型一样，使用```{}```进行初始化，例如
```cpp
class MyClass{
public:
    MyClass(std::initializer_list<int> list) { /* ... */}
}

int a[]={1,2,3,4};
MyClass myClass = {1,2,3,4};

```