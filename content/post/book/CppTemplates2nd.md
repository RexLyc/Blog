---
title: "C++ Templates 第二版：读书笔记"
date: 2024-03-26T20:41:51+08:00
categories:
- 计算机科学与技术
- C++
tags:
- C++
- 读书笔记
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/book/CppTemplate2nd.jpg
draft: true

---
本文记录对C++模板一书第二版的书本摘抄，学习笔记。
<!--more-->

## 语法补充

对一些下文用到，但是可能之前不常用的语法进行补充

```cpp
// 类型判断
std::is_same<class,class>::value;
// 使用is_same_v则不需要再使用::value，since C++14
std::is_same_v<class,class>;
static_assert(std::is_same<int,int&>::value, "int !=int& ");

// 计算decay退化后的公共类型
std::common_type<class,class>::type;
// 同理，C++14后，添加_t不需要再用::value，下略
std::common_type_t<class,class>;

// 退化类型
std::decay_t<class>;

// 成员指针，语法是固定的
struct C { int m; };
// 指向作为类 C 的成员的非静态数据成员 m 的指针，能准确地以表达式 &C::m 初始化。
int C::* p = &C::m;          // 指向类 C 的数据成员 m，必须这样写，而不是&(C::m)
// 此时可以使用成员指针访问运算符operator.* operator->*的右运算数
C c = { 7 };
c.*p = 5;
C *cp = &c;
c->*p = 6;
// 当然，推荐用auto获取
auto mp = &C::m;

```

另外注意书中经常使用的写法是```T const&```，而不是```const T&```，前者实际上更符合从右向左念的方式。

本节参考[指针和成员指针](https://zh.cppreference.com/w/cpp/language/pointer)

## 术语速览
1. 实例化：用具体类型取代模板类型参数的过程叫做“实例化”。它会产生模板的一个实例。即使是类模板也是这样，类模板的模板成员函数也只有在需要该函数的的时候才会实例化。类模板的static成员也是只在每种类模板实例化后的类，分别实例化一次。
2. 两阶段编译检查（Two-Phase Translation ）：模板编译分为两个步骤，定义处做和模板参数无关的内容的检查（语法、未定义的引用、static_assert），实例化处做第二次检查，这一次检查所有和模板参数有关的内容，是否能满足语法要求。
3. 模板参数：```<>```括号内的
4. 模板默认参数：可以为模板参数（类型、值）设置默认值，且可以使用在前面的模板参数，如```vector```的定义
5. 调用参数：函数模板中的形参
6. 类型退化（decay）：一套固定规则，去掉const、volatile、引用等，数组变指针。
7. 模板参数推断（推导）：模板参数在使用时可以只是类型的一部分（如```const T&```）。在此基础上类型推断是受限制的。
   - 如果形参是引用，任何隐式类型转换都不允许，实参形参类型必须一致。这也能保持形参类型**完全不退化**。
   - 如果调用参数是按值传递的，那么**也只有退化**（decay）这一类简单转换是被允许的：const和 volatile 限制符会被忽略，引用被转换成被引用的类型，raw array 和函数被转换为相应的指针类型。通过模板类型参数 T 定义的两个参数，它们实参的类型在退化（decay）后必须一样。注意**不一定真的退化**。
   > 注一：建议显式使用const和volatile限制符，模板参数推断中不一会携带这种限制，可能退化。

   > 注二：企图用同一种模板参数，接收字符串字面值和string是不可能的。因为无论是引用还是按值传递，这种类型转换都不允许。此时实例化失败。
        ```cpp
        // ok
        std::max("233", "666");
        // 不ok，用const char*和string类型都无法满足另一个的转型限制
	    std::max("233", std::string("666"));
        ```
    
    > 注三：如果此时显式指定模板参数，则不受推导限制（废话）
8. 类模板的模板类型推导：和函数模板类似，但此时更多的问题在于构造函数。在C++17之后，开始支持自动推导。此时有两种方式。
    - 在构造函数中：提供一个值，允许通过该值来推断类模板的模板参数类型，形如```vector v{1};```
    - **推断指引**：在模板的同级作用域内，显式指定希望的推断方式，形如```MyTemplate(char const*)->MyTemplate<std::string>;```
9.  模板重载：注意不是指特化，模板是可以重载的（完全不同的模板参数数量和类型），只要在实例化阶段保证只有一个模板匹配即可。但这种重载可能出现隐藏的错误。最好还是只做特化。
10. 类模板内的非函数模板
11. 友元函数：友元函数 friend 函数是一个不为类成员的函数，但它可以访问类的私有和受保护的成员。 友元函数不被视为类成员；它们是获得了特殊访问权限的普通外部函数。在类模板中，友元函数也相对特别，它不是函数模板，也同样不属于这个类模板的实例化类型，它只会在真正需要的时候随类模板一同实例化出来。
12. 特化：函数模板和类模板都可以进行特化，比如将一部分模板参数固定下来、为形参提供特殊限制符（引用、const等），并为其设计特殊的代码实现。分为特化、部分特化。
13. 类型别名：推荐使用```using ```，例如```using xxx = YourTemplate<int>```。
14. 别名模板：对别名进行一定程度的繁华。形如。
    ```cpp
    template<typename T>
    using DequeStack = Stack<T, std::deque<T>>;
    ```
15. 类成员别名模板：对类模板的成员，取别名。这种技术也是上面```common_type_t```等类型萃取模板的实现原理。
    ```cpp
    // 这里是少数必须使用typename而不能用class的场景，用于说明等号右侧是一个类型，而不是成员变量/函数
    template<typename T>
    using MyTypeIterator = typename MyContainerType<T>::iterator;
    ```
16. 聚合类：特殊的class/struct，没有用户定义的显式的，或者继承而来的构造函数，没有 private 或者 protected 的非静态成员，没有虚函数，没有 virtual，private 或者 protected的基类。表现上就是最传统的C风格struct。聚合类也支持模板化，而且可以用推断指引（也只能用，毕竟没有构造器）的方式，支持模板类型推导。
17. 剩余参数：变长参数模板中的```Types ... args```。
18. 函数参数包：形参列表
19. 模板参数包：模板参数列表
20. 变参推断指引：在变长参数模板中，对模板参数的推断。
21. 类型成员：在类中由```typedef```、```using```定义的类型成员。只能用类名域操作符进行使用，```YourClass::TypeMember```。它们不是类实例的成员。
22. 非依赖基类：非依赖指的是对模板参数非依赖，即不需要知道模板参数就能确定的一种完整的类型定义。注意这里不是说基类的模板，而是说派生类的类模板，它继承的基类，不需要等待派生类的模板参数。例如下面的```D2```。这种基类在大多数情况下的使用和非类模板的基类一样。除了一件事情，就是对于无作用域限定的名称的查找，会优先查找基类中的定义，而不是模板参数。这也说明了，基类的实例化早于派生类的实例化，因此在对派生类内的符号进行确定时，才会优先使用基类。这个类型查找的优先顺序无法被改变，也无法强制指定，因此当对非限定基类进行继承时，一定要小心来自基类的名称。
    ```cpp
    template<typename X>
    class Base {
    public:
        int basefield;
        using T = int;
    };

    class D1: public Base<Base<void>> { // not a template 
    public:
        void f() { basefield = 3; } // 正常访问
    };

    template<typename T>
    class D2 : public Base<double> { // nondependent base，非依赖型基类，这个基类不需要等待T，就已经是完整类型了
    public:
        void f() { basefield = 7; } // 正常访问，实际上即使这里有同名的非类型模板参数，也不会被访问到，优先用非依赖基类中查找到的名称
        T strange; // T会从基类中优先查找，而不是使用这里的模板参数，因此T实际上是Base<double>::T，即int
    };
    ```
23. 依赖基类：作为对比，如果基类的类型也需要派生类模板的模板参数才能决定。就称之为依赖基类。此时如果对无作用域限定名称的查找延后到二者的模板实例化的阶段，可能会造成一些无法解决的错误（延迟处理会造成很大问题，尤其是如果在延迟处理的过程中，比如多次使用相同模板参数的类的调用中间，又发生了基类模板的特化定义，那么最后的类对象，到底应该使用哪一个呢）。因此C++标准规定了，在有依赖基类的情况下，对无作用域限定的名称的查找，会立刻进行（实例化的第一阶段），但不会去依赖型基类中进行。如果需要，则必须使用```this```，```XXX<>::```此类方式进行限定。这种方式将会强制延迟名称查找到实例化阶段。此时基类已经完成实例化。但对于此类情况，**多继承**将会变得更加棘手，对于有作用域限定的名称，会优先从非依赖基类中查找名称，如果没有找到再从依赖型基类中查找。
    ```cpp
    template<typename T>
    class DD : public Base<T> { // dependent base
    public:
        // 此时会立刻查找名称basefield，实际上，如果在当前作用域（全局、当前命名空间等）内找不到的话，会立刻报错。
        void f() { basefield = 0; } // #1 problem...
    };
    
    template<>
    class Base<bool> { // explicit specialization
    public:
        enum { basefield = 42 }; // #2 tricky!
    };
    
    void g (DD<bool>& d) {
        d.f();  // #3 oops?
    }

    template<typename T>
    class DD3 : public Base<T> {
    public:
        using Base<T>::basefield;
        void f() { basefield = 0; } // 这种情况下是可以的，因为已经在当前作用域给出了basefield。此时对无限定名称的查找会成功（虽然还未实例化）
    };
    ```
24. 泛型lambda：形如```[](auto x) {}```，实际上这时编译器会为其生成一个类，并创建一个```operator()```的成员模板。
25. 变量模板：从C++14开始，允许变量被类型参数化。变量模板其实本质上也是模板编程的一种语法糖，用于更好的使用代表类模板成员的变量模板。是各种```is_same_v```类型萃取模板的原理。例子和意义如下
    ```cpp
    template<typename T>
    constexpr T pi{3.1415926535897932385};

    auto piF = pi<float>;

    // 更有价值的用法是，提供对类模板成员的访问
    template<typename T>
    class numeric_limits {
    public:
        static constexpr bool is_signed = false;
    }; 

    template<typename T>
    constexpr bool isSigned = std::numeric_limits<T>::is_signed;
    
    // 因此可以使用
    isSigned<char>
    // 而不是更长更原始的
    std::numeric_limits<char>::is_signed
    ```
26. 模板参数模板：在有些情况下，我们会要求模板参数的实际类型，也必须是一个模板。实际上STL的大部分容器都有这种要求。这里是少数需要用到class，而不是typename的场景（但在C++17之后，也不再需要了）。但这里有一些细节，模板参数的模板中的模板参数（下面Elem）并没有用到，而且实际上也不能在模板参数表以外使用，因此不需要写（写了可以作为提示）。
    ```cpp
    // 这里的内层template，说明模板参数是模板。

    // template<typename T, template<typename Elem> class Cont = std::deque> // C++17之前
    // 实际上，如果是C++17之前，要求模板实参和形参数量匹配，必须写成
    // template<typename Elem, typename Alloc = std::allocator<Elem>> class Cont = std::deque
    template<typename T, template<typename Elem> typename Cont = std::deque> // C++17之后
    class Stack {
    private:
        // 注意是用T，而不是Elem，Elem只能在模板参数列表里使用，它在类模板内部是不存在的
        Cont<T> elems;
    public:
        // 此时的友元声明也要多一点
        template<typename, template<typename>typename> friend class Stack;
    };
    
    ```
27. 特殊成员函数模板：构造函数等特殊成员函数也可以是模板，上面已经写过了重载运算符的情况，写法都是一样的，不再赘述。


## 模板常用工具
### auto

### decltype：
1. 一些特例：
   1. ```template<decltype(auto) N>```，此时非类型模板参数N的类型，会被推导为引用类型。

### 变长参数模板
1. 定义在前，使用在后：例如```T ...t```是类型T的变长个参数t，```<typename ..T>```是变长个类型T，使用时（只作为实参传递，无op情况下），```t...```/```T...```。
    > T和t，在作为变长参数使用时语法是一样的
3. ```sizeof...()```：注意这是一个固定的写法可以对参数包的类型和参数使用。
    ```cpp
    sizeof...(Types)
    sizeof...(args)
    ```
4. 折叠表达式，是变长参数模板中最强大的工具之一，详见[C++可用性优化]({{<relref "/content/post/book/modern-cpp-tutorial.md#可用性优化">}})，下面是一些折叠表达式在模板中的应用
    ```cpp
    template<typename T1, typename... TN>
    constexpr bool isHomogeneous (T1, TN...)
    {
        return (std::is_same<T1,TN>::value && ...); // since C++17
    }
    ```
    > 使用折叠表达式的过程中，使用圆括号是一个不错的习惯，可以避免一些语法问题
5. 变参下标：变长参数可以作为数组下标
    ```cpp
    template<typename C, typename... Idx>
    void printElems (C const& coll, Idx... idx)
    {
        print (coll[idx]...);
    }

    template<std::size_t... Idx, typename C>
    void printIdx (C const& coll)
    {
        print(coll[Idx]...);
    }
    ```
6. 变参类模板，是Tuple、Variant的实现基础
    ```cpp
    template<typename... Elements>class Tuple;
    template<typename... Types>class Variant;
    ```
7. 变参基类：继承时可以使用变参定义基类，这在需要继承若干种类型的场景中很有用（下面的例子用来合并一些operator()）
    ```cpp
    template<typename... Bases>
    struct Overloader : Bases...
    {
        using Bases::operator()...; // OK since C++17
    };

    class Customer {};
    // 假设已经定义仿函数
    struct CustomerEq {
        bool operator() (Customer const& c1, Customer const& c2) const {
            return c1.getName() == c2.getName();
        }
    };
    struct CustomerHash {
        std::size_t operator() (Customer const& c) const {
            return std::hash<std::string>()(c.getName());
        }
    };

    using CustomerOP = Overloader<CustomerHash,CustomerEq>;
    // 以前需要填两种类型，现在一个就够（虽然实际上也有两个）
    std::unordered_set<Customer,CustomerOP,CustomerOP> mySet;
    ```

## 模板常用技巧
1. typename：说是技巧，实际上是必须的。对于模板中的类型成员的使用，必须用typename说明，否则可能会被编译器错误的解析（编译器会优先假设是一个非类型成员）。随着编译器越来越完善，必须添加的场景可能减少，但是加上总没有坏处。
    ```cpp
    template<typename T>
    struct Type {
        using SubType = T*;
    };

    // 偏特化可能出现的奇怪类型
    template<>
    struct Type<int> {
        const static int SubType{};
    };

    template<typename T>
    class MyClass {
    public:
        void foo() {
            // 如果不添加typename，SubType会被认为应当是一个静态成员，并和ptr做乘法
            // 实际上，如果SubType真的是这样的成员，这样的运算也确实是合法的
            typename T::SubType* ptr;
        }
    };
    ```
2. 零初始化：由于模板参数可能是内置类型，这些类型在类中无法被默认初始化（因为没有默认构造函数），因此对于类成员，最好的办法是在构造函数中进行初始化。
    ```cpp
    template<typename T>
    class MyClass {
    public:
        T x;
        MyClass(): x{} {}
        // 默认参数也可以这样用
        void foo(T x = T{}) {}
    }
    ```
4. 使用```this```和```XXX<T>::```明确指向模板类的成员。具体原因在13.4章节中可读。在本文的术语速览中已经给出给出，简单说这是一个非依赖基类和依赖基类的问题。加上作用域限定符，可以避免查找错误（当然你应当了解你要找的名称到底是全局中的还是基类中的）。
5. 裸数组或字符串常量的模板
    ```cpp
    // 理解模板参数的类型推断
    template<typename T>
    void arrayTemplate2(T& a) {
        cout << typeid(T).name() << endl;
        cout << is_same_v< T&, char const (&)[7]> << endl;
        cout << is_same_v< T, char const [7]> << endl;
    }

    // 提供更好的可读性
    template<typename T,int N>
    void arrayTemplate(T const (&a)[N]) {
        cout << "my array" << endl;
        cout << typeid(T).name() << endl;
        cout << typeid(a).name() << endl;
    }

    arrayTemplate2("123456");
    arrayTemplate("123456");
    ```
7. 成员模板：不管类本身是不是模板，成员模板都是很有用的一类工具。最典型的应用是提供类型转换能力。成员模板可能需要再**搭配友元类模板**声明。以允许访问其他模板参数实例化下的类型成员的访问能力。
    ```cpp
    template<typename T,typename container = deque<T>>
    class Stack {
    private:
        // 真正的存储方案
        container elems;
    public:
        template<typename U,typename cont2>
        Stack& operator=(Stack<U,cont2> const&);
        // 友元类声明，不需要给出任何模板参数，如果不做此声明，无法访问到其他模板实例的elements
        // 这里不需要声明具体参数，是因为这个参数没有被用到，所以也就没有必要给出
        template<typename,typename> friend class Stack;
    };

    // 此时如果在类外定义，语法相对繁琐一些，需要写两个template
    template<typename T, typename container>
    template<typename U, typename cont2>
    Stack<T,container>& Stack<T,container>::operator=(Stack<U,cont2> const& op2)
    {
        /* 做转型和赋值尝试 */
        elems.clear(); // remove existing elements
        // 注意
        elems.insert(elems.begin(), // insert at the beginning
            op2.elems.begin(), // all elements from op2
            op2.elems.end());
    }

    ```
8. 成员模板特化：不需要额外声明，直接定义即可。但是这里有一些问题。这里沿用上面的例子
    ```cpp
    // 这样的特化是可以的
    template<>
    template<typename U,typename Container2>
    Stack<int>& Stack<int>::operator=(Stack<U,Container2> const& op2)
    {
        elems.clear(); // remove existing elements
        elems.insert(elems.begin(), // insert at the beginning
            op2.elems.begin(), // all elements from op2
            op2.elems.end());
        cout << "? specialization" << endl;
        return *this;
    }

    // 全特化是可以的
    template<>
    template<>
    Stack<int>& Stack<int>::operator=(Stack<float> const& op2)
    {
        elems.clear(); // remove existing elements
        elems.insert(elems.begin(), // insert at the beginning
            op2.elems.begin(), // all elements from op2
            op2.elems.end());
        cout << "? specialization" << endl;
        return *this;
    }

    
    template<typename T, typename container>
    template<> // 特化不能跟在一个模板定义后面，这种特化是不允许的
    Stack<T, container>& Stack<T, container>::operator=(Stack<float> const& op2)
    {
        elems.clear(); // remove existing elements
        elems.insert(elems.begin(), // insert at the beginning
            op2.elems.begin(), // all elements from op2
            op2.elems.end());
        return *this;
    }
    ```
9. ```.template```、```::template```、```->template```的使用。在调用成员模板的时候需要显式地指定其模板参数的类型，但此时如果被编译器误认为是要执行一次比较运算就比较尴尬（和模板成员类型一样，那个需要用```typename```说明）。因此可能需要强制指定。这种情况只有在点号前面的对象依赖于模板参数的时候，且本身就在一个模板中使用时才需要注意。
才会发生。在我们的例子中
    ```cpp
    template<unsigned long N>
    void printBitset (std::bitset<N> const& bs) {
    // 并不一定真的需要，看编译器以及该模板实例化时的查找情况
        std::cout << bs.template to_string<char,
        std::char_traits<char>,
        std::allocator<char>>();
    }
    ```
10. 

## 非类型模板参数
C++对非类型模板参数的支持也是日趋完善，这类参数形如。
```cpp
template<int Val, typename T>
T addValue (T x)
{
    return x + Val;
}

// 可以替代lambda进行使用
std::transform(source.begin(), source.end(), dest.begin(), addValue<5,int>);

// 更厉害的是auto加强之后，从 C++17 开始，可以不指定非类型模板参数的具体类型（代之以 auto）
template<auto Val, typename T = decltype(Val)>
T foo() {/*..*/}

foo<123>();
```

但是整体而言，非类型模板参数的限制还是很多，主要集中在允许使用的类型。通常它们只能是整形常量（包含枚举），指向objects/functions/members 的指针，objects 或者 functions 的左值引用，或者是 std::nullptr_t（类型是 nullptr）。或者反过来说，以下的内容（截止C++20）暂时仍然是不可以的
1. 浮点型数值或者 class 类型的对象都不能作为非类型模板参数使用
2. 传递对象的指针或者引用作为模板参数时，对象不能是字符串常量，普通（非静态）临时变量或者数据成员以及其它子对象。

> 总而言之，用整数是最好的。另外C++也在发展，说不定过几年也都可以了。


## 函数模板
### 重载决议
规则
1. 普通函数允许自动类型转化，函数模板不允许或者有限制（退化）
2. 决议时只能看到前方已定义的函数和模板
3. 两个函数模板的区别只在于尾部的参数包时，优先选择没有尾部参数包的，以结束递归。

### 特殊点
1. 类型推断并不适用于默认调用参数，需要提供默认模板参数
    ```cpp
    template<typename T>
    void f(T = "") {}
    f(1); // OK: T 被推断为 int, 调用 f<int> (1)
    f(); // ERROR: 无法推断 T 的类型

    template<typename T = std::string>
    void f(T = "") {}
    ```

## 类模板
### 基本结构
由于类模板使用相对较少，这里摘抄一个基本例子
```cpp
// 由于模板的特殊性，一般使用hpp头文件
#include <vector>
#include <cassert>

template<typename T>
class Stack {
private:
    std::vector<T> elems; // elements

public:
    // 注意，对于不需要在函数签名中对T做单独使用的情况，可以直接用类名，此时默认包含T
    Stack (Stack const&); // copy constructor
    Stack& operator= (Stack const&) =default; // assignment operator

    void push(T const& elem); // push element
    
    void pop(); // pop element

    T const& top() const; // return top element

    bool empty() const { // return whether the stack is empty
        return elems.empty();
    }

    friend std::ostream& operator<< (std::ostream& strm, Stack<T> const& s) {
        // 类模板的<<等运算符重载的的最好方式，就是直接在类内定义友元函数
    }
};

// 直接在类定义内可以不写template，如果放在外面，还是要写的（否则显然找不到T啊）
template<typename T>
Stack<T>::Stack(Stack<T> const&)
{
    // do sth.
}

template<typename T>
void Stack<T>::push (T const& elem)
{
    elems.push_back(elem); // append copy of passed elem
}

template<typename T>
void Stack<T>::pop ()
{
    assert(!elems.empty());
    elems.pop_back(); // remove last element
}

template<typename T>
T const& Stack<T>::top () const
{
    assert(!elems.empty());
    return elems.back(); // return copy of last element
}
```

### 特殊点
友元。在前面的基本结构中，已经展示了一个类模板中的友元函数的定义方式（注意此时友元函数**不是模板**）。这种方式也是最推荐的。如果要将声明和定义分开。是很麻烦的一件事情。因为此时我们实际上必须要在外部定义一个```operator<<``函数模板（类内定义的时候不是）。此时一方面不能丢失模板参数的定义，另一方面也需要保证不出现模板参数冲突。这个问题虽然意义不大，但是对理解函数模板和类模板之间参数的关系有很大意义。
```cpp
// 错误示范
template<typename T>
class MyTemplate
{
private:
	T value;
public:
	MyTemplate(T t):value(T) {}

	//template<typename T>
	friend std::ostream& operator<<(std::ostream& out, MyTemplate<T> const& t);
};

// 该模板，不被认可为友元函数的实现。编译仍然会报未定义的引用。
// 个人的理解时，这个定义在友元函数声明的下方，在随类模板对友元函数实例化时，看不到这个定义
template<typename T>
std::ostream& operator<<(std::ostream& out, MyTemplate<T> const& t) {
	out << "in my template" << t.value;
	return out;
}

// 该函数，则不被认为是友元函数，无法访问value
template<>
std::ostream& operator<<(std::ostream& out, MyTemplate<int> const& t) {
	out << "in my template" << t.value;
	return out;
}
```

正确示范
```cpp
// 前置声明
template<typename T>
class Stack;
// 定义函数模板
template<typename T>
std::ostream& operator<< (std::ostream&, Stack<T> const&) { /*...*/ }

template<typename T>
class Stack {
public:
    // 注意此时，需要对其指定一个模板参数<T>，因为我们需要的是一个用T类型的实例化的函数模板
    friend std::ostream& operator<< <T> (std::ostream&, Stack<T> const&);
}
```

> ToDo：仍然存有疑问，友元函数模板，如何进行函数模板特化？

## 难点
### 移动语义
模板的设计中，对移动语义的考虑是非常重要的。在模板编程中，最重要的相关点在于利用```std::forward```，完成完美转发
1. 可变对象被转发之后依然可变。
2. Const 对象被转发之后依然是 const 的。
2. 可移动对象被转发之后依然是可移动的。

在引入模板之前，如果想要同时支持完美转发的三种情况，必须分别编程。
```cpp
class X {};
// 第一点：底层的支持肯定是单独编程的，由开发者决定三种情况的处理方式
void g (X&) {
    std::cout << "g() for variable\n";
}
void g (X const&) {
    std::cout << "g() for constant\n";
}
// 这里的X只是一个右值引用
void g (X&&) {
    std::cout << "g() for movable object\n";
}

// 第二点，也是完美转发需要处理的问题，是对这三个操作的包装
void f (X& val) {
    g(val);
}
void f (X const& val) {
    g(val);
}
void f (X&& val) {
    g(std::move(val)); // 注意想触发右值引用的重载，必须用move，否则val本身只是一个左值（其内容是一个右值引用）
}

```
这显然不够优雅。

模板编程对这种方式的有固定解决办法，也就是被称为完美转发的```forward```，以及模板参数组成的万能引用```T&&```
```cpp
template<typename T>
void f (T&& val) { // 万能引用
    g(std::forward<T>(val)); // perfect forward val to g()
}
```

但是当我们遇到一些重载决议的情况时，有会有一些新的问题。比如在对特殊函数进行模板化时。
```cpp
class Person
{
private:
    std::string name;
public:
    // generic constructor for passed initial name:
    template<typename STR>
    explicit Person(STR&& n) : name(std::forward<STR>(n)) {
        std::cout << "TMPL-CONSTR for ’" << name << "’\n";

    }
    // copy and move constructor:
    Person (Person const& p) : name(p.name) {
        std::cout << "COPY-CONSTR Person ’" << name << "’\n";
    }
    Person (Person&& p) : name(std::move(p.name)) {
        std::cout << "MOVE-CONSTR Person ’" << name << "’\n";
    }
};

//
Person p1("1234"); // ok, construct
Person p2(p1); // error!
Person p3(std::move(p1)); // ok, move construct
```
上面的想法是很美好的，我们有一个万能引用的构造，一个复制构造，一个移动构造。但是当我们遇到一个左值```Person```时，按理来说应该调用复制构造。但实际上万能引用的匹配优先级更高。因为根据 C++重载解析规则，对于一个非 const 左值的 Person p，万能引用比拷贝构造更匹配。因为它可以不用做const转型。在这里万能引用可以直接将其处理为```Person&```。我们可以单独提供一个构造函数的重载，但是这显然不犹豫。

此时就需要使用```enable_if```，来避免某些情况，进入到万能引用的匹配范围。

### enable_if
enable_if的核心原理是利用了模板匹配的SFINAE机制。如果一个实例化过程中，进行的类型替换是错误的，那这种实例化将会被忽略。enable_if的返回有两种情况
1. 模板参数列表内表达式的值为true，则返回一个类型（void或者第二个参数）
2. 模板参数列表内表达式的值为false，则替换失败

enable_if可以用在函数返回值，模板参数列表（**推荐**）。

对于上面的例子来说。
```cpp
// C++17 提供的is_convertible，这里省略了第二个模板参数的名字，因为它不必要
// 此时只有可以转换为string的类型，才能通过enable_if的替换检查
template<typename STR, typename =
    std::enable_if_t<std::is_convertible_v<STR, std::string>>>
Person(STR&& n) : name(std::forward<STR>(n)) {}
```

> enable_if_t也是enable_if的一个包装，从C++11、14，最后到17才简化为如今的形式。

更进一步的，就是C++20开始正式支持的concept特性。此时的写法将会变为
```cpp
template<typename STR>
requires std::is_convertible_v<STR,std::string>
Person(STR&& n) : name(std::forward<STR>(n)) {}
```

### 元编程


## 设计思路
1. 传值还是传引用？
    - 通常而言，建议将按引用传递用于非简单类型（POD、string_view）以外的类型，这样可以免除不必要的拷贝成本。
    - POD和string_view等，推荐按值传递。因为此时按引用传递会遇到更大的问题。比如字符串字面值，如果按引用的话，是```const char[x]```类型，这会导致每种长度的字符数组类型都作为一个单独的模板参数。
1. 对```const char*```始终保持注意。
    - 例如：不能通过将字符串字面量传递给一个期望接受 std::string 的构造函数来拷贝初始化（使用=初始化）一个对象。
1. 面向模板编程，在对类模板的对象进行编程时，要时刻注意，使用最广泛，或者最准确的接口。因为用户最终使用的类型，很可能缺少某些必要的运算符重载，或者不符合某些迭代器要求。例如以下的两种方式。
    ```cpp
    template<typename T>
    void MyPrint(T data){
        // 只需要T类型支持begin/end
        for(auto&t:data){
            cout << t << " ";
        }
        // 需要T类型支持size，随机访问
        for(size_t i=0;i!=data.size();++i){
            cout << data[i] << " ";
        }
    }
    ```
2. 

## 一些危险的错误例子

不小心返回了局部变量的常引用。
```cpp
// 这一步的传值，会在调用者的栈上构造临时变量
std::string max(std::string a, std::string b) {
	return a < b ? b : a;
}

template<typename T>
T const& max(T const& a, T const& b, T const& c)
{
    // runtime error if max(a,b) uses call-by-value
	return max(max(a, b), c);
}
```

## 未翻译章节
### 12
### 13
### 14
### 15
### 16
### 17

## 存疑
1. 章节6.4中的禁用某些成员函数。在Visual Studio2020（C++20标准）不能很好复现。