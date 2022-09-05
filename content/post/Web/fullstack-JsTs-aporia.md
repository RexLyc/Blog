---
title: "全栈学习：JsTs难点篇"
date: 2021-11-30T16:56:35+08:00
categories:
- 计算机科学与技术
- 全栈
tags:
- 系列开坑
- 全栈
- 施工中
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


## 兼容性问题
1. 概述：兼容性问题目前主要体现在es5和es6上，虽然es6的特性非常好，但是并不能保证所有的浏览器都支持了es6，事实上很多浏览器可能支持的标准更低、更不完善。
1. babel