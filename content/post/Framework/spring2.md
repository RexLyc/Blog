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
- 请移步至Java系列阅读。
# 常用注解
- 在此并不直接区分Spring、SpringCloud等，如有必要会单独列出。
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
1. @Component
    1. 概述：用于声明一个类型，用于后续的实例化。结合@ComponentScan将会自动创建。
    1. 用法：直接添加在对应类上面
    1. 注意：@Service注解其实本质是@Component的别名，为了更好的区分业务。
1. @Bean
    1. 概述：用于声明一个Spring框架的Bean。
    1. 用法：直接添加在对应类上面。使用该注解的类型，需要由其他代码手动创建实例。
1. @Resource
    1. 概述：自动注入
    1. 用法：和Autowired的区别在于，Resource是按照名称匹配
1. @Autowired
    1. 概述：变量自动注入的常用写法。
    1. 用法：可直接对变量、方法和构造方法使用，完成对其中内容自动装配。按照类型匹配。
1. @Configuration
    1. 概述：用于手动创建Bean。内部一般会搭配@Bean使用。
    1. 示例：
        ```java
        @Configuration
        public class MyServerEndPoint {
            @Bean
            public ServerEndpointExporter serverEndpointExporter() {
                return new ServerEndpointExporter();
            }
        }
        ```
1. @PostConstruct
    1. 概述：**Java本身**提供的注解，在构造函数后执行。应用于非静态void函数。
    1. 顺序：Constructor(构造方法) -> @Autowired(依赖注入) -> @PostConstruct(构造后)
1. @PreDestroy
    1. 概述：在销毁之前执行
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
1. @ServerEndpoint
    1. 概述：创建WebSocket服务器端的一种方式。主要需要实现一些固定接口：onOpen、onClose、onMessage、onError.
    1. 注意本方法创建WebSocket服务器，需要搭配一个@Configuration用于创建ServerEndpointExporter。
    1. 用法示例：
        ```java
        // Configuration.java
        @Configuration
        public class WebSocketServerConfig {
            @Bean
            public ServerEndpointExporter serverEndpointExporter() {
                return new ServerEndpointExporter();
            }
        }
        // MyWebSocketServer.java
        @ServerEndpoint(value = "/your/ws/path/{param}") // 可以添加路径变量
            private Session session;
            private String param;
            // 一般会使用一个静态变量来管理所有连接的信息
            private static Map<String, Session> sessionPool = new ConcurrentHashMap<>();

            @OnOpen
            public void onOpen(Session session, @PathParam(value = "param") String param) {
                this.param = param;
                sessionPool.put(this.param, session);
            }

            @OnClose
            public void onClose() {
                sessionPool.remove(this.param);
                session.close();
            }

            @OnMessage
            public void onMessage(@PathParam(value = "param") String param, String message, Session session) {
                // 处理从客户端接收到的消息
                // ...
            }

            public void send(String message) {
                // 发送消息到当前会话的客户端
                this.session.getAsyncRemote().sendText(message);
            }

            @OnError
            public void onError(Session session, Throwable throwable) { // 错误处理
                onClose();
            }
        ```
1. 字段校验：
    - @NotEmpty、@NotBlank、@Null、@NotNull、@AssertTrue、@AssertFalse、@Pattern、@Email、@Min、@Max、@DecimalMin、@DecimalMax、@Size、@Digits、@Past、@Future...
# 数据库
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
1. @Transactional

## 切面（AOP）
1. @Aspect
1. @PointCut

## 异步
1. @Async

## 环境
1. @Profile
1. @Conditional

## 定时
1. @Scheduled

## 开关
1. @EnableAsync
1. @EnableScheduling
1. @EnableConfigurationProperties
1. @EnableTransactionManagement
1. @EnableCaching

## 测试
1. @RunWith
1. @ContextConfiguration
1. @Test
1. @ActiveProfiles

# 参考资料
- [Spring常见注解大全](https://blog.csdn.net/lixiaolian123/article/details/108090499)
