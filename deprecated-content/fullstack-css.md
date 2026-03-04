---
title: "全栈学习：CSS篇"
date: 2022-09-27T14:24:06+08:00
categories:
- 计算机科学与技术
- 全栈
tags:
- 全栈
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/fullstack.jpg
---
界面绘制从来都是一个很值得探索的问题，CSS在这方面做的很好，但也造成了其使用上的复杂性。
<!--more-->
## 概述
### 基础语法
1. 一个CSS定义由选择器、属性组构成
    ```css
    /* 选择器 {属性组} */

    /* 类选择 */
    p {color:#ff0000;}

    /* id选择 */
    #id {height:100%;}

    /* class选择 */
    .className {text-align:center;}

    /* 子元素选择（作用于子元素p） */
    div>p {}

    /* 相邻兄弟选择（即作用于紧随其后的同级元素） */
    div+p {}

    /* 后续兄弟选择（作用于其后的所有同级元素）*/
    div~p {}

    /* 属性选择器，选择所有带指定属性的标签 */
    [title] {}
    ```
1. 优先级：内联样式（标签内的） > 内部样式表（文件内的） > 外部样式表（从外部加载的）> 浏览器默认
1. 盒子模型（从外到内）：margin、border、padding、content
    - 指定元素的width、height时，只是在指定其content的宽高，实际大小还会加上内外边距和边框
### 基础属性
1. 背景：background-color/image/repeat/attachement/position
1. 文本：text-align/decoration/transform/indent、font-family/size/style/weight

### 布局
1. 定位：用于控制元素的定位方式
    - position：static（无定位，文档流）、fixed（相对浏览器窗口位置固定）、relative（相对其正常位置）、absolute（相对于父元素）、sticky（relative和fixed的结合）
        - left、top、right、bottom并非每种定位时都能生效，其生效方式也是当前元素的left对父类（或正常位置）的left、right对right
    - z-index：指定重叠时的优先顺序
1. 溢出：用于控制内容超出元素边框时的规则
    - overflow：visible、hidden、scroll、auto、inherit
1. 浮动：常用于图像和文字的组合展示（将图像浮动到文字的某一侧）
    - float
1. 对齐：水平、垂直居中都有很多种实现方式，如text-align、float、position、transform
### CSS3

### 其他
1. 移动端适配
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
1. CSS变量：CSS官方提供的用于从JS控制CSS的方式，以两个横线开头
    1. 基本用法示例
        ```css
        .div {
            width: --myVariable
        }
        ```
    1. 相关js用法
        ```js
        // 检查是否受支持
        const isSupported =
            window.CSS &&
            window.CSS.supports &&
            window.CSS.supports('--a', 0);
        
        document.body.style.setProperty('--myVariable','20px');
        document.body.style.getPropertyValue('--myVariable').trim();
        document.body.style.removeProperty('--myVariable');
        ```
### 不建议
1. !important标记：用于声明某一个样式无视级联规则，固定为最高优先级

## 参考
1. [菜鸟教程CSS](https://www.runoob.com/css/css-tutorial.html)
1. [阮一峰：CSS 变量教程](https://www.ruanyifeng.com/blog/2017/05/css-variables.html)