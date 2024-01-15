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
draft: true

---
虽然听起来唬人，但函数式编程出现也已经有数十年的历史了。本文从应用角度出发，记录一些函数式编程思想的应用方式。
<!--more-->

## 基本概念
1. 函数是一等公民：函数和内置类型变量、对象一样，可以存储到变量，数组中，可以作为参数传递。
2. 纯函数：相同的函数，永远得到相同的输出，而且没有任何**可以观察**到的副作用。纯函数的优越性：
    1. 可缓存性：同样的输入，永远有相同的输出。
    2. 可移植性/自文档化：函数是自给自足的，不需要依赖外部，很好观测。可以很方便的通过网络通信发送给其他机器执行。
    3. 可测试性：只需要给定输入，就能观测输出
    4. 可并行化：显然没有数据竞争 
    > 副作用包括但不限于：修改文件、修改数据库、发送通讯请求、打印日志、获取输入、访问系统状态等等
3. 柯里化(Curry)：将一个多参数函数，分成多层使用单个或多个参数的函数，返回新的函数。形式上从```f(a,b,c)```变为```f(a)(b)(c)```。
4. 函数组合(Compose)：将多个函数按需求结合调用，并返回新的函数。形式上完成了```f(g(h(x)))```。
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
        2. 态射：纯函数
        3. 态射组合：也就是前文所说的函数组合
        4. identity：单位元（接收输入并原样返回）
6. 声明式代码：相比于命令式给出完整的执行方式，声明式是给出一个表达式的形式(比如SQL)。当你在编写函数式代码时，实际上就是在完成声明式代码。工程上，这种形式能够提供给底层实现保留更大的优化空间。
7. Hindley-Milner类型签名：Hindley-Milner系统对不同类型的定义
    1. 函数：```a -> b```，其中a和b分别代表输入和输出，而且都可以是函数类型
    2. 参数化：函数签名的所包含的隐藏语义。应当是在不考虑具体参数类型（或者假定参数类型可以是任意类型）的前提下，而且不再依赖任何已知信息，所能完成的功能的最小集合。这种特性能够使得我们具备对函数签名的搜索能力，根据类型签名去搜索我们所需的函数。
        > 原书在这里举了个例子，对函数```[a] -> [a]```，不能认为它可以具有排序功能，因为没有给定任何排序规则，而且类型a也不一定支持排序。但它可以是一个重排列函数。
    3. 自由定理
    4. 类型约束：将签名用作类型约束

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

## 参考
[Professor Frisby's Mostly Adequate Guide to Functional Programming](https://mostly-adequate.gitbook.io/mostly-adequate-guide/)

上面的翻译[Gitbook：函数式编程指北](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/)

