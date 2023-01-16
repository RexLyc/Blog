---
title: "Java系列：构建篇"
date: 2023-01-11T15:54:13+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Java系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/java.jpg
---
本文总结JVM系列语言在构建时的所有相关内容，和Android篇会有一定重叠。
<!--more-->
## 构建工具
### Maven
1. 基于XML的构建配置管理工具，广泛使用于Spring相关项目。使用上局限性比较大，性能比Gradle差一些，但是其呆板的使用方式一定程度也避免了部分“大聪明”乱来。
1. 常用指令示例
```bash
# 将本地jar包手动安装到本地仓库
# packaging可选 jav、war、maven-plugin、pom等
mvn install:install-file -DgroupId=xxx -DartifactId=xxx -Dversion=xxx -Dpackaging=jar -Dfile=xxx
```

### Gradle
1. 基于Kotlin-Based DSL(领域特定语言)，吸收了Ant和Maven的一些设计思想，相对更简洁、灵活，性能更强大。但这份灵活对于大型项目也是一个双刃剑。

### Ant
1. 原始的管理方法，已经基本销声匿迹

## 参考
1. [Gradle与Maven的区别](https://www.jianshu.com/p/7248276d3bb5)
1. [Gradle官方做的和Maven的对比](https://gradle.org/maven-vs-gradle/)