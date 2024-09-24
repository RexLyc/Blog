---
title: "全栈学习：JsTs难点篇"
date: 2021-11-30T16:56:35+08:00
categories:
- 计算机科学与技术
- 全栈
tags:
- 全栈
- 暂停施工
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/fullstack.jpg
---
本文讲解的是JavaScript和TypeScript在使用中容易犯错的点，以及一些特别重要的其他要点。
<!--more-->
## 闭包
1. 闭包中的this:
    1. 使用function进行的闭包定义，其中的this，会在运行时动态解析，导致和编程的含义不同（除非你确实希望是动态时的this）。
    1. 使用箭头函数进行闭包定义，其中的this，会绑定为定义时所在位置的this。在执行时也不会变。**推荐**
    1. 使用bind等方式，可以在运行时手动绑定this。

## 自带属性
1. name：类型名称
    ```ts
    class A{}
    console.log(A.name);
    ```

## 兼容性问题
1. 概述：兼容性问题目前主要体现在es5和es6上，虽然es6的特性非常好，但是并不能保证所有的浏览器都支持了es6，事实上很多浏览器可能支持的标准更低、更不完善。
1. babel

## Interface
1. interface是ts中非常灵活的一个种类，提供了非常强大的泛型能力，可以用来限定实现者必须具备的静态、普通成员变量、函数
1. 限定必须具备的常规成员、静态成员（但是静态成员只属于类型，不属于实例，这一点注意和其他语言区分）
    ```ts
    interface NormalMember {
        props3:string;
    }

    interface StaticMember {
        prop1:string;
        build():string;
        // 而这里又要求构造产生一个NormalMember类型
        // 相当于限制了常规的非静态成员
        new():NormalMember
    }

    class Type {
        static prop1:string="Type";
        static build(): string {
            return "";
        }
        prop3:string;
        constructor() {
            this.prop3="h";
        }
    }

    buildType(type:StaticMember){
        // 在这一行，会检查，type必须具备StaticMember里要求的成员 
        // 因为是属于类的成员，所以只能是静态成员（构造函数也是）
        return new type();
    }
    ```

## 类类型
1. C++中有模板元编程可以对类类型进行运算，Java则更是提供了完整的关于类类型的表示、处理、反射等内容，Typescript中也有类似功能，但是其实现相对更trick
1. 基本原理：利用interface、extends、Function共同完成
```ts
// 同时限定，返回值类型是T，而且是一个具有new构造函数能力的东西，也就是一种类型
export interface Type<T = any> extends Function {
    new (...args: any[]): T;
}

// 搭配InstanceOf使用，用于获取构造函数类型所构造出的实例类型
function f(t:Type){
    const a: InstanceType<Type> = new t();
}
```
> TypeScript中所谓的类，其实就是一种构造函数，因此所谓类类型，也可以叫做构造函数类型

## 一些工具
1. 序列化和反序列化

## 参考
1. [TypeScript 中文手册接口(interface)](https://typescript.bootcss.com/interfaces.html)
1. [同事问我：TypeScript中的new () => Xxx 究竟是什么惊喜？ ](https://juejin.cn/post/7032280251119960078)