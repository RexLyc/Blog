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
    3. 智能指针：```unique_ptr```、```shared_ptr```、```weak_ptr```

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

## 函数对象和Lambda表达式
从C++11引入lambda表达式，其基本形式如
```cpp
[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型 {
// 函数体
}
```

在C++14，又为其补充了表达式捕获，使其拥有了对右值的捕获能力，示例如下
```cpp
auto important = std::make_unique<int>(1);
// 在之前的情况下，因为只能捕获值或引用，此时独占智能指针important无法被捕获
// C++14允许表达式捕获，这样就可以使用右值
auto add = [v1 = 1, v2 = std::move(important)](int x, int y) -> int {
    return x+y+v1+(*v2);
};
std::cout << add(3,4) << std::endl;
```

此外，C++14还支持了泛型lambda，这一特性在C++11中并不支持，其写法就是使用```auto```对参数进行类型推导，例如
```cpp
auto add = [](auto x, auto y) {
    return x + y;
}
```

> 值得注意的是，如果lambda表达式内部想要递归，则不能用auto自动推导lambda表达式的类型，需要手动指定，例如```function<int(int,int)> doSth = /*..递归调用doSth..*/```。

Lambda表达式的本质是一个很像函数对象的类类型对象（闭包对象）。而且**当捕获列表为空**时，Lambda表达式还能够和函数指针进行转换。为了协调统一这些概念，C++11开始引入了函数对象概念，即```std::function```。

函数对象统一了所有的可调用类型，可以更安全方便的对函数、函数指针等可调用内容进行复制存储调用。在此基础上又提供了很多实用的工具，如绑定函数调用参数的```bind / placeholders```，例如
```cpp
void foo(int a, int b, int c) {
	cout <<"foo: " << a << " " << b << " " << c << endl;
}
int main() {
    // 第一个参数用占位符暂时代替，给出了后两个参数
	auto fooBind = bind(foo, placeholders::_1, 1, 2);
    // 此时只需要传入1个参数，也就是_1位置的占位符
	fooBind(3);
    // 占位符数量和编号和正式调用时的参数相对应
    auto fooBind2 = bind(foo, placeholders::_1, 1, placeholders::_2);
    fooBind2(2,3);
    // 也可以使用同一个占位符
    auto fooBind2_dup = bind(foo, placeholders::_2, 1, placeholders::_2);
    fooBind2_dup(2,3); // 输出为3 1 3
    return 0;
}
```

## 容器
本章节主要记录一些容器的使用习惯推荐
1. ```array```：在数组长度固定的情况下，用```array```代替传统```[]```和```vector```。
2. 初值列可用于对关联容器的初始化，例如
    ```cpp
    std::unordered_map<int, std::string> u = {
        {1, "1"},
        {3, "3"},
        {2, "2"}
    };
    ```
3. C++17引入的实用工具：```variant```、```optional```、```any```
    - ```any```：类，存储任何可复制构造类型的对象、内置类型，通过```any_cast```进行转型并访问。
    - ```optional```：类模板，任意时刻可包含一个值，或者为空。
    - ```variant```：类模板，类型安全的联合体。其构造函数有相对复杂的语法，[参考](https://zh.cppreference.com/w/cpp/utility/variant/variant)，基本示例如
        ```cpp
        // 用 std::string{"ABCDE", 3}; 初始化第一个可选项类型
        // 理解代码需联系初值列和参数展开
        std::variant<std::string, std::vector<int>, bool> var{
            std::in_place_index<0>, "ABCDE", 3};
        
        assert(var.index() == 0);
        std::cout << std::get<std::string>(var) << std::endl; // "ABC"
        ```
    - 工具：
        - ```std::get```：```get```对多种类型都有重载
        - ```std::in_place, std::in_place_type, std::in_place_index, std::in_place_t, std::in_place_type_t, std::in_place_index_t```：一系列构造指示，用来说明将一个值作为哪个类型选项来构造```variant```
4. 元组
    - 基本用法：```tuple```、```make_tuple```，拆包```std::get<>()```、```std::tie()```。但是```get<>```有一个重大的缺点就是其访问的内容的下标或类型是模板参数，需要在编译期决定，例如
        ```cpp
        auto t = std::make_tuple(1,2.2,'3');
        auto t1 = std::get<0>();
        // 当使用类型get时，要求元组中没有重复类型的元素
        auto td = std::get<double>();
        ```
    - ```std::tuple_cat```：元组合并
    - ```std::tuple_size<>::value```：获取利用类型萃取元组大小
    - C++17引入了```variant<>```，提供了同时容纳多种类型的变量存储能力，可以理解为更安全的union。而且借助```variant```能进一步提高元组的运行期访问能力，例如。
        ```cpp
        template <size_t n, typename... T>
        constexpr std::variant<T...> _tuple_index(const std::tuple<T...>& tpl, size_t i) {
            if constexpr (n >= sizeof...(T))
                throw std::out_of_range(" 越界.");
            // 利用递归，找到i==n的情况，并原位构造variant，且该值构造为第n个variant选项
            if (i == n)
                return std::variant<T...>{ std::in_place_index<n>, std::get<n>(tpl) };
            return _tuple_index<(n < sizeof...(T)-1 ? n+1 : 0)>(tpl, i);
        }

        template <typename... T>
        constexpr std::variant<T...> tuple_index(const std::tuple<T...>& tpl, size_t i) {
            return _tuple_index<0>(tpl, i);
        }
        ```
    > 理解元组主要是理解它的类型和值，C++的元组之间，不同模板参数的元组完全是不同的类型。

## 并行和并发
C++引入的几个基本类型
- ```thread```：线程
- ```mutex```：基础互斥锁
    - ```lock_guard```：离开作用域自动释放
    - ```unique_lock```：离开作用域自动释放，也可手动提前释放/再上锁
- ```future```：访问一个异步操作的结果，通过 ````std::async```` 、```std::packaged_task```或```std::promise```创建。在调用```get/wait```以及**析构**时会阻塞当前线程，以等待结果。
    - ```package_task<>```：包装一个函数，是可调用对象，本身并不具备开辟新线程的能力。
    - ````std::async````：异步的运行一个函数（也可以选择在当前线程中执行），还可以选择惰性求值。
    - ```std::promise```：一次性使用的，存储值、异常的类型。是更方便的并发同步、数据同步工具类型。通过```promise```推送数据，```future```获取消息。
    > 三种类型的功能各不相同，但都具有创建future的能力。想传值用promise，想自定义包装用packaged_task，想无脑异步用async。
    示例代码
    ```cpp
    // 拷贝自cppreference

    // 来自 packaged_task 的 future
    std::packaged_task<int()> task([](){ return 7; }); // 包装函数
    std::future<int> f1 = task.get_future();  // 获取 future
    std::thread(std::move(task)).detach(); // 在线程上运行

    // 来自 async() 的 future
    std::future<int> f2 = std::async(std::launch::async, [](){ return 8; });

    // 来自 promise 的 future
    std::promise<int> p;
    std::future<int> f3 = p.get_future();
    std::thread( [&p]{ p.set_value_at_thread_exit(9); }).detach();

    std::cout << "Waiting..." << std::flush;
    f1.wait();
    f2.wait();
    f3.wait();
    std::cout << "Done!\nResults are: "
                << f1.get() << ' ' << f2.get() << ' ' << f3.get() << '\n';
    ```
- ```condition_variable```：条件变量。消费端在```wait```时自动释放锁，唤醒后自动获取锁。生产端用```notify_one / notify_all```控制唤醒。


在C++11之后，为了进一步提高性能，也开始出现了对无锁编程的支持，具体是用原子变量和内存顺序模型两个内容来完成的。具体原理还会牵扯到缓存一致性等问题，可以详细参考阅读[原子操作与内存模型/序/屏障](https://zhuanlan.zhihu.com/p/611868395)。下文是单独对C++的内存顺序模型做一个总结。

原子变量```atomic<>```。类模板，原子类型，通过CPU支持的CAS指令等原子操作去修改变量。但并不是每个平台都能支持真正的无锁，可以用```is_lock_free```进行判断。

有了原子变量，还不能直接把原子变量当成一种互斥或同步的手段，这是因为现代编译器、CPU，都有可能会对指令重排。编译器的重排可能出于指令优化的考虑，CPU的重排则可能是出于执行速度的考虑，避免CPU空转。因此任何操作，尤其是在原子操作附近的临界区中的操作，并不能保证顺序和代码顺序是一致的。表现上就是，这一段代码单线程执行不会有任何问题，但是如果在多线程环境下，就会出错。这一点的复现依赖于平台，偏强内存一致性模型的主要是x86，偏弱内存一致性有PowerPC、MIPS、ARM，RISC-V则两种都有支持。

对于内存顺序不一致的情况。不同的CPU都提供了底层的指令来明确地插入内存屏障。但这种底层的方法显然需要进行进一步的抽象。因此C++需要有一种比较高层的抽象模型，对关键的代码指定其内存一致性模型，避免编译器和CPU指令重排造成错误的运行结果。

因此C++11开始引入内存顺序模型：```std::memory_order```。问题要从一致性开始说，为了尽量提高并发性能，对若干变量的修改，不应当强制要求立刻对所有线程可见，也不应当强制要求变量修改的可见性是顺序的，具体来说，可将这种并发种不同线程之间的数据一致性分类为四个等级：
1. 线性一致性：完全线性化的修改，表现上就像是单核心单线程处理。
2. 顺序一致性：读取数据一定能够读取到最近一次的修改
3. 因果一致性：只对有因果、依赖性的数据之间的同步做要求，而无关数据之间的修改不做要求
4. 最终一致性：类似于分布式系统的最终一致，最终一定有某个时刻，修改将会对所有线程可见

一致性模型不仅给出了多线程之间的读写要求，更重要的是对一个线程内部的读写顺序要求。对于这四种一致性情况，C++使用了6种内存模型来表达，分别是
1. ```std::memory_order_relaxed```：宽松顺序
3. ```std::memory_order_release```、```std::memory_order_consume```：释放-消费顺序
4. 2. ```std::memory_order_release```、```std::memory_order_acquire```：释放-获取顺序
4. ```memory_order_acq_rel```：释放-获取顺序
4. ```std::memory_order_seq_cst```：顺序一致顺序

在实际应用中顺序一致性是最强的约束，也是C++默认的内存序。在实际编码中，肯定要优先保证正确性，而性能的优化则是一个很复杂的问题。并发算法更重要一些，并不是说无锁一定高效。在个别场景下，锁操作尤其是自旋锁可能会比无锁数据结构有更高的性能。

## 其他不太实用的
### 正则表达式
C++11引入[正则表达式](https://zh.cppreference.com/w/cpp/regex)，具有匹配、搜索、替换三种功能。其主要类型和作用如下:
- ```std::regex```：类模板，存储正则表达式，可以根据需要选择支持的正则表达式语法（ECMA、awk等）、匹配条件（如无视大小写等）
- ```std::sub_match```：类模板，通过begin/end存储一个匹配的起始/结束。
- ```std::match_result```：类模板，存储所有匹配结果，一般常用的是```std::cmatch```（const char *）、```std::smatch```（string）。其内存储了若干个```sub_matches```。
- ```regex_match()```：函数模板，只考虑完全匹配，即整个原数据刚好可以被正则表达式描述。
- ```regex_search()```：函数模板，考虑部分匹配，每一次匹配，可以获得匹配结果，匹配结果前缀，匹配结果的后缀。对后缀循环，就能计算出全部的匹配。
- ```regex_replace()```：函数模板，对匹配结果进行替换，替换所用的正则表达式语法也是可选的。
- ```regex_iterator```：迭代正则表达式的匹配，可以通过原数据和正则表达式进行构造，可以通过迭代器遍历直接获得类似于```regex_search```的效果。通过迭代器可以获得```match_result```，无法获得子匹配的情况。
- ```regex_token_iterator```：每个匹配结果和子匹配的迭代器，实现上内部可能持有```regex_iterator```，可以遍历匹配结果，以及每次匹配的某个指定子匹配。而且还可以通过调整构造函数参数，遍历由匹配片段分割的各个失配片段。

其基本使用如下例子所示
```cpp
// 拷贝自cppreference
std::string s = "Some people, when confronted with a problem, think "
    "\"I know, I'll use regular expressions.\" "
    "Now they have two problems.";

// 使用ECMA正则语法，大小写不敏感
std::regex self_regex("REGULAR EXPRESSIONS",
        std::regex_constants::ECMAScript | std::regex_constants::icase);

// 返回结果true/false表示是否存在部分匹配
if (std::regex_search(s, self_regex)) {
    std::cout << "Text contains the phrase 'regular expressions'\n";
}

std::regex word_regex("(\\w+)");
// 匹配所有单词，构造匹配结果迭代器
auto words_begin = 
    std::sregex_iterator(s.begin(), s.end(), word_regex);
auto words_end = std::sregex_iterator();

std::cout << "Found "
            << std::distance(words_begin, words_end)
            << " words\n";

const int N = 6;
std::cout << "Words longer than " << N << " characters:\n";
for (std::sregex_iterator i = words_begin; i != words_end; ++i) {
    // 匹配结果是smatch，也就是match_result<string>
    std::smatch match = *i;
    std::string match_str = match.str();
    if (match_str.size() > N) {
        std::cout << "  " << match_str << '\n';
    }
}

// 替换，$&代表将匹配的子结果整体保留，并前后添加方括号
std::regex long_word_regex("(\\w{7,})");
std::string new_s = std::regex_replace(s, long_word_regex, "[$&]");
std::cout << new_s << '\n';
```

> 目前来说，C++标准库中的正则表达式性能很可能还是很差（看平台和编译器），很多时候不如Python3的re，在实际使用场景下需要谨慎使用。