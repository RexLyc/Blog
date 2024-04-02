---
title: "Spring框架开坑篇"
date: 2021-07-22T16:52:58+08:00
categories:
- 计算机科学与技术
- Spring
tags:
- 系列开坑
- Spring
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/spring.jpg
---
Java长期稳占编程语言Top3，好用的框架功不可没。Spring框架是其中的集大成者。学一点总没有错。
<!--more-->
本系列主要以讲解Spring框架中的原理为主。目标是理清一个Spring程序在执行时的行为。
## 前置知识
1. Java语言基础
2. JVM
3. 设计模式
## 发展概况
- 世界上只有两种编程语言：一种被人痛骂，一种没人使用。Java发展20多年，有很多痛点和缺陷。但不妨碍它曾经代表了语言发展的一种潮流。
- Spring框架创建之初的主要目的是用来替代更加重量级的企业级Java技术，尤其是EJB（企业级JavaBean，Enterprise JavaBean）。
- Spring是一个开源框架，由Rod Johnsom主导创建。第一本著作是《Expert One-on-One：J2EE Design and Development》
- Spring框架的最终目的是简化Java开发，学习内容核心主要是弄懂其运行原理，内容如下。
## 控制反转（Inversion of Control，IoC）
- 依赖对象的获取反转。即原来依赖类实例由当前类型自己获取，使用，销毁。但现在需要经Spring框架进行创建和注入。
- 搭配依赖注入，共同实现松耦合。

## 依赖注入DI（Dependency Injection，DI）
- 让代码之间保持松耦合（耦合是必要的，但需要被谨慎的管理），也是让POJO也能发挥魔力的方式之一。
- 通过接口来表明依赖关系，可以在类型毫不知情的情况下，替换具体实现类型。

## 面向切片编程（Aspect-Oriented Programming，AOP）
- 允许把遍布应用各处的功能分离出来形成可重用的组件。能够促进软件系统中各个组件实现关注点分离，即每个组件只关注自己的核心功能。例如日志、事务管理、安全这样的服务组件常常融入其他业务组件，最适合分离出来。
- 通过切面编程，业务代码中可以完全不用编写无关功能的代码。业务类型也完全不知道有附属组件的存在。
- 面向对象编程是将一整套业务逻辑进行封装（包含所有的附属代码），而AOP则是将一整套业务逻辑按步骤进行封装，单独抽取出其中的某一个统一的步骤，如打开连接，关闭连接，事务提交和回滚。
- 几个术语：
    1. aspect：切面，如日志
    1. jointpoint：连接点，是切面插入应用程序的地方
    1. advice：处理逻辑，实现切面功能的代码
    1. pointcut：切点，控制处理逻辑作用于哪部分的连接点
    1. target：目标类，指将会使用切面处理逻辑的类
    1. proxy：代理类，使用代理模式将目标类和切面进行结合
    1. weaving：插入，指创建代理类，将切面应用到目标类的过程

## 其他概念
1. 容器（container）：在基于Spring的应用程序中，对象生存于Spring容器中。bean工厂是最简单的容器，应用程序上下文也是基于bean工厂构建的。
2. POJO：一个不实现任何接口，不继承任何类，不需要遵守任何Java开发框架、模型要求的类型。Spring框架极力避免代码侵入，它不希望强迫使用者通过继承、实现等方式将代码和框架绑定在一起。也即是说使用者只需按照以往的方式编写类型。最坏的情况顶多是，给一个类使用了Spring注解，但它还是一个POJO。
3. JavaBean：可序列化的（实现serializable接口），具有一个无参构造器，有按照命名规范的getter、setter、is方法。由于serializable接口是一个标记接口（无方法），因此宽松情况下也可以认为JavaBean只是一个支持序列化的POJO。
4. 装配（wiring）：创建类型实例之间的运行时协作关系的行为。Spring提供XML、Java等装配方式。
5. 应用程序上下文（Application Context）：装载bean的定义并负责装配的组件。Spring提供多种应用程序上下文，如从java配置类代码、xml配置中加载。
6. 样板式代码（boilerplate code）：为了实现通用的和简单的任务，不得不一遍遍重复编写的代码。（经典案例是JDBC的初始化、查询、异常处理代码）

## Spring框架典型构成
![Spring框架运行时结构](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/images/spring-overview.png)
图片来自[Spring官网，4.2.x版本](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/overview.html)，最新版本可能有变动。

## 工具
1. javap：java自带的反汇编工具，和idea不同，仅把字节码做简单可视化，保留字节码原汁原味


## Spring进化
1. Spring Framework：就是指Spring的基础框架，提供一个轻量化的Bean模型，提供控制反转、依赖注入等能力。
1. Spring Boot：继承了若干使用的模块，能够良好的完成一个单机服务。
1. Spring Cloud：进一步整合了若干微服务模块，如网关、熔断、注册中心、服务发现。但实际上，Spring Cloud的定位有些尴尬，大部分微服务模块在各个公司内部都有自己自研或者其他开源方案代替。使用的时候可以辩证的对比一下。

## 参考资料
- [Spring官方文档5.0.0M1版本](https://docs.spring.io/spring-framework/docs/5.0.0.M1/javadoc-api/overview-summary.html)
- [SpringBoot Reference Guide](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/)
- 《Spring实战》
- 《Spring5核心原理》
- 《精通Spring》
- 《精通Spring 4.X:企业应用开发实战》
- 《从零开始造Spring》
- Spring源代码（打断点跟进阅读）
- [Spring面试题，35道Spring八股文](https://javabetter.cn/sidebar/sanfene/spring.html#_3-spring-%E6%9C%89%E5%93%AA%E4%BA%9B%E5%B8%B8%E7%94%A8%E6%B3%A8%E8%A7%A3%E5%91%A2)
- [SpringBoot启动流程及其原理](https://www.cnblogs.com/theRhyme/p/11057233.html)