---
title: 《C++20 高级编程》读书笔记
date: 2026-05-25T23:29:34+08:00
categories:
- 计算机科学与技术
- C++
tags:
- C++
- 读书笔记
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/book/performance_cpp_5th.png
draft: true

---
尝试认真读完一本C++砖头书！
<!--more-->

> 全书一共34章，900+页，计划是争取两个月内读完。

# 第一章 C++和标准库速成

> 这里只记录一些不太熟悉的新的东西

- C++11：
1. 数值极限，如：`numeric_limits<int>::max()`
2. 预定义变量：如：`__func__`
3. 属性：`[[nodiscard]]`、`[[maybe_unused]]`、`[[noreturn]]`、`[[deprecated]]`

- C++17：
1. 嵌套命名空间写法：`namespace rex::lyc::life { /* ...*/ }`，`namespace rexlycLife = rex::lyc::life;`
2. switch，if语句初始化器，如：`if (Employee employee {getEmployee()}; employee.xxx >= 100) {}`
3. switch中的`[[fallthrough]]`属性声明
4. 结构化绑定
5. optional

- C++20：
1. 基本类型：`const char8_t* v = u8'你好'`
2. 枚举使用简化：`using enum CustomType;`，或单独`using enum CustomType::A`
3. 模块：标准库模块`import <iostream>;`，自定义模块`import your_module;`
4. 三向比较操作符：`auto result = i <=> 0`
5. 基于范围的for循环的初始化器：`for (std::array arr { 1, 2, 3}; int i: arr)`