---
title: "《现代C++教程》：读书笔记"
date: 2023-12-27T12:25:30+08:00
categories:
- 读书笔记
- 技术书
tags:
- C/C++
- 施工中
- 编程语言
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/cpp.png
# draft: true
mermaid: true
---
从C++11开始，C++开始加速向一门更现代的语言进化。很多需求都有了更优秀的写法来替代原来的技巧。
<!--more-->
## 标准更迭
1. 实际上C++并不是C的超级，从一开始的标准就无法完全做到。在代码中应当使用宏和```extern```来严格标记混合使用的位置。参考[C和C++互操作]({{<relref "/content/post/language/Cpp-compile.md#C和C++互操作">}})。
2. 在编写代码的过程中，注意允许使用的C++标准，给IDE或编译器传递正确的选项。


## 可用性优化
1. ```constexpr```：指示编译器该表达式、函数，能够在编译期成为常量表达式，应该尝试在编译期确定其数值。有以下几点值得注意
    - ```constexpr```修饰函数，仍然允许在运行期求值，但是不能有副作用调用（比如调用```cout```输出）。
    - ```constexpr```和```const```在语义上的区别，前者说明是一个常量表达式，后者说明是一个不可修改的常量（但其值则可能是运行期给出）。
    - 在C++11初次引入时，常量表达式内可以递归，但仍不能使用循环、分支、定义局部变量等简单语句。从C++14开始允许使用。
    - C++17开始，支持```if constexpr```，可以尝试在编译期提前决定分支判断，和函数定义不同，此时的条件表达式**必须具备**编译期常量结果。

1. 允许在```if/switch```语句作用域内定义临时变量，例如
    ```cpp
    // 先定义，以 ; 分割
    // if(Type t; boolean statement)
    if (const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 3);
        itr != vec.end()) {
        *itr = 4;
    }
    ```

1. 初始化列表```std::initializer_list<>```，从C++11开始，为了给类初始化提供近似于POD类型的写法，提出的初始化列表。这使得具有支持初始化列表构造器的类型，可以和POD类型一样，使用```{}```进行初始化，例如
    ```cpp
    class MyClass {
    public:
        MyClass(std::initializer_list<int> list) { /* ... */}
        MyClass(std::initializer_list<int> list, int append) {/* ... */}
        // 构造函数调用优先级弱于初值列
        MyClass(int a,int b) {/* ... */}
        MyClass() {/* ... */}
    }

    int a[]={1,2,3,4};
    MyClass myClass = {1,2,3,4};
    // 初始化列表也可以直接通过类似构造函数的方式调用
    MyClass myClass2 {1,2,3,4};
    // 可以混合
    MyClass myClass3 {{1,2,3,4},1};
    // 优先使用初值列，实际上带有初值列的构造函数的类型，其大括号构造方式会默认劫持所有{}的构造调用
    MyClass myClass4 {1,2};
    // 因为存在无参构造函数，此时以下两种都会调用无参构造
    // 如果没有无参构造，前者会编译失败，后者会调用到初始化列表的构造
    MyClass myClass5,myClass6{};
    // 另外注意无参构造函数用圆括号初始化会导致一个歧义，此处声明了一个新函数，而非调用了MyClass构造函数
    MyClass myClass();
    ```
    > 对于使用圆括号构造，大括号构造的优劣对比，以及使用场景，请参考[C++创建对象时区分圆括号( )和大括号{ }](https://zhuanlan.zhihu.com/p/268894227)。总而言之，如果一个类型没有使用初始化列表的构造函数，那么可以无脑使用大括号，它相对更安全（避免默认的数据窄化，以及无参构造的歧义）。但如果具备初始化列表，则会被初始化列表的劫持，此时仍然需要用圆括号来指定其他类型的构造。


1. 结构化绑定，这一功能主要完善了C++11开始拥有的```std::tuple```，从C++17开始，可以自动对元组进行解包，例如
    ```cpp
    // 拷贝自原书
    std::tuple<int, double, std::string> f() {
        return std::make_tuple(1, 2.3, "456");
    }
    int main() {
        // 自动解包，绑定tuple内容
        auto [x, y, z] = f();
        std::cout << x << ", " << y << ", " << z << std::endl;

        std::map<int, int> myMap;
        myMap[1] = 2;
        myMap[2] = 4;
        // 自动绑定字典键值结构（原来的写法只能获得键值迭代器，再用first/second来获取键/值）
        for (const auto& [key, val] : myMap) {
            cout << key << " " << val << endl;
        }

        return 0;
    }
    ```

1. 类型自动推导，```auto```和```decltype```，其中
    - ```auto```用于对变量类型进行类型推导。用于对变量的定义和声明。
    - ```decltype```用于对表达式进行类型推导，其结果可以进一步用于变量定义、或者模板类型计算
    同时，在C++11、C++14都对返回值类型推导做了一定优化，逐渐让返回值的类型推导更自动化，例如
    ```cpp
    // C++11之前，虽然T、U可以自动推导，但是R必须由用户给出
    template<typename R, typename T, typename U>
    R add(T x, U y) {
        return x+y;
    }

    // C++11，允许在函数签名处用尾返回类型，将返回类型后置（更像一个现代语言）
    // 但是这种语法还是很丑陋
    template<typename T, typename U>
    auto add2(T x, U y) -> decltype(x+y) {
        return x + y;
    }

    // C++14，直接优化到位
    template<typename T, typename U>
    auto add3(T x, U y){
        return x + y;
    }
    ```
    从C++14起，编译器还提供了对带有封装的返回类型进行推导，例如
    ```cpp
    std::string lookup1();
    std::string& lookup2();

    decltype(auto) look_up_a_string_1() {
        return lookup1();
    }
    decltype(auto) look_up_a_string_2() {
        return lookup2();
    }
    ```