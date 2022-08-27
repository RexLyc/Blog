---
title: "Spring框架：关键注解大全"
date: 2021-07-26T23:59:42+08:00
categories:
- 计算机科学与技术
- Spring
tags:
- Spring
- 施工中
thumbnailImagePosition: left
thumbnailImage: images/thumbnail/spring.jpg
---
框架中有一些特别重要，或者基础而通用的注解，在这里进行一定的讲解。本文尚未完成。
<!--more-->
## 注解的原理
- 请移步至Java系列阅读。
## 常用注解
- 在此并不直接区分Spring、SpringCloud等，如有必要会单独列出。
### 配置篇
1. 基础类型数据结构的配置注入步骤：Integer、String等
    1. main类中添加注解：@EnableConfigurationProperties
    2. 需要注入的地方添加注解形如：@Value("${file-storage.path}")，花括号内为配置名称后缀，和ConfigurationProperties结合就是完整的配置名称。
        - 当使用#号时，是SpEL表达式写法。例如@Value("#{fileStorage['path']}")。Spring Expression Language，[知乎：玩转Spring中强大的spel表达式！](https://zhuanlan.zhihu.com/p/174786047)。
1. 一种Map类型配置注入步骤：
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
1. 一种Set类型配置的注入方式：
    1. 其他内容和Map注入方式相同
    1. 配置文件中写法：
        ```yml
        YourSetName:
            - a
            - b
            - c
        ```
1. 当想使用独立的配置文件时，可以引入@PropertySource来指定对应的配置文件
    ```java
    // 在resources下创建的rest.properties 文件
    connectionTimout=5000
    requestTimeout=5000
    socketTimeout=5000
    poolSize=10

    // 配置类
    @Configuration
    @PropertySource(value="classpath:rest.properties")
    public class RestTemplateConfiguration {
        // ... 用自定义配置创建Bean
    }
    ```
### 核心
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
1. @Qualifier
    1. 概述：和@Autowired搭配使用，描述bean的名称
    1. 用法：对于同时存在多个可用bean的情况，可以使用名称以指定特定bean。
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
### Web
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
1. @RequestBody
    1. 概述，提取请求中的body部分，并将其转化为指定类型。不使用本注解，则可以自行处理HttpServletRequest类型的请求数据。
    1. 对比：
        ```java
        public void add(@RequestBody int i) {}

        public void add(HttpServeletRequest requets) {}
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
    - 搭配：@ControllerAdvice/@RestControllerAdvice，为其配置异常处理，对于需要校验的内容，以@Validated、@Valid调用校验逻辑，二者各有优劣。
    - 用法：
        ```java
        public class MyEntity {
            @NotBlank
            private String name;
            @NotNull
            private String id;
        }

        @RestController
        public class MyController {
            @PostMapping("/test")
            public Response test(@Validated @RequestBody MyEntity entity) {
                //...
            }
        }
        ```
1. @Cookievalue
    1. 概述：用于获取指定的Cookie值
    1. 用法
        ```java
            @PostMapping("test")
            public void getCookieValue(@CookieValue("testId") String cookie){
                // ...
            }
        ```
1. @CrossOrigin
    1. 概述：用于支持跨域请求。
    1. 用法：施加于类或方法上。
1. @RestControllerAdvice
    1. 概述：用于在全局范围内对Controller绑定键值、预处理数据、处理异常的类型。RestControllerAdvice是组合注解，包含了@ControllerAdvice、@ResponseBody
    1. 搭配：
        - @ExceptionHandler，施加于函数，代表对某些类异常的处理
        - @InitBinder，施加于函数，用于对输入数据预处理
        - @ModelAttribute，施加于函数，用于绑定键值数据，该数据对所有受该RestControllerAdvice的Controller均可用
    1. 用法：施加于类上，凡是未经处理的异常，会在该类内部搜索是否存在对应的ExceptionHandler。
        ```java
        @RestControllerAdvice
        public class MyExceptionHandler {
            // 支持同时处理多种异常
            @ExceptionHandler(value = {CustomAException.class, CustomBException.class})
            public CustomResponse handleCustomException(Exception e) {
                // ...
            }

            // 接收所有仍然未被处理的异常类型
            @ExceptionHandler(value = Exception.class)
            public CustomResponse handleException(Exception e) {
                // ...
            }
        }
        ```
### 数据库
1. @MapperScan
    1. 概述：扫描指定包下的数据库mapper，根据SQL注解（或嵌入SQL的XML文件）对接口进行实例化
    1. 搭配：@Param用于将Mapper中的函数参数名称映射为数据库语句中的名称。
    1. 用法：在入口类添加
        ```java
        @MapperScan({"your.mapper.package.path"})
        ```
        此时的entity 和 Mapper形如
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
        public interface StationUserAccountMapper {
            // column指数据库中的列名称
            @Results(value = {
                @Result(property = "userName", column = "user_name"),
                @Result(property = "userId", column = "user_id"),
                @Result(property = "userAge", column = "user_age")
            })
            @Select("select user_name, user_id"
                    + "from user_account where user_id = #{queryUserId}")
            List<UserAccount> getUserList(String queryUserId);

            @Select("select count(1) from user_account where user_type=#{userType} and user_group=#{userGroup}")
            int getUserCount(@Param("userType")String userTpe,@Param("userGroup")String userGroup)
        }
        ```
    1. 注：SQL语句中，#引用将会把输入数据作为字符串，即添加一对儿单引号，$引用则会直接把输入数据放在语句中。$的能力更为强大，但是可能被sql注入。
1. @Transactional
    1. 概述：用于进行事务管理。
    1. 搭配：需要入口类添加，@@EnableTransactionManagement，开启对事物的支持。
    1. 用法：对函数或类型使用。

### 切面（AOP）
1. @EnableAspectJAutoProxy
    1. 概述：Spring应用中，默认AOP不开启，需要使用本注解进行开启。SpringBoot中则有自动配置。
1. @Aspect
    1. 概述：将当前类标识为一个切面供容器读取
1. @PointCut
    1. 概述：植入
1. @Around
1. @AfterReturning
1. @Before
1. @AfterThrowing
1. @After

### 异步
1. @Async
    1. 概述：对函数使用，表明该函数为一个异步调用，其所在线程由线程池负责分配。也可以自定义配置线程池、执行器、异常处理类。
    1. 搭配：入口类需要提供注解，@EnableAsync，开启对异步的支持
    1. 用法：
        ```java
        // 自定义Executor
        @Configuration
        public class MyExecutorConfiguration {
            // 一些属性
            // ...

            @Bean(name="myExecutor")
            public Executor asyncServiceExecutor() {
                // ... 创建Executor
            }
        }

        // 异常处理类
        @Configuration
        public class MyAsyncExceptionConfig implements AsyncConfigurer {
            @Override
            public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
                return new AsyncUncaughtExceptionHandler() {
                    @Override
                    public void handleUncaughtException(Throwable throwable, Method method, Object... objects) {
                        // 自定义处理...
                    }
                }
            }
        }

        // 异步方法服务类
        @Service
        public class MyService {
            // 使用自定义的Executor 可以无返回值
            @Async("myExecutor")
            public void helloWorld(){
                // 干活...
            }

            // 有返回值则需由Future和AsyncResult包装
            @Async("myExecutor")
            public Future<String> job1() {
                // 干活...
                return new AsyncResult<String>("...");
            }

            // 进一步可以根据需要使用CompletableFuture
            @Async("myExecutor")
            public CompletableFuture<String> job2() {
                return CompletableFuture.completedFuture("...");
            }
        }
        ```


### 环境
1. @Profile
1. @Conditional

### 定时
1. @Scheduled
    1. 概述：启动定时任务，需提供一个cron格式的字符串代表任务启动时间。
        - cron字符串解释：秒 分 时 日 月 星期 \[年\]
    1. 搭配：入口类需要提供注解，@EnableScheduling，启用定时任务
    1. 用法：
        ```java
        @Component
        public class MyJobs {
            // 每天早8点0分0秒执行
            @Scheduled(cron = "0 0 8 * * ?")
            public void jobOne() {
                // ...
            }
        }
        ```
    

### 测试
1. @RunWith
1. @ContextConfiguration
1. @Test
1. @ActiveProfiles

### 其他
1. @Slf4j
    - 在当前类下启用日志，内部可以直接使用成员变量log，用log.info/warn/error写日志
1. Swagger文档
    - 用于生成API文档，和Spring框架无缝衔接。以@EnableSwagger2注解配置类。
    - 注解：@Api（类）、@ApiOperation（函数）、@ApiModelProperty、@ApiModel、@ApiResponses、@ApiResponse、@ApiImplicitParams、@ApiImplicitParam
    - [官方网站](https://swagger.io/)
1. lombok
    - 概述：大幅简化JavaBean编写，避免样板式代码，主要用于自动提供构造器、Getter、Setter等。常用于各种视图类型、枚举类型。
    - 部分注解：
        - 自动生成特定函数：@Setter/@Getter、@ToString、@EqualsAndHashcode、@NoArgsConstructor/@RequiredArgsConstructor/@AllArgsConstructor、@Data、@Cleanup
        - 指示字段非空：@NonNull
        - 生成构造器模式：@Builder
        - 将类的实例声明为final：@Value，注意和配置所用的注解区分
        - 更安全的互斥锁注解：@Synchronized
    - [官方网站](https://projectlombok.org/)


## 参考资料
- [Spring常见注解大全](https://blog.csdn.net/lixiaolian123/article/details/108090499)
