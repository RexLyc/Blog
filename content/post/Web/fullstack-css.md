---
title: "全栈学习：CSS篇"
date: 2022-09-27T14:24:06+08:00
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
界面绘制从来都是一个很值得探索的问题，CSS在这方面做的很好，但也造成了其使用上的复杂性。
<!--more-->
## 移动端适配
1. 主要目的是对于不同宽度、像素数量的设备，进行更好的界面展示
1. 常用适配方式
    1. @media
    ```css
    <!-- 当屏幕宽度最大为1024像素时 -->
    @media only screen and max-width1024px {
    .your-class {    }
    #your-item {    }
    }
    ```
1. 特别的：
    1. 换行:white-space和word-break属性，根据需求配置
    ```css
    <!-- 代码段 -->
    code {
        white-space: pre-wrap;
    }
    <!-- MathJax  -->
    mjx-math {
        white-space: normal;
    }
    <!-- 普通部分 -->
    body {
        word-break: break-all;
    }
    ```
## 不建议
1. !important标记：用于声明某一个样式无视级联规则，固定为最高优先级