---
title: "函数式编程：合集"
date: 2024-01-15T10:26:36+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- 函数式编程
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/function-lambda.jpg
---
虽然听起来唬人，但函数式编程出现也已经有数十年的历史了。本文从应用角度出发，记录一些基础的函数式编程思想的应用方式。
<!--more-->

> 本文主要参考Gitbook《Professor Frisby's Mostly Adequate Guide to Functional Programming》

## 基本概念
1. 函数是一等公民：函数和内置类型变量、对象一样，可以存储到变量，数组中，可以作为参数传递。
2. 纯函数：相同的函数，永远得到相同的输出，而且没有任何**可以观察**到的副作用。纯函数的优越性：
    1. 可缓存性：同样的输入，永远有相同的输出。
    2. 可移植性/自文档化：函数是自给自足的，不需要依赖外部，很好观测。可以很方便的通过网络通信发送给其他机器执行。
    3. 可测试性：只需要给定输入，就能观测输出
    4. 可并行化：显然没有数据竞争 
    > 副作用包括但不限于：修改文件、修改数据库、发送通讯请求、打印日志、获取输入、访问系统状态等等
3. 柯里化(Curry)：将一个多参数函数，分成多层使用单个或多个参数的函数，返回新的函数。形式上从```f(a,b,c)```变为```f(a)(b)(c)```。
4. 函数组合(Compose)：将多个函数按需求结合调用，并返回新的函数。在内部完成了上完成了```f(g(h(x)))```。
    1. 无值Pointfree模式：定义一个函数，但不使用形参的模式，就是pointfree。在封装过程中也体现为，不调用数据的操作，而是用操作调用数据。下面给出一个简单的示例。说白了就是，返回函数的包装函数不要写参数，也没有必要写参数。
        ```js
        const _ = require('lodash');

        const compose = function(f,g) {
          return function(x) {
            return f(g(x));
          }
        };

        const myJoin = _.curry((sep,arr)=> _.join(arr,sep));
        const mySplit = _.curry((sep,data)=>_.split(data,sep));
        const initials = compose(myJoin('.'),mySplit(' '))

        console.log(initials("rex lyc love code"));
        ```
5. 范畴学（Category Theory）：数学的一个抽象分支，对集合论、类型论、群论、逻辑学等数学分支中的概念进行形式化。范畴学的讨论内容包括：对象（Object）、态射（Morphism）、变化式（Transformation）。
    1. 范畴：一个组件的搜集，具体应包含对象搜集（Object Collection）、态射及其组合的搜集、identity态射。
    2. 范畴学在程序语言中的对应：
        1. 对象：数据类型（数据类型可以认为是所有可能取值的一个集合）
        2. 态射：纯函数。一个范畴内的一个态射，表示了两个对象之间的关系。
        3. 态射组合：也就是前文所说的函数组合
        4. identity：单位元（接收输入并原样返回）
6. 声明式代码：相比于命令式给出完整的执行方式，声明式是给出一个表达式的形式(比如SQL)。当你在编写函数式代码时，实际上就是在完成声明式代码。工程上，这种形式能够提供给底层实现保留更大的优化空间。
7. Map、Reduce、Filter：最常见的三种函数式编程接口。
    1. Map：对一个对象的元素进行遍历。并返回每一个元素的处理结果成为一个列表。
    1. Reduce：对一个对象的元素进行遍历。并将处理结果聚合成一个值。
    2. Filter：对一个对象的元素进行遍历，过滤并处理一些，返回一个子集。
8. Hindley-Milner类型签名：Hindley-Milner系统对不同类型的定义
    1. 函数：```a -> b```，其中a和b分别代表输入和输出，而且都可以是函数类型
    2. 参数化：函数签名的所包含的隐藏语义。应当是在不考虑具体参数类型（或者假定参数类型可以是任意类型）的前提下，而且不再依赖任何已知信息，所能完成的功能的最小集合。这种特性能够使得我们具备对函数签名的搜索能力，根据类型签名去搜索我们所需的函数。
        > 原书在这里举了个例子，对函数```[a] -> [a]```，不能认为它可以具有排序功能，因为没有给定任何排序规则，而且类型a也不一定支持排序。但它可以是一个重排列函数。
    3. 自由定理
    4. 类型约束：将签名用作类型约束
8. 函子：
    1. Functor函子：一种函子，是实现了map函数接口，并遵守特定规则的容器类型。是最基本的运算和功能单位。将一个容器变为另一个容器。函数式编程某种程度上就是编写各种函子并进行组合的过程。
    1. Pointed函子：实现了Of方法，将new封装在Of内部，并返回函子
    1. Maybe函子：这一类函子在任意时刻，对容器内的值考虑其为空的可能性。
    1. Either函子：包含Left函子、Right函子两种情况。右值是正常，左值为异常。表达条件运算，也更适合用于处理异常。
    2. Monad函子：在函子运算过程中，经常会出现函子嵌套的场景（尤其是用函子包装不纯操作时）。Monad函子作用于有嵌套结构的函子，将其最外层的函子剥离。一个monad函子是同时实现Of、join方法的函子。是Pointed函子的升级。此外，Monad函子也经常将join和map结合，形成接口chain，简化了配对的join/map，并支持链式调用。
    3. applicative函子：实现了ap方法的Pointed函子，而且这类函子所保管的值是一个函子。对applicative函子```F```，有```F.of(x).map(f) == F.of(f).ap(F.of(x))```，对单个对象来说，map实际上等价于of + ap（获取一个函数，作用于当前对象，并返回一个对象）。就是说，ap方法能够将一个Functor作用于另一个Functor。替代Monad函子，尤其是在希望打破链式顺序，能够并行执行的场合。
    
    > 注1：applicative函子在ap方法操作过程中还保持容器类型不变。但Monad函子不保证，他可能在chain方法调用流程中变动容器类型。
9. 如何处理不纯的操作：包装不纯的部分，并延后执行，将最终的执行交给用户
10. ToDo进阶概念：自然变换（用于对函子进行类型变换）、伴随函子、极限和余极限。（11、12、13章等未学习）

## 范畴学
> 函数式编程背后由范畴学提供理论支撑，这里有必要对相关概念再进行一点展开。

[Wiki百科](https://zh.wikipedia.org/wiki/%E8%8C%83%E7%95%B4%E8%AE%BA)中给出了一些基本的描述。范畴论是对于数学结构的抽象描述。

一个范畴内：对象、态射。

对范畴的抽象：将不同的范畴本身看作一个对象，态射就是表示这些范畴之间的关系。


函子（编程中的概念）：将[范畴学概念中的函子](https://zh.wikipedia.org/wiki/%E5%87%BD%E5%AD%90)引入到编程中，是函数式编程最重要的一种数据类型、基本的功能和数据单位。
- 定义：一个容器，包含值，以及值之间的变形关系（函数）。根据需要，函子来负责实现不同的映射。
- 例子：
    - Functor函子：实现了map函数并遵循特定规则的容器。特定规则指（只含有一个属性值、实现map函数）。map函数中的输入函数，就是单个范畴内的态射。此时的Functor函子就是实现了map语义的，一个范畴间的映射。


各类函子的实现目的，并不是给出什么必不可少的语言功能。只是在满足范畴学理论的情况下，拓展出来的各种更好用的基础设施。
<!-- 插入图片？ -->

## 函数式JS
柯里化(Curry)可以使用```lodash.curry```，或者```ramda```库。形如
```js
// Node.js
const curry = require('lodash').curry;

const threeParamFunc = (a, b, c) => {
    return a + b + c;
}

const curryFunc = curry(threeParamFunc);

// 以下调用都是合法的
curryFunc(1)(2)(3);
curryFunc(1,2)(3);
curryFunc(1)(2,3);
curryFunc(1,2,3);
```

关于函数组合，在基本概念这一节已经有详细的一个描述。理解Pointfree的写法，就可以说是理解了组合。

在书中集中给出了关于[重要函数](https://mostly-adequate.gitbook.io/mostly-adequate-guide/appendix_a)、[数学结构](https://mostly-adequate.gitbook.io/mostly-adequate-guide/appendix_b)、[Pointfree工具](https://mostly-adequate.gitbook.io/mostly-adequate-guide/appendix_c)的示例代码。


## ToDo函数式C++
在C++中实现curry、compose
```cpp
// compose 简单实现，模板递归
// 存在的问题：对调用链上的输入输出类型未作检验、只支持单一参数、参数拷贝性能问题
template<typename F>
auto compose(F fh) {
	return [&](auto x)  {return fh(x); };
}

template<typename F,typename ...Funcs>
auto compose(F fh,Funcs...ft) {
	return [&](auto x)  {
		return fh(compose(ft...)(x));
	};
}

// curry 简单实现，模板递归
template <typename Func, typename... Args>
auto curry_helper(Func func, Args... args) {
    // is_invocable_v是C++17支持的，判断
	if constexpr (std::is_invocable_v<Func, Args...>) {
		// 如果函数、参数已经可以被调用了
		return std::invoke(func, args...);
	}
	else {
		// 返回一个能继续接收实参lambda函数
		return [=](auto... moreArgs) {
            // 折叠表达式
			return curry_helper(func, args..., moreArgs...);
		};
	}
}

// curry化接口
template <typename Func>
auto curry(Func func) {
    // 返回柯里化后的函数，一个可以接收任意参数的lambda表达式
	return [func](auto... args) {
		return curry_helper(func, args...);
	};
}

```


## ToDo 关于协变反变逆变不变

## 参考
[Professor Frisby's Mostly Adequate Guide to Functional Programming](https://mostly-adequate.gitbook.io/mostly-adequate-guide/)

上面的翻译[Gitbook：函数式编程指北](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/)

[阮一峰：函数式编程](https://www.ruanyifeng.com/blog/2017/02/fp-tutorial.html)