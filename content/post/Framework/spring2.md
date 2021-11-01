---
title: "Spring框架-关键注解大全"
date: 2021-07-26T23:59:42+08:00
categories:
- 计算机科学与技术
- Spring
tags:
- Spring
- 施工中
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/spring.jpg
---
框架中有一些特别重要，通用的注解，在这里进行一定的讲解。本文尚未完成。
<!--more-->
# 配置篇
1. 基础类型数据结构的配置注入步骤：Integer、String等
    1. main类中添加注解：@EnableConfigurationProperties
    2. 需要注入的地方添加注解形如：@Value("${file-storage.path}")，花括号内为完整的配置路径
2. 一种Map类型配置注入步骤：
    1. main类仍然添加注解：@EnableConfigurationProperties
    2. 注入位置所在类型添加注解：
        ```java
        @Data
        @ConfigurationProperties(prefix="ibmmq-rocketmq") // 指定Map在配置中的路径前缀
        ```
    3. 注入位置不需要注解，代码形如：
        ```java
        // 注意topicMap为配置中的Map名称
        private Map<String,String> topicMap = new HashMap<>() 
        ```
    4. 此时的配置文件形如：
        ```yml
        ibmmq-rocketmq:
            topicmap:
                A: 1
                C: 2
                E: 3
        ```