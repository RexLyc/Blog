---
title: "Spring框架-Bean详解"
date: 2021-07-26T09:42:07+08:00
categories:
- 计算机科学与技术
- Spring
tags:
- Spring
thumbnailImagePosition: left
thumbnailImage: https://gitee.com/rexlyc/BlogMedia/raw/master/thumbnail/spring.jpg
---
Bean作为Spring程序构成的基础之一，其行为奠定软件的基础功能。本章主要介绍Bean的部分原理和实用操作。
<!--more-->
# 什么是Bean
Spring Bean是被Spring实例化、组装并由Spring容器管理的Java对象。
# 定义方式
1. 用@Component注解（或其衍生物）注解一个Java类型。
2. 编写配置类型（带有@Configuration的类型），使用@Bean注解编写一个工厂方法返回Bean。
3. 通过XML配置声明Bean定义。
> 更推荐使用类型安全的1和2来完成定义。
# 装配
1. （推荐）自动化的装配：使用@ComponentScan来自动化扫描，对于支持Bean定义，将会自动生成Bean。搭配@Autowired注解，将会自动把生成的Bean注入到依赖的对象中。
    - @Autowired可以用于成员变量、构造函数、成员函数。由于bean生命周期的先后顺序，对构造函数添加注解更稳妥（避免在构造函数中使用null的成员变量）。
    - 自动注入存在两种可能性，如果没找到任何bean或找到多个符合条件的bean，默认情况都会报错。
2. 显式装配，编写配置类型@Configuration，用@Bean返回。一般会手动调用new来创建对象。
3. xml跳过（不推荐使用）
# 一些细节
1. 默认情况下，Spring中的bean都是单例的。
2. 
# 一个bean的典型生命周期
1. 实例化（instantiation）
2. 填充属性（Populate）
3. 初始化（Initialization）
    1. 调用setBeanName方法【可选，需要实现BeanNameAware接口】
    2. 调用setBeanFactory方法【可选，需要实现BeanFactoryAware接口】
    3. 调用setApplictionContext方法【可选，需要实现ApplicationContextAware接口】
    4. 调用预初始化方法【可选，需要实现BeanPostProcessor接口】
    5. 调用afterPropertiesSet方法【可选，需要实现InitializingBean接口，或在用xml等方式装配时指定了初始化方法】
    6. 调用自定义初始化方法
    7. 调用初始化后方法【可选，需要实现BeanPostProcessor接口】
4. bean开始使用
5. 销毁（Destruction）
    1. 容器关闭后：调用destroy方法【可选，需要实现DisposableBean接口，或在用xml等方式装配时制定了销毁方法】
    2. 调用自定义销毁方法
> 分清楚初始化方法和构造函数的区别，构造函数和构造块会在实例化阶段被调用。

```Java
//在使用处Autowired就能看到效果
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