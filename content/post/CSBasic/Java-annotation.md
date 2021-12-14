---
title: "Java系列：注解篇"
date: 2021-12-13T14:44:54+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Java系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/java.jpg
---
注解是Java所提供的一种特别的编程方式。从Java5被引入，在Spring框架中应用尤为广泛。
<!--more-->
# 为什么要使用注解
1. 注解能够将一些元数据直接写入代码当中。而不需要另写文档。
1. 代码更加易于阅读，并且拥有了编译期类型检查的能力。
1. 可以用于控制编译期代码的生成。
> 总之，一旦你的代码中出现了重复性的工作，你就可以考虑使用注解来简化、自动化该过程。
# 内置注解
1. 标准注解：
    1. @Override：表示当前方法将覆盖父类中的方法。
    1. @Deprecated：表示当前方法已被废弃，如果被使用，编译器会发出警告。
    1. @SuppressWarnings：关闭不恰当的编译器警告信息
1. 元注解：
    1. @Target：说明注解的作用对象，可选ElementType.CONSTRUCTOR / FIELD / LOCAL_VARIABLE / METHOD / PACKAGE / PARAMETER / TYPE
        - 可以使用逗号分隔，添加多个
        - 不使用@Target则可作用于任何目标
    1. @Retention：说明注解的作用级别，可选RetentionPolicy.SOURCE/CLASS/RUNTIME
    1. @Documented：用于生成说明文档
    1. @Inherited：允许子类继承父类的注解
# 注解的定义
1. 注解的定义很像接口的定义。事实上，注解也确实会一样被编译成class文件。例如@Test注解
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {}
```
2. 组成部分
    1. 元注解：定义注解时所需要给出的一些基本信息
    2. 注解的成员：在处理注解时可以使用，可指定默认值，使用时需要提供无默认值的所有其他值
        - 支持的成员类型包括：所有基本类型（int，float，boolean等），String，Class，enum，Annotation，以上类型的数组
        - 不允许以null作为默认值。如果想要表现默认无效的状态，需要自行约定，如使用数值使用-1，串使用空字符串等。
```java
public @interface Example {
    public int id();
    public int name() default "Example Name";
}
```
3. 编写注解处理器
    1. 注解处理器需要使用反射，通过反射来获取施加给类型、函数等的注解。只有编写了注解处理器，注解才能够被正确的使用。例如对于@Test来说，如果不使用测试工具，则该注解并没有意义。而有些工具则可能会删除带有该注解的函数，以生成纯净的代码。
```java
public class ExampleTracker {
    private class ExampleCases {
        @Example(id = 233, name = "233?")
        public void sayHello() {
            System.out.println("hello");
        }

        @Example(id = 666)
        public boolean isOk(String name) {
            return name.equals("ok");
        }
    }

    public static void ExampleTest(class<?> cl) {
        for(Method m: cl.getDeclaredMethods()) {
            Example ex = m.getAnnotation(Example.class);
            if (ex != null) {
                System.out.println("found example: " + ex.id() + " " + ex.name());
            }
        }
    }

    public static void main(String args[]) {
        ExampleTest(ExampleCases.class);
        return;
    }
}
```
# 示例代码
## 一种创建SQL数据表的注解写法
```java
// 以下注解定义分散在各自的源文件中

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface DBTable {
    String name();
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Constraints {
    boolean primaryKey() default false;

    boolean allowNull() default true;

    boolean unique() default false;
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQLString {
    String name() default "";

     value() default 0;

    Constraints constraints() default @Constraints;
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Uniqueness {
    Constraints constraints() default @Constraints(unique = true);
}

@DBTable(name = "TESTDB")
public class TestDB {
    @SQLString(name = "first", value = 30)
    String firstCol;
    @SQLString(name = "second", value = 50, constraints = @Constraints(primaryKey = true))
    String key;
    static int rowCount;
}
```
## 