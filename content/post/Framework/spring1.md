---
title: "Spring框架-Bean详解"
date: 2021-07-26T09:42:07+08:00
categories:
- 计算机科学与技术
- Spring
tags:
- Spring
- 暂停更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/spring.jpg
---
Bean作为Spring程序构成的基础之一，其行为奠定软件的基础功能。本章主要介绍Bean的部分原理和实用操作。
<!--more-->
## 什么是Bean
Spring Bean是被Spring实例化、组装并由Spring容器管理的Java对象。默认情况下，Spring中的bean都是单例的。
## 定义方式
1. 用@Component注解（或其衍生物）注解一个Java类型
2. 编写配置类型（带有@Configuration的类型），使用@Bean注解编写一个工厂方法返回Bean
3. 通过XML配置声明Bean定义（不推荐使用）
> 更推荐使用类型安全的1和2来完成定义。
## 装配
1. 装配：创建应用组件之间协作的行为
1. （推荐）自动化的装配：使用@ComponentScan来自动化扫描，对于支持Bean定义，将会自动生成Bean。搭配@Autowired注解，将会自动把生成的Bean注入到依赖的对象中。
    ```java
    @Service
    class MyService {
        // 成员变量
        @Autowired
        private Dependency0 dep0;

        // 构造函数
        @Autowired
        public MyService(Dependency1 dep1) {
            // ...
        }

        public SaySth(@Autowired String words) {
            // ...
        }

        // 成员函数
        @Autowired
        public DoSth(Dependency2 dep2) {
            // ...
        }
    }
    ```
    - @Autowired可以用于成员变量、构造函数、参数、成员函数。由于bean生命周期的先后顺序，对构造函数添加注解更稳妥（避免在构造函数中使用null的成员变量）。
    - @Resource、@Reject也可以完成注入，三者在注入方式上略有区别，有的以类型为主，有的以名称为主。
    - 自动注入存在两种可能性，如果没找到任何bean或找到多个符合条件的bean，默认情况都会报错。可以用@Qualifier、@Primary指定。
1. 显式装配，编写配置类型@Configuration，用@Bean返回。一般会手动调用new来创建对象。
1. xml跳过（已经不再推荐使用）
## 关键原理
1. @ComponentScan：
    - 处理器ComponentScanAnnotationParser，位于org.springframework.context.annotation包。大致流程（有待确认）：配置类加载→ComponentScan→加载所有已找到的bean的定义→AbstractApplicationContext开始创建bean。
    - ComponentScan过程中，扫描的是.class字节码文件，从中寻找@Component注解。（findCandidateComponents）
1. @Scope:
    - 负责控制组件的生命周期。作用于@Component类型，或@Bean函数前。
    - 常用选项包括：
        - "prototype"：每一次populateBean都产生一个新的Bean
        - "singleton"：全局唯一的单例模式
        - "request"：每一个http request产生一个
        - "session"：每一个http session产生一个
1. @Configuration：
    - 对于一些无法自动注入的场景，可以手动封装到@Configuration注解的类内部。
    - 基本原理是：spring应用启动时，所有被Configuration注解的类会被记录，等到后续注入了相关类型时，会用这些记录生成具体的bean并注入。
1. @Bean：
    - 作为显式装配的必要注解，对一个返回Bean的工厂函数进行注解。
    - 工厂函数可以拥有参数，参数内容将会自动注入（隐含@Autowired）
        - 可以搭配@Primary、@Qualifier区分自动注入的Bean来源
1. @Autowired
    - 在bean实例化过程，属性注入时会将需要的bean注入。
    - 关键函数：buildAutowiringMetadata。
> 注解和注解处理器的代码都可以直接阅读，如果不理解了，网上搜不到好的，不妨直接去对应的包下面找一找。或者通过调试，跟踪注解的启动流程（仅限RUNTIME类型）。
## 一个bean的典型生命周期
1. 大的阶段就四个：实例化、填充属性、初始化、销毁。此外还有一个加载过程，虽然不属于生命周期，但也很重要。
1. 加载
    1. 通过BeanDefinitionReader加载Bean定义，整理为BeanDefinition
    1. Bean注册到BeanDefinitionRegistry
    1. 注册后置增强器（BeanFactoryPostProcessor，可能有多个）等附属步骤
    1. BeanDefinition合并阶段，自子向父合并一个Bean的内部的属性
1. 实例化（instantiation）
    1. 实例化过程前置处理
    1. 实例化：通过反射进行。（思考为什么不能用new？动态编译问题、private属性问题）
    1. 实例化过程后置处理
    1. 属性修改：可对属性值进行添加、修改、删除。（此时的属性值还未被填充到具体实例）
1. 填充属性（Populate）
    1. 用户属性赋值
    1. 容器属性赋值（执行实现了Aware接口的对应方法）
1. 初始化（Initialization）
    1. Bean Aware接口回调阶段，如
        1. 调用setBeanName方法【需要实现BeanNameAware接口】
        1. 调用setBeanFactory方法【需要实现BeanFactoryAware接口】
        1. 调用setApplictionContext方法【需要实现ApplicationContextAware接口】
    1. 调用初始化前置方法【需要实现BeanPostProcessor接口】
    1. 初始化：三种方式，注解@PostConstruct、调用afterPropertiesSet方法（实现InitializingBean接口）、定义时指明@Bean(initMethod="")
    1. 调用初始化后置方法【需要实现BeanPostProcessor接口】
1. bean开始使用
1. 销毁（Destruction）
    1. 调用注解方法，@PreDestroy
    1. 容器关闭后：调用destroy方法【需要实现DisposableBean接口，或在用xml等方式装配时制定了销毁方法】
    1. 调用自定义销毁方法，定义于@Bean(destroyMethod="")
    1. 如果是非单例模式，bean将跳过上述步骤，返回给用户处理
> 分清楚初始化方法和构造函数的区别，构造函数和构造块会在实例化阶段被调用。
> 由于目前推荐使用基于注解的Bean方式，所以大部分相关接口的实现都是以注解（Annotation）或类似开头，如AnnotatedBeanDefinitionReader，AnnotationConfigApplicationContext。
## 代码示例
```Java
// 在使用处Autowired就能看到效果
// 可以实现各种接口查看构造阶段、销毁阶段的顺序
@Component
public class PojoTest implements BeanFactoryAware {
    private BeanFactory myBeanFactory;

    public PojoTest(){
        System.out.println("in self construct");
    }

    {
        System.out.println("in construct block");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("in setBeanFactory");
        this.myBeanFactory=beanFactory;
    }
}
```

## 扩展点
1. 注意到其实在整个流程中，对于Spring框架的使用者来说。最有用的就是各种前置、后置处理器，Pre&PostProcessor。可以根据需求，在自己的程序中合理使用、扩展这些处理器。

## 参考资料
- [史上最通俗易懂的ASM教程 知乎](https://zhuanlan.zhihu.com/p/94498015?utm_source=wechat_timeline)
- [可以硬刚面试官的：Spring IOC详解 以及 Bean生命周期详细过程](https://baijiahao.baidu.com/s?id=1700981758220165177&wfr=spider&for=pc)
- [Spring Bean生命周期doCreateBean源码阅读](https://blog.csdn.net/weixin_42955916/article/details/117824749?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-117824749-blog-109271899.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-117824749-blog-109271899.pc_relevant_default&utm_relevant_index=1)
- [Spring | 深入理解Bean的生命周期](https://blog.csdn.net/weixin_42886699/article/details/122693816)
- [Spring bean 生命周期详解](https://blog.csdn.net/weixin_44145478/article/details/120217272)