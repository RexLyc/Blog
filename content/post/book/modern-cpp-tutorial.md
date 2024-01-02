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
1. 一些很熟悉的C++11特性一笔带过：
    1. 区间迭代语法：```for(auto &t:vec)```
    2. 类型别名模板，类型别名。即推荐用```using```代替```typedef```
        ```cpp
        // 弥补typedef只能对类型指定别名，不能对模板指定别名的缺陷
        template<typename T>
        using MyTemplateAlias = MyTemplate<std::vector<T>>;

        // 无法通过编译，因为模板并不是类型，模板是用来生成类型的
        // typedef MyTemplateAlias ... 

        // using 同时也能给函数指针的别名带来更好的可读性
        using MyFunc = int(*)(void *);
        // 相比之下，使用typedef可读性较差
        typedef int (*MyFunc)(void *);
        MyFunc f = [](void*) ->int {
            return 0;
        };
        ```
    1. 变长参数模板，函数模板、类模板都可用。解包可通过递归、模板展开，支持```sizeof...(args)```获取变长参数数量
        ```cpp
        // Magic可接收0，1，2...个模板参数
        template<typename ...Ts> class Magic;
        // 变长参数函数模板
        template<typename... Args> void log(Args... args);

        // C++11中用递归模板函数，需要提供两个模板
        template<typename T, typename...Args> void log(T v, Args... args) {
            std::cout << v << std::endl;
            log(args...);
        }
        // 该模板用于结束递归
        template<typename T> void log(T v) {
            std::cout<< v << std::endl;
        }

        // C++17中由于出现if constexpr，因此可以在一个模板中完成
        template<typename T0, typename... T>
        void printf2(T0 t0, T... t) {
            std::cout << t0 << std::endl;
            // 在if constexpr出现之前，if无法在编译期进行判断
            if constexpr (sizeof...(t) > 0) printf2(t...);
        }

        ```
    1. 委托构造，允许构造函数调用另一个构造函数
    2. 继承构造，对于构造函数继承的情况，允许通过```using```直接继承。
    3. 显式虚函数重写，添加关键字```override```，```final```，从而可以对虚函数的重写进行显式控制。
        ```cpp
        class A {
        public:
            virtual f();
            virtual h() final;
        }

        class B: public A{
            virtual f() override;
            virtual f(int) override; // 非法，不存在该虚函数可供重写
            virtual h(); // 非法，基类中已声明为final
        }

        ```
    1. 显式禁用默认函数，```=delete```，显式使用默认函数```=default```
    2. 强枚举类型：```enum class```

1. ```constexpr```：指示编译器该表达式、函数，需要在编译期被编译为常量表达式。有以下几点值得注意
    - ```constexpr```和```const```在语义上的区别，前者说明（可能）是一个编译期常量表达式，后者说明是一个不可修改的常量（但其值则可能是运行期给出）。
    - ```constexpr```修饰函数，仍然允许在运行期求值，但是不能有副作用调用（比如调用```cout```输出）。就是说该函数应当保证如果输入是常量表达式，那么能够在编译期也获得常量表达式。
    - 修饰构造函数，用于说明类型可以成为可用于常量表达式的对象。当然并不限制成为一个运行期才确定的对象实例（和修饰函数的要求相同）
    - 在C++11初次引入时，常量表达式内可以递归，但仍不能使用循环、分支、定义局部变量等简单语句。从C++14开始允许使用。
    - ```constexpr```用于定义变量时，要求变量的值必须是编译期常量表达式。
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

    // 此外，C++11中初始化列表展开还能做到对变长参数的展开
    // 不过该写法不如C++17支持的折叠表达式简洁
    template<typename T, typename... Ts>
    auto printf3(T value, Ts... args) {
        std::cout << value << std::endl;
        // 理解...参数展开可以在复杂表达式中展开
        // 即args...是普通展开，也可以用expr(args)...
        // 编译器会为其扩展为expr(args0),expr(args1),...

        // 理解逗号表达式的计算规则

        // (void)转型消解对未使用的初值列表的警告
        (void) std::initializer_list<T>{([&args] {
            std::cout << args << std::endl;
        }(), value)...};
    }
    ```
    > 对于使用圆括号构造，大括号构造的优劣对比，以及使用场景，请参考[C++创建对象时区分圆括号( )和大括号{ }](https://zhuanlan.zhihu.com/p/268894227)。总而言之，如果一个类型没有使用初始化列表的构造函数，那么可以无脑使用大括号，它相对更安全（避免默认的数据窄化，以及无参构造的歧义）。但如果具备初始化列表，则会被初始化列表的劫持，此时仍然需要用圆括号来指定其他类型的构造。
    > 初始化列表展开，可参考[知乎回答](https://www.zhihu.com/question/443285720/answer/1723184923)。

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
1. [折叠表达式](https://en.cppreference.com/w/cpp/language/fold)：C++17开始对于主要的32种二元运算符，编译器支持将变长参数直接展开。展开表达式依然可以是一个复杂表达式。这个特性大大提高了变长参数的易用性。下面是一些示例
    ```cpp
    template<typename ... T>
    auto sum(T ... t) {
        // 无初值的左、右展开
        int rightExpansion = (t + ...); // t0 + (t1 + (...))
        int leftExpansion = (... + t); // ((t0 + t1) + ... )
        // 有初值的左、右展开
        // 初值包/参数包内运算符优先级需要高于转型，否则要添加括号，如(1 * 2)
        int initRight = (t + ... + (1 * 2)); // t0 + (t1 + ... + (1 * 2))
        int initLeft = ((1 * 2) + ... + t); // ((1 * 2) + t0) + ...)
        return 0;
    }

    // 借助折叠表达式，可以很方便的一系列处理可变长的相同操作
    template<class T, class... Args>
    void push_back_vec(std::vector<T>& v, Args&&... args)
    {
        // 用折叠表达式对所有参数检查是否是可构造的
        // 这里的参数包甚至是一个模板元运算
        static_assert((std::is_constructible_v<T, Args&> && ...));
        // 用折叠表达式push_back
        (v.push_back(args), ...);
    }
    ```
1. 非类型模板参数推导，允许使用```auto```指示编译器推导非类型模板参数的类型，形如
    ```cpp
    template <auto value> void foo() {
        std::cout << value << std::endl;
        return;
    }
    ```