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
```

另外注意书中经常使用的写法是```T const&```，而不是```const T&```，前者实际上更符合从右向左念的方式。

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
15. 类成员别名模板：对类模板的成员，取别名。这种技术也是上面```is_same_t```等类型萃取模板的实现方式。
    ```cpp
    // 这里是少数必须使用typename而不能用class的场景，用于说明等号右侧是一个类型，而不是成员变量/函数
    template<typename T>
    using MyTypeIterator = typename MyContainerType<T>::iterator;
    ```
16. 聚合类：特殊的class/struct，没有用户定义的显式的，或者继承而来的构造函数，没有 private 或者 protected 的非静态成员，没有虚函数，没有 virtual，private 或者 protected的基类。表现上就是最传统的C风格struct。聚合类也支持模板化，而且可以用推断指引（也只能用，毕竟没有构造器）的方式，支持模板类型推导。


## 模板常用工具
### auto

### decltype：
1. 一些特例：
   1. ```template<decltype(auto) N>```，此时非类型模板参数N的类型，会被推导为引用类型。

### 变长参数模板


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


## 设计思路
1. 传值还是传引用？
    - 通常而言，建议将按引用传递用于非简单类型（POD、string_view）以外的类型，这样可以免除不必要的拷贝成本。
    - POD和string_view等，推荐按值传递。因为此时按引用传递会遇到更大的问题。比如字符串字面值，如果按引用的话，是```const char[x]```类型，这会导致每种长度的字符数组类型都作为一个单独的模板参数。
1. 对```const char*```始终保持注意。
    - 例如：不能通过将字符串字面量传递给一个期望接受 std::string 的构造函数来拷贝初始化（使用=初始化）一个对象。

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