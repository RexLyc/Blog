---
title: "Java系列：框架和标准篇"
date: 2024-03-21T22:32:05+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Java系列
- 框架
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/java.jpg
draft: true
---
本章节记录一些Java语言，接近标准的一些框架，库。
<!--more-->

## Servlet
和Servlet概念相关的一共有两个部分：Servlet容器、Servlet。

Servlet容器也叫做Servlet引擎，是Web服务器或应用程序服务器的一部分，用于在发送的请求和响应之上提供网络服务，解码基于 MIME的请求，格式化基于MIME的响应。

Servlet没有main方法，不能独立运行，它必须被部署到Servlet容器中，由容器来实例化和调用 Servlet的方法（如doGet()和doPost()），Servlet容器在Servlet的生命周期内包容和管理Servlet。在JSP技术 推出后，管理和运行Servlet/JSP的容器也称为Web容器。

生命周期：
1. 实例化：在第一次访问或启动tomcat时，tomcat会调用无参构造方法实例化servlet容器。
1. 初始化：tomcat在实例化此servlet容器后，会立即调用init方法初始化servlet容器。
1. 就绪：容器收到请求后调用servlet的service方法来处理请求。HttpServlet的service()方法会根据不同的请求转调不同的doXxx()方法,比如 doGet, doPost。
1. 销毁：容器依据自身算法删除servlet对象，删除前会调用destory方法

## Tomcat
Tomcat是一个免费的开放源代码的Servlet容器。在Spring中，当依赖spring-boot-web-starter之后，就会通过```ServletWebServerFactoryAutoConfiguration```，添加Web服务器，默认tomcat，

## 参考
- [知乎：几个概念：Servlet、Servlet容器、Tomcat](https://zhuanlan.zhihu.com/p/40249834)