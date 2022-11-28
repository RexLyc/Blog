---
title: "C/C++：标准篇"
date: 2022-11-08T23:07:29+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- C/C++系列
- 持续施工
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
math: true
---
语言标准是语言发展的重要基础，这一点在C/C++上尤为明显，从C++11开始，C++标准委员基本以3年为一个周期快速更新C/C++，这门语言也终于焕发越来越强的活力。
<!--more-->
## 概述
&emsp;&emsp; 标准是繁杂且专业的内容，本文仅记录一些我认为比较有学习价值的标准内容。具体特性的使用还是要去参考文档。
## C标准
1. K&R C：大佬Dennis Ritchie和Brian Kernighan合作的书《The C Programming Language》的参考指南中提到的C语言完整定义，成为当时（1978年）的事实标准。
1. ANSI C和GNU C
    - GNU C是GNU计划在开发GNU项目GCC、Linux过程中制定的一个C标准
    - ANSI C则是美国国家标准协会（ANSI），在1989年提出的C标准：C89。该标准随后在1990年被ISO修订后接纳为标准，即C90。因此C89、ANSI C、C90几乎相同
    - 一些对比
        | 特性 | 代码示例 | GNU C | ANSI C |
        | --- | --- | --- | --- |
        | 零长度和可变长度数组 | int x[n],y[0] | 支持 | 不支持 |
        | 范围case | case 0 ... 10 : /\*...\*/ | 支持 | 不支持 |
        | 括号语句表达式 | #define min(type,x,y) ({/\*...\*/}) | 支持 | 不支持 |
        | typeof关键字 | const typeof(x) _x=(x); | 支持 | 不支持 |
        | 可变参数宏 | #define myPrint(fmt,arg...)\\</br>&emsp;printf(fmt,##arg) | 支持 | 不支持 |
        | 标号元素 | int a[5]={[2 ... 4]=-1,[1]=0,[0]=1} | 支持 | 不支持 |
        | 函数、变量、类型特殊属性 | \_\_attribute\_\_((各种属性)) | 支持 | 不支持 |
        | 内建函数 | - | - | - |
        >  注意&emsp;...&emsp;前后必须有空格
    - GNU C和后续的标准C：一般来说GNU会扩展一些用法，也会出现一些不完全支持的情况。
1. C89、C95、C99
    - C89的主要目标是建立一个“无歧义、平台无关”的C语言定义
    - C95是一些印刷更正和小的技术勘误
    - C99提供了更多的技术改进，如
        - 可变长数组、单行注释、for语句内的变量声明、结构体变量初始化
        - printf&scanf增强
        - 新增了复数/浮点/字符方面的头文件和库、\_\_func\_\_预定义宏
        - 放宽/加强了一些技术上的编译限制、禁止非void函数不写return
        - 引进long long int、规范整型向上转型规则
1. C11：[标准变更](https://zh.cppreference.com/w/c/11)，如
    - 弃用gets
    - 更好的多线程支持
    - 增强的内存对齐支持
    - 匿名union、struct成员
    - 细粒度的求值顺序
    - 可分析性
    - unicode支持
1. C17：主要是解决一些标准缺陷
1. C23：还没出呢
## C++标准
1. 在C++98之前，1985年是C++的第一个大版本，1991年是第二个大版本，但此时语言显然不够稳定。1990年、1991年，ANSI C++委员会、ISO C++委员会相继建立。
1. C++98
    - 第一个真正意义上的C++标准版本
    - 新特性：运行时类型识别RTTI（Runtime Type Identification）、转型运算符、mutable、export、模板实例化、成员模板、在条件语句中定义、规定一些类型标准（nullptr_t、符号和大小修饰符）
    - 新库：locales、bitset、valarray、auto_ptr、模板化的string、I/O流、复数
    - STL：容器、函数、迭代器、函数对象
        > 1999年，Boost建立，意在为C++标准试验编写高质量库
1. C++03：小改进，主要是技术勘误，但也提出了一些小技术点：如构造对象时，使用空初始化的一些语法标准。
1. C++11：**超大改进**，从TR1和Boost中接纳了大量的特性和库。[C++11 Main Artical](https://zh.cppreference.com/w/cpp/11)
    - 核心新特性：
        - 关键字：类型推导（auto和decltype）、default和delete函数声明，final和overide、constexpr和LiteralType（编译期常量及其类型）、noexcept函数声明和运算符、nullptr、对齐声明符（alignas）、对齐值运算符（alignof）
        - 易用性：尾置返回类型（trailing return type）、初值列、委派构造（delegating）、构造函数继承、lambda表达式、类型别名和别名模板（using）、字符串字面值前缀（L、u8、u、U、R）、自定义整型/浮点型/字符/字符串字面值后缀、范围for语法糖
        - 模板：可变参数模板（typename ... pack-name)
        - 语法统一：在列表初始和拷贝列表初始化中均使用大括号\{\}
        - 性能：右值引用、移动构造、移动赋值
        - 安全性：scoped enum（禁止转型的enum）、多线程内存模型、线程本地存储声明符（thread_local）、static_assert（编译期断言）、动态内存管理（智能指针）、存储类型声明符（如extern、static、mutable、thread_local）
        - *不太懂的：属性声明符序列\[\[attr-list\]\]*，该特性可以作用于很多语法结构（函数、if、类型、对象）
    - 新头文件：
        - C库迁移：cfenv（浮点库）、cinttypes、cstdint（整型类型和宏）、cuchar
        - 数据：array、forward_list、initializer_list、scoped_allocator、tuple、unordered_map、unordered_set、scoped_allocator
        - 工具：chrono、random、regex、ratio（编译期有理数运算）
        - 类型：typeindex
        - 系统：system_error
        - 元编程：type_traits
        - 并发：atomic、condition_variable、future、mutex
    - 新的库特性
        - 容器：emplace原地构造、forward_list前向列表、begin/end、prev/next
        - 类型：智能指针、function、exception_ptr、error_code、error_condition、move_iterator
        - 函数：xxx_of、xxx_if、shuffle、partition系列、is_xxx、minmax、iota、current_exception、rethrow_exception
1. C++14：小的版本更新，主要是勘误。[C++14 Main Article](https://zh.cppreference.com/w/cpp/14)
    - 新语言特性：
        - 泛型：变量模板、泛型lambda、
        - lambda：初值列捕获、右值补货
        - 构造：聚合类（一种特殊的类）具有默认构造函数、
        - 字面值：整数字面值内可以加入单引号\'作为分隔符提升可读性（编译器会自动忽略）
        - 模板元：constexpr对函数的要求放宽
    - 新库特性：
        - 内存：make_unique（制造unique_ptr更好的方法）
        - 并发：shared_timed_mutex、shared_lock
        - 模板元：integer_sequence编译期整型序列
        - 工具：exchange（交换并返回）、quoted（输出或输入带引号的字符串）
1. C++17：大版本更新，合并了大量的新特性，废除了一些内容。[C++17 Main Article](https://zh.cppreference.com/w/cpp/17)
    - 移除：
        - 关键字：register、
        - 类型：auto_ptr、
        - 函数：random_shuffle()、旧的函数对象&适配器和binder、意外的异常抛出unexpected()
        - IO流别名：io_state/open_mode/seek_dir/streamoff/streampos（命名变更了）
        - 三字母词（trigraphs）：以??开头的三个字符，有特殊的转义含义，已被移除
        - bool类型禁止自增运算符
        - 动态异常说明：允许异常声明符标记一个父类，抛出异常是其子类即可。该特性已被移除。*那如何判断是否抛出了意料之外的异常类型呢？*
    - 废除：
        - 类型：std::iterator、raw_storage_iterator
        - 模板元：is_literal_type、result_of、result_of_t
        - 函数：get_temporary_buffer、
        - 头文件：本地化库codecvt
    - 新特性：
        - 字面值：字符字面值前缀符（u8、u、U、L）、
        - 语法：...现在叫折叠表达式了（4种形式的二元运算表达式）
        - 类型：noexcept成为函数签名的一部分、值类型变动（左值右值等）、几个新的attr声明符
        - 底层：求值顺序规则变动、inline变量（头文件也可以有变量定义）、强制拷贝构造优化（避免多次拷贝构造）、临时量实质化（也是为了避免出现不必要的拷贝）
        - 语法糖：lambda表达式可以捕获\*this、lambda表达式可以使用constexpr描述符（即使不用也可能是隐式的）、结构化绑定（同时赋多个值给多个标识符）、if和switch内可以先使用初始化语句序列（以;号结束）、using可以嵌套（using A::B::C \{\})、命名空间定义时也可以嵌套（namespace A::B \{\}）
        - 模板元：编译期条件语句（if constexpr表达式）、CTAD（模板元参数自动推导，对pair、tuple等很实用）、非类型模板参数可以使用auto声明类型
        - 宏：__has_include判断是否引入了某个头
    - 新库：
        - 工具：any通用包装类型、optional空或非空对象包装类型、variant一种类型安全的union、string_view高性能的只读字符串类型、charconv对C字符串转整/浮点型库
        - 内存：memory_resource
        - 辅助：execution用于指示泛型算法的执行策略（开发并行程序更简单）
        - 系统：filesystem文件系统
    - 新的库特性：
        - 工具：tuple（apply、make_from_tuple）、as_const左值转常量类型、not_fn非语义、
        - 算法：search字符串搜索算法、gcd最大公约数、lcm最小公倍数、clamp取三者的中值、reduce可并行非顺序的规约、xx_scan求前缀和、
        - 容器：非成员的std::size/empty/data函数、新的迭代器细分（老式连续内存迭代器）、map/set支持merge和extract、map/unordered_map接口扩展
        - 内存：destroy_at析构函数族、unintialize_move内存移动函数族、weak_from_this构造weak_ptr、std::pmr::memory_resource封装内存资源、std::pmr::polymorphic_allocator分配器和前面搭配使用、shared_ptr支持数组、new和new[]支持显式对齐、owner_less提供对于共享指针控制块的比较能力、
        - 对C兼容库的更新：
            - cstdlib：std::aligned_alloc申请内存并自动对齐。
            - cmath：hypot勾股求斜边、大量数学函数。由于cmath本身被numeric库包含，因此也可以使用。
            - cstddef：byte方便二进制运算的字节类型
            - ctime：timespec_get对timespec对象进行修改
        - 模板元：
            - 逻辑运算：conjunction合取、disjunction析取、negation否定
            - 基础类型值：大量的xxx_v类型（对xxx\<T\>::value简化）
            - 类型判断：is_swappable是否支持swap函数、is_invocable是否可调用、is_aggregate是否是聚合类型、has_unique_object_representations是否具备平凡可复制特性（主要用于判断类型是否可hash）
        - 其他：
            - 安全：launder提供对指针处对象的确定行为访问（和裸指针访问的未定义行为相对应）
            - 字符串：to_chars/from_chars位于charconv的实用函数
            - 并发：is_always_lock_free、scoped_lock获取多个锁、hardware_destructive_interference_size/hardware_constructive_interference_size优化数据共享的地址偏移来避免假共享、
            - 异常：uncaught_exceptions获取当前线程仍活跃的异常对象
            - chrono：对时间点和时间段提供floor、ceil、round等实用函数
1. C++20：大版本更新。[C++20 Main Article](https://zh.cppreference.com/w/cpp/20)
    - 新的语言特性：
        - 
    - 新库：
    - 新库特性：
        - 
1. C++23：还没出
1. 编译器支持情况（截止2022年底）

> 注1：一些在C++20、C++23决议被抛弃的特性将不再列出

> 注2：移除、废除的理由一般都是，设计理念始终没有较好的被实现，最终被移除。特性将会先标记废除，再在下一次的大版本中完全移除。

## 参考
1. [cppreference](https://zh.cppreference.com/w/)
1. [History of C++](https://zh.cppreference.com/w/cpp/language/history)
1. [C89和C99区别](https://www.cnblogs.com/xiaoyoucai/p/6146784.html)
1. 《C++17 the complete guide》，[中文翻译版](https://github.com/MeouSker77/Cpp17)

> 如果不能理解翻译的含义，建议切换到英文版本的网页