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
框架中有一些特别重要，或者基础而通用的注解，在这里进行一定的讲解。本文尚未完成。
<!--more-->
# 注解的原理
# 自定义注解方法
# 常用注解
## 配置篇
1. 基础类型数据结构的配置注入步骤：Integer、String等
    1. main类中添加注解：@EnableConfigurationProperties
    2. 需要注入的地方添加注解形如：@Value("${file-storage.path}")，花括号内为完整的配置路径
2. 一种Map类型配置注入步骤：
    1. 入口类仍然添加注解：@EnableConfigurationProperties
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
    5. 支持多级Map，例如Map<String,Map<String,String>>。
3. 一种Set类型配置的注入方式：
    1. 其他内容和Map注入方式相同
    1. 配置文件中写法：
        ```yml
        YourSetName:
            - a
            - b
            - c
        ```
## 核心
1. @ComponentScan
    1. 概述：扫描指定包下的的组件，并进行实例化
    1. 用法：
        ```java
        // 扫描Controller类型组件的示例
        @ComponentScan(basePackage = {"the.root.package.path.to.your.components"}
                    , includeFilters = @Filter(type = FilterType.ANNOTATION
                        , classes = {Controller.class, ControllerAdvice.class}))
        ```
1. @MapperScan
    1. 概述：扫描指定包下的数据库mapper，根据SQL注解（或嵌入SQL的XML文件）对接口进行实例化
    1. 用法：在入口类添加
        ```java
        @MapperScan({"your.mapper.package.path"})
        ```
    1. 注，此时的Mapper形如：
        ```java
        // entity.java
        @Data
        public class UserAccount {
            // @Result中的property指返回值中的java数据类型的成员名称
            private String userName;
            private String userId;
            private Integer userAge;
        }

        // mapper.java
        @CacheNamespace(blocking = true) // 支持SQL本地查询缓存
        public interface StationDeviceAccountMapper {
            // column指数据库中的列名称
            @Results(value = {
                @Result(property = "userName", column = "user_name"),
                @Result(property = "userId", column = "user_id"),
                @Result(property = "userAge", column = "user_age")
            })
            @Select("select user_name, user_id"
                    + "from user_account where user_id = #{queryUserId}")
            List<UserAccount> getDeviceList(String queryUserId);
        }
        ```
## Web
1. @RestController
    1. 概述：http后端的入口，一般和@RequestMapping、@PostMapping等组合使用
    1. 写法：
        ```java
            @RestController
            @RequestMapping("/path/to/your/service")
            public class YourController {
                @Autowired
                YourService yourService;

                @PostMapping("/your-service-suffix")
                public ResultType SendCmdCode(@RequestBody ParamType param) {
                    // write your code
                    // ...
                }
            }
        ```
1. @EnableDiscoveryClient / @EnableEurekaClient
    1. 概述：对于采用微服务架构的系统，该注解用于将本服务注册到注册中心。
    1. 区别：@EnableDiscoveryClient可以使用不同的注册中心(如Consul)，Eureka则只能使用Eureka注册
    1. 使用：在入口类中添加该注解，并在配置文件中添加（以使用eureka为例）
        ```yml
        spring:
          application:
            name: your-service-name # 你的服务名字

        eureka:
          instance:
            prefer-ip-address: true # 将本机ip注册到eureka
          client:
            service-url: 
              defaultZone: http://your-eureka-server-address # eureka注册中心服务的地址
        ```
    1. 注：在高版本的Spring Boot中，可以省略注册中心注解，只要在pom.xml中正确写出所用的注册中心的依赖，并在配置中进行配置即可。