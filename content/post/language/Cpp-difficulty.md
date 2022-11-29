---
title: "C/C++：难点总篇"
date: 2022-02-22T20:22:20+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
- 开坑篇
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
---
C/C++中有一些经常会出现的难点，也是考研功底的要点。本文致力于总结这一部分。
<!--more-->
## 指针篇
1. 函数指针
    1. C：
        1. 函数签名：即函数的类型，由返回类型+参数类型列表构成。
            ```cpp
            // 函数签名为int(int,int)
            int add(int a,int b);
            ```
        1. 创建一个函数指针，需要指明指向的函数的类型。pf附近的括号必须存在，否则变为声明一个返回指针的函数。
            ```cpp
            // 局部 & 全局变量
            int (*pf) (int, int);
            pf = add;
            pf(100,100); // 使用方式1
            (*pf)(100,100); // 使用方式2
            
            // 形参定义
            void func1(int pf(int, int)); // 写法1
            void func2(int (*pf)(int, int)); // 写法2
            ```
        1. 创建别名
            ```cpp
            typedef int (*PF) (int, int);
            PF pf_2 = add; // PF定义了一个类型，和类型用法相同
            ```
        1. 函数指针作为函数返回值
            ```cpp
            // 阅读方式从自定义名称向外，func是一个函数，形参为(int)
            // 返回一个int(int, int)类型的函数指针
            int (*func1(int))(int, int);

            // 当然并不建议以上写法，过于隐晦
            PF func2(int);
            ```
    1. C++：
        1. auto特性
            ```cpp
            int add(int,int);
            auto pf1 = add; // auto将会自动解析类型为函数指针
            // auto *pf1 = add; // 等价写法
            pf1(100,100); // 可以
            (*pf1)(100,100); // 可以
            ```
        1. 函数类型和函数指针类型。decltype仅解析到函数签名。
            ```cpp
            decltype(add)* pf2 = add; // 正确
            decltype(add) pf1 = add; // 错误，类型不对
            ```
        1. 形参
            ```cpp
            typedef decltype(add) func_add;
            typedef decltype(add)* funcP_add;
            void func1(func_add a);
            void func2(funcP_add a); // 错误，已有定义
            ```
        1. 成员函数指针
            ```cpp
            typedef void(A::*PF1)(); // 必须指定类型别名PF1所属的类
            PF1 pf1 = &A::func; // 必须有&
            A a;
            (a.*pf1)(); // 使用成员函数指针，需要和类型实例关联
            ```
            >> 由于成员函数指针需要获取类成员函数，因此没有多态性。取到哪个类，就是其对应的实现。即使关联了其子类的实例。
    1. 参考
        - [函数指针使用总结](https://www.cnblogs.com/lvchaoshun/p/7806248.html)似乎并不完全正确，**等待确认**。

## 模板编程
1. 类型萃取
    1. 参考
        - [type_traits类型萃取](https://www.cnblogs.com/gtarcoder/p/4807670.html)
1. 类型擦除
    1. 参考
        - [C++类型擦除和std::function性能测试](https://www.codercto.com/a/57707.html)

## 高性能IO
1. 流
    1. 参考：
        - [std::streambuf从示例到应用](https://izualzhy.cn/stream-buffer)

## 右值
1. [右值详解](https://www.cnblogs.com/jiu0821/p/7920837.html)
1. 右值相关的重载：先看一段程序
    ```cpp
    void print(string a) {}             // 1
    void print(string &a) {}            // 2
    void print(const string &a) {}      // 3
    void print(string &&a) {}           // 4
    void print(const string && a) {}    // 5
    int main() {
        // 重载错误，1/3/4/5均可
        print("hello world");
        return 0;
    }
    ```
    - 当这五个函数同时实现后，调用时会出现重载错误。
    - 重载函数的选择逻辑
        - 优先选择不需要做转型的重载函数
        - 右值不会自动绑定到非常量左值引用
        - 其他情况下优先情况未确定
    - 第五种几乎没有意义，不要这么用
    - c语言的历史遗留问题：const int和int不能作为重载函数的区分