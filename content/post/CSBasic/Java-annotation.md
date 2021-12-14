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
# 核心原理
1. 运行期注解：利用Java的反射机制，在运行期对注解进行解析。
1. 编译期注解
1. 字节码工程
# 内置注解
1. 标准注解：
    1. @Override：表示当前方法将覆盖父类中的方法。
    1. @Deprecated：表示当前方法已被废弃，如果被使用，编译器会发出警告。
    1. @SuppressWarnings：关闭不恰当的编译器警告信息
1. 元注解：
    1. @Target：说明注解的作用对象，可选ElementType.CONSTRUCTOR / FIELD / LOCAL_VARIABLE / METHOD / PACKAGE / PARAMETER / TYPE
        - 可以使用逗号分隔，添加多个
        - 不使用@Target则可作用于任何目标
        - 可以用于作用对象出现的任何位置，比如作为TYPE注解，则可以出现在泛型类的类型参数中，如:public class Cache<@Immutable V>
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
# 运行时注解示例代码
## 一种创建SQL数据表的注解写法
1. 注解部分
```java
// 以下注解定义分散在各自的源文件中

// 类注解，声明一个数据表
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface DBTable {
    // 表名
    String name() default "";
}

// 约束注解，声明该列的约束条件
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Constraints {
    // 默认非主键
    boolean primaryKey() default false;
    // 默认允许空
    boolean allowNull() default true;
    // 默认非唯一
    boolean unique() default false;
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQLString {
    // 列名
    String name() default "";
    // 列占用存储大小
    int value() default 0;

    Constraints constraints() default @Constraints;
}

// 本例子仅用于展示对于注解类型默认值中的成员的修改方式
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Uniqueness {
    Constraints constraints() default @Constraints(unique = true);
}

// 定义一个实际的数据表
@DBTable(name = "TESTDB")
public class TestDB {
    // 设定某一列
    @SQLString(name = "first", value = 30)
    String firstCol;
    // 设定主键
    @SQLString(name = "second", value = 50, constraints = @Constraints(primaryKey = true))
    String key;
    // 用于统计行数
    static int rowCount;
}
```
> 反思：在注解中配置注解并不是一个很好的办法。虽然看起来炫酷，但其实写起来很别扭。更普遍的做法是，让一个域拥有多个不同的注解。比如本例中的@SQLString中的@Constraints域独立出来，在使用的时候，二者平级同时使用。
2. 注解处理器部分
```java
public class SQLAnnotationProcessor {
    // 输入待处理的含数据表注解的类型名称数组
    public static void createTable(String[] classes) throws Exception {
        for (String className : classes) {
            // 反射获取实际类型
            Class<?> cl = Class.forName(className);
            DBTable dbTable = cl.getAnnotation(DBTable.class);
            if (dbTable == null) {
                System.out.println("No DBTable annotations in class: " + className);
                continue;
            }
            String tableName = dbTable.name();
            if (tableName.isEmpty()) {
                // 未设定表名，默认使用类型名代替
                tableName = cl.getName().toUpperCase();
            }
            List<String> columnDefs = new ArrayList<>();
            for (Field field : cl.getDeclaredFields()) {
                String columnName = null;
                Annotation[] anns = field.getDeclaredAnnotations();
                if (anns.length < 1) {
                    // 非数据表列定义字段
                    continue;
                }
                for (Annotation ann : anns) {
                    if (ann instanceof SQLString) {
                        SQLString sqlString = (SQLString) ann;
                        if (sqlString.name().isEmpty()) {
                            columnName = field.getName().toUpperCase();
                        } else {
                            columnName = sqlString.name();
                        }
                        columnDefs.add(columnName
                            + " VARCHAR("
                            + sqlString.value()
                            + ")"
                            + getConstraintsString(sqlString));
                    }
                }
                StringBuilder createCommandBuilder = new StringBuilder("CREATE TABLE "
                    + tableName + "(");
                for (String column : columnDefs) {
                    createCommandBuilder.append("\n " + column + ",");
                }
                String createCommand = createCommandBuilder.substring(0
                    , createCommandBuilder.length() - 1) + ");";
                System.out.println("SQL: " + createCommand);
                // do create stuff
                // ...
            }
        }
    }
}
```
> 逐个处理类型，逐个处理类型中的域，对于每个域，逐个处理其注解
## 事件监听器

# 编译时注解处理
1. 英文名称Annotation Processing Tool（APT）

# AMS

进度
核心技术 P397（但要把事件监听器补上）
编程思想 P662
