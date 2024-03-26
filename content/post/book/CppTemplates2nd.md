---
title: "C++ Templates 第二版：读书笔记"
date: 2024-03-26T20:41:51+08:00
categories:
- 计算机科学与技术
- C++
tags:
- C++
- 读书笔记
# thumbnailImagePosition: left
# thumbnailImage: //example.com/image.jpg
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
4. 调用参数：函数模板中的形参
5. 类型退化（decay）：一套固定规则，去掉const、volatile、引用等，数组变指针。
6. 模板参数推断（推导）：模板参数在使用时可以只是类型的一部分（如```const T&```）。在此基础上类型推断是受限制的。
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
7. 模板重载：注意不是指特化，模板是可以重载的（完全不同的模板参数数量和类型），只要在实例化阶段保证只有一个模板匹配即可。但这种重载可能出现隐藏的错误。最好还是只做特化。

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
// 头文件
#include <vector>
#include <cassert>

template<typename T>
class Stack {
private:
    std::vector<T> elems; // elements

public:
    // 注意，对于不需要在函数签名中对T做单独使用的情况，可以直接用类名
    Stack (Stack const&); // copy constructor
    Stack& operator= (Stack const&) =default; // assignment operator

    void push(T const& elem); // push element
    
    void pop(); // pop element

    T const& top() const; // return top element

    bool empty() const { // return whether the stack is empty
        return elems.empty();
    }
};

// 源文件

// 虽然头文件中可以省略类模板参数，但是在源文件中定义的话，还是要有的
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

## 设计思路
1. 传值还是传引用？
    - 通常而言，建议将按引用传递用于非简单类型（POD、string_view）以外的类型，这样可以免除不必要的拷贝成本。
    - 按值的情况：一些类型按引用传递会遇到更大的问题（有待验证？）

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