---
title: "Spring框架开坑篇（施工中）"
date: 2021-07-22T16:52:58+08:00
categories:
- 计算机科学与技术
- Spring
tags:
- 系列开坑
- Spring
thumbnailImagePosition: left
thumbnailImage: https://gitee.com/rexlyc/BlogMedia/raw/master/thumbnail/spring.jpg
---
Java长期稳占编程语言Top3，好用的框架功不可没。Spring框架是其中的集大成者。学一点总没有错。
<!--more-->

# 发展概况
- 世界上只有两种编程语言：一种被人痛骂，一种没人使用。Java发展20多年，有很多痛点和缺陷。但不妨碍它曾经代表了语言发展的一种潮流。
- Spring框架创建之初的主要目的是用来替代更加重量级的企业级Java技术，尤其是EJB（企业级JavaBean，Enterprise JavaBean）。
- Spring是一个开源框架，由Rod Johnsom主导创建。第一本著作是《Expert One-on-One：J2EE Design and Development》
- Spring框架的最终目的是简化Java开发，学习内容核心主要是弄懂其运行原理，内容如下。
# 控制反转

# 依赖注入DI（Dependency Injection）
- 让代码之间保持松耦合（耦合是必要的，但需要被谨慎的管理），也是让POJO也能发挥魔力的方式之一。
- 依赖注入的方式

# 面向切片编程

# 其他概念
1. 容器
2. POJO：一个不实现任何接口，不继承任何类，不需要遵守任何Java开发框架、模型要求的类型。Spring框架极力避免代码侵入，它不希望强迫使用者通过继承、实现等方式将代码和框架绑定在一起。也即是说使用者只需按照以往的方式编写类型。最坏的情况顶多是，给一个类使用了Spring注解，但它还是一个POJO。
3. JavaBean：可序列化的（实现serializable接口），具有一个无参构造器，有按照命名规范的getter、setter、is方法。由于serializable接口是一个标记接口（无方法），因此宽松情况下也可以认为JavaBean只是一个支持序列化的POJO。

# 参考资料
《Spring实战》

《Spring5核心原理》

《精通Spring》

《精通Spring 4.X:企业应用开发实战》