---
title: "全栈学习：JsTs基础篇"
date: 2021-11-30T16:48:25+08:00
categories:
- 计算机科学与技术
- 全栈
tags:
- 全栈
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/fullstack.jpg
---
本文讲解Javascript和Typescript的基础概念和基本语法。
<!--more-->
## 概述
1. Javascript是一种轻量级的，解释型编程语言。虽然是以为浏览器页面服务而开发的脚本语言。但也可以用于一些非浏览器的环境。
    - 标准名为ECMAScript，即ES，目前支持较广的高版本为ES6。
1. Typescript是Javascript的超集。由于Js有一些设计上的问题（或者说特性），在面向大规模工程时，并不是特别好的选择。因此Ts应运而生。

## 坑
1. NaN：数值是否有效，只能用isNaN函数进行判断
1. null和undefined，==和===：
    - 空变量===undefined成立，而且空变量!==null成立，但空变量==null也成立
    - null==undefined成立，但null!==undefined成立
    - 总之：==和===的主要区别是==会尝试进行类型转换，转换后相等即可，===要求类型内容完全一致

## 推荐工具
[一个有依赖库的在线运行环境：JavaScript PlayGround](https://playcode.io/)