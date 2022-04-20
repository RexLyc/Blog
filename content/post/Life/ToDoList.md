---
title: "更新计划"
date: 2022-04-19T01:07:49+08:00
categories:
- 秘密
tags:
- 不公开
draft: true
---
此文档不应该被看见
<!--more-->
## 博文更新路线：
1. 修正所有错误，补充目录
1. BlogTutorial
1. linux-shell
1. spring1
1. design-pattern-init
1. git
1. cpp-compile
1. qt-all-in-one
1. jmeter-all-in-one
1. ibmmq-all-in-one
1. linux-file
1. linux-system-run
1. linux-process
1. spring2
1. cicd-jenkins
1. mysql-backend
1. mysql-command
1. mysql-practice
1. fullstack-jsts-basic
1. fullstack-jsts-aporia
1. design-concurrency
1. design-patter-terms
1. redis-single
1. WebLab-init
1. algo1

## 长期型
1. readwrite

## 网站建设
1. 搜索功能
    - 参考：
        - [Hugo 实现搜索功能小白教程](https://blog.csdn.net/weixin_44903718/article/details/108541002)
2. 博文置顶功能
3. 侧边导航
    - 基本原理就是使用Hugo从0.64（大概）开始提供的toc（table of content）变量，可以直接生成一个目录，剩余的事情就是配置一个css来美化
        ```html
        <div>{{ .TableOfContent }}</div>
        ```
    - 参考
        - [最好的一篇](https://www.bram.us/2020/01/10/smooth-scrolling-sticky-scrollspy-navigation/)
        - [这篇也可以看看](https://orianna-zzo.github.io/sci-tech/)

## 带整理博客
1. CSS学习：https://zhuanlan.zhihu.com/p/124284328