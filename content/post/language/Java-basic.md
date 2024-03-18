---
title: "Java系列：基础难点总结篇"
date: 2022-08-23T11:17:29+08:00
categories:
- 计算机科学与技术
- 编程语言
tags:
- Java系列
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/java.jpg
---
本文总结一些常见的较为特别的Java基础要点
<!--more-->
## 类型大小
1. 基本类型不再复述，记得char是两个字节
1. 类类型的大小分为四个部分：
    - 对象头：包含jvm信息（如gc、锁），和计算机字长相等
    - oop指针：指向类型信息：开启指针压缩是4个字节，否则是8个
    - 实际数据成员：
    - 对齐区域：默认对齐到8个字节的倍数

## Collection和线程安全性
- 安全的：Vector、HashTable、Properties、ConcurrentXXX
- 不安全：ArrayList、LinkedList、HashSet、TreeSet、HashMap、TreeMap

## 语言特性
1. 序列化：
    - 一般写法：继承Serializable接口，提供一个serialVersionUID
    ```java
    public class MyEntity implements java.io.Serializable {
        // 该UID可以用于序列化内容的版本控制
        private static final long serialVersionUID=1L;
    }
    ```
## 反射
1. ```Method```：保存，并可以调用从Class中获取到的函数
```java
List<String> myList = new ArrayList<>();
Method method 
    = myList.getClass().getDeclaredMethod("add", Object.class);
method.invoke(myList, "ok");
// 通过Method绕过类型擦除限制
method.invoke(myList, 233);
```

## 泛型
1. Java的泛型和C++的是两种实现方式
    - Java做了类型擦除，准确的说是擦除到通配的上限或下限（如果有的话）。此时泛型代码只会生成一份，在编译的时候，由编译器添加上各种运行时转型。在```getClass()```中获得的都是擦除类型的。
    - C++对每份模板参数都做了代码展开
1. Java泛型机制关键点
    1. 类型检查是针对引用，而非原始内存
        ```java
        // 因为不对原始内存做检查，所以new后面不需要写泛型类型
        List<String> strList = new ArrayList<>();
        strList.add(1);     // 编译错误，编译器对strList会做类型检查
        
        // 如果引用不指定泛型类型，则无法在运行时约束类型
        List strList2 = new ArrayList<String>();
        strList2.add(1);    // ok
        strList2.add("1");  // ok

        // 引用传递时也会进行类型检查，未使用限定符的情况下，不允许有任何转型
        List<Object> objList = strList; // 编译错误
        ```
    2. JVM使用桥方法，完成泛型类的方法重写
        ```java
        public class Base<T> {
            public T add(T a, T b);
            public T get() {/* ... */}
        }

        public class Sub extends Base<String> {
            // 如果没有桥方法，Base的add实际都是Object
            // 因此会和这里的String不同，是方法重载而非重写
            @Override
            public String add(String a, String b) { return a + b;}

            // 而且即使只有返回值不同，jvm的桥接依然有效
            // 对于用户来说，返回值不能作为重载区分
            // 但是对于编译器和虚拟机来说是可以的，它是函数签名的一部分
            public String get() {/* ... */}
        }

        // 实际上的jvm字节码（示意）
        public class Sub extends Base<String> {
            public Object add(Object a, Object b) {
                this.add((String)a, (String)b);
            }
            
            public String add(String a, String b) {
                return a + b;
            }

        }
        ```
    3. 受制于类型擦除，泛型类型不能实例化。但可以通过反射的方式进行实例化。
        ```java
        T test = new T();   // 编译失败

        // 通过反射保留类型信息
        static <T> T newTclass (Class <T> clazz) 
            throws InstantiationException
                , IllegalAccessException {
            T obj = clazz.newInstance();
            return obj;
        }
        ```
    4. 受制于类型擦除，限定泛型类型的数组是不支持的。因为会导致自动转型异常（并不是技术上做不到，而是可能造成用户误用）。但是无限定泛型类型的数组是允许的，这是因为无限定情况下，必须由用户进行强制类型转换（如果你再写错，就不能怪Java了嗷）
        ```java
        // 如果允许限定泛型的数组的话，会出现以下情况
        List<String>[] lsa = new ArrayList<String>[10];
        Object o = lsa;
        Object[] oa = (Object[]) o;
        List<Integer> li = new ArrayList<Integer>();
        li.add(new Integer(3));
        oa[1] = li;                 // 能存
        String s = lsa[1].get(0);   // 取出来的时候转型是失败的

        // 正常的用法（其实也没必要）
        List<?>[] lsa = new ArrayList<?>[10]; // 限定符
        Object o = lsa;
        Object[] oa = (Object[]) o;
        List<Integer> li = new ArrayList<Integer>();
        li.add(new Integer(3));
        oa[1] = li; // Correct.
        Integer i = (Integer) lsa[1].get(0); // OK
        ```
    5. 如果一定要使用泛型数组，那么请使用反射```(T[])Array.newInstance(type, size)```
    6. 受制于类型擦除，泛型类中的静态方法和静态变量，中不能出现泛型类型参数。这是因为静态方法没有创建实例，编译器无法给出正确的转型。但注意，可以有静态泛型方法（泛型方法有自己的泛型类型参数）。
        ```java
        class Test<T> {
            public static T get() { /* */ }              // 编译失败
            public static T instance;                    // 编译失败
            public static <P> P pFunc(P a)  { /* */ }    // 编译成功
        }
        ```
    7. 受制于类型擦除，无法用泛型类去泛化异常，或者用泛型类去捕获异常。但仍然可以抛出泛型类型参数定义的变量。
        ```java
        class MyException<T> extends Exception {}       // 编译失败

        try { /* ... */ }
        catch (MyException<Integer> a) { /* ... */ }    // 联系第一行，泛型异常没有意义
        catch (MyException<String>  b) { /* ... */ }    // 擦除后是一样的
        ```
    8. 
2. 代码示例
```java
// --------------------定义方式--------------------

// 非泛型类中定义泛型方法
public class GenericsFunc {
    // 泛型方法的类型参数写在返回值前
    public static <T> void test(T t) { }
    public static <T> T test2(T t) { return t; }
}

// 泛型类中定义泛型方法
public class GenericsClass<T> {
    // 使用和泛型类相同的类型参数
    public void test(T t) {}

    // 使用和泛型类不同的类型参数
    public <P> P test2(P p) { return p; }
}

// 泛型接口定义和实现
public interface Info<T> { }
public class InfoImpl<T> implements Info<T> { /* ... */}

// 通配符使用
public class GenericsContainer {
    // 无限定通配符使用场景较少，它有只读限制
    public Object test(List<?> a) { return a.get(0); }

    // 也无法用于泛型类、泛型函数的类型参数定义
    // public <?> ? test2(? a); // 不能通过编译
    // class Test<?> {}         // 不能通过编译
}
// 设置上限，接受子类
public class GenericsContainer1 <? extends Base> {}
// 设置下限，接受超类
public class GenericsContainer2 <? super Sub> {}
// 同时多个上限限制
public class GenericsContainer3 <? extends BaseA & BaseB> {}

// --------------------使用方式--------------------
GenericsFunc.<Integer>test(1);  // 指定泛型参数
GenericsFunc.test(2);           // 自动推导
```
3. 问答:
    1. 如何保留完整的泛型类型？
    使用```Type```、```TypeToken```（在使用Json相关库进行序列化和反序列化时是很有必要的）


## 原生实用库
1. PropertyChangeSupport：观察者模式，用于监视一个Java Bean的属性修改，可以包装并发送属性修改事件
    > 无法监听被观察类型的实例，在构造函数中发生的修改

## 内部类
内部类一共有4种，本节内容参考自[菜鸟教程](https://www.runoob.com/w3cnote/java-inner-class-intro.html)
1. 成员内部类。和普通类不同的主要是其对外的可见性，普通类有public和本包可见两种可见性，而成员内部类和成员一样，可以拥有public/protected/private三种。内部类必须依赖于外部类才能存活，同时，外部类的一切成员都对内部类可见。
```java
class Circle {
    private Draw draw = null;
    double radius = 0;
     
    public Circle(double radius) {
        this.radius = radius;
    }
     
    class Draw {     //内部类
        public void drawSahpe() {
            System.out.println("drawshape");
        }
    }
}
```
2. 局部内部类，和成员内部类的区别是，这种类类型是在函数内定义的，所以称之为局部，也不能有public/protected/private和static修饰。
3. 匿名内部类，在实现回调时非常实用，使用IDEA等都会自动补全这种情况。匿名内部类不能有构造器（从语法上来看是匿名，当然也就没法有构造函数了），写法形如
```java
Button button = new Button();
button.setOnClickListener(new OnClickListener(){
    @Override
    public void onClick(View v) {
        // ...
    }
});
```
4. 静态内部类，也是成员内部类，但是是static的，和成员内部类必须依赖于外部类才能存在的情况不同，静态内部类可以单独存在，但是访问外部类受限制了，只能放为其静态成员。

## 标准
目前比较流行的版本主要有3个：JDK8、JDK11、JDK17。这中间如果有一些特性也比较优秀，会单独标记。以后的新项目都应该考虑使用JDK17以上了。
### Java8
2014年发布。生态最庞大的。很多语法可能已经很熟悉了，在这里快速列一下
```java
// ================== 更新1 ==================
// 默认方法，可以在接口中添加默认实现
interface MyInterface {
    default void DoSth() {
        // ...
    }
}

// ================== 更新2 ==================
// lambda表达式，支持参数类型推断
// ([parameters]) -> expression
// ([parameters]) => {statements;}

// ================== 更新3 ==================
// stream，串行流，一个非常重要的内容，和lambda表达式结合起来，像函数式中的map/reduce/filter一样
// 分为三种操作，intermediate、short-curcuit、terminate。中间、短路、终结。
List<String> list = new ArrayList<>();
// 注意本例演示，DoubleStream和Stream<Double>是完全不一样的，后者是装箱类型，可用操作更多
List<Double> result = list.stream()
    .filter(s -> s.matches("正则表达式"))
    .mapToDouble(Double::parseDouble)
    .limit(3)              // short-curcuit
    .map(d -> d+=0.1)      // 所有的map、filter都是中间操作
    .boxed()               // 特殊情况需要装箱
    .sorted(Comparator.comparingDouble(Double::doubleValue).reversed())
    .limit(2)              // 可以排序、limit
    .collect(Collectors.toList);        // collect就是一个reduce类型的收集工作
result.forEach(System.out::println);

// toMap示例，toMap相对复杂
Map<Double,Integer> reMap =  
    result.stream().collect(Collectors.toMap(Double::doubleValue, Double::intValue));
reMap.forEach((aDouble, integer) -> System.out.println(aDouble+ " -> " + integer));

// parallelStream则是并行流，适合对顺序没有要求的处理场景

// ================== 更新4 ==================
// 新的日期类LocalDate、LocalTime、LocalDateTime
// 新的时间戳和时间间隔Instant、Period、Duration

// ================== 更新5 ==================
// 方法引用
class YourClass {
    // ...
}
class YourFactory {
    private Supplier<Object> sup;
    public YourFactory(final Supplier<Object> supplier) {
        sup=supplier;
    }

    public Object produce() {
        return sup.get();
    }
}
// 可以用以下方式引用构造器、方法
YourFactory fac = new Factory(YourClass::new);  // 构造器引用的类型是Supplier<YourClass>
YourClass yc = (YourClass) fac.produce();

// ================== 更新6 ==================
// 函数式接口
@FunctionalInterface
interface YourFuncInterface {
    // ...
}

// ================== 更新7 ==================
// Optional类
// Optional.ofNullable - 允许传递为 null 参数
Optional<Integer> a = Optional.ofNullable(value1);
// orElse提供如果null，则使用后面的值的方法
Integer value = a.orElse(new Integer(0));

// Optional.of - 如果传递的参数是 null，抛出异常 NullPointerException
Optional<Integer> b = Optional.of(value2);
System.out.println(java8Tester.sum(a,b));

// ================== 更新8 ==================
// 自带Base64
static class Base64.Decoder;
static class Base64.Encoder;
```

### Java9/10
9的更新内容有
- 支持模块化，支持多版本Jar
- 接口可以有私有方法
- 实验性的内置HTTP/2客户端，```HttpClient/HttpRequest/HttpResponse```
- JShell，无需创建项目，而允许执行Java片段
- 更好的日志
- 进程API更新，```ProcessHandler```，能获得更多进程信息
- Collection的API改进
- Stream的API改进
- @Deprecated改进，Java文档（javadoc）改进
- GC改进

10的更新内容
- var关键字，局部变量类型推断（类似C++的auto）
- GC改进，GC的接口被统一，方便对GC进行替换
- 实验性的JIT

### Java11
2018年发布，生态稍微差。主要原因是收费。一些比较重要的特性如下。
- 正式引入```java.net.http```，内置HTTP支持
- 字符串API改进，提供更多的实用函数，如```repeat/isBlank/strip/lines/String[]::new```
- TLS支持
- GC改进
- 飞行记录器（运行时信息收集）

### Java12~16
Java版本发布速度非常快，这里统一记录一些比较重要的，不做区分了
- 更多字符串改进
- switch表达式改进，可以用作语句表达式，出现在等号右侧，分情况赋值更方便
- 多行文本块，跨行文本更方便，也不需要再转义，用连续3个引号```"""```开启。
- SocketAPI重构，```NioSocketImpl```
- NullPointerExceptions改善
- instanceOf改进，```if(xx instanceOf String s)```
- record关键字，对只需要getter/setter类型，可以直接使用

### Java17
2021年发布，据Oracle的说法至少在发布的三年内免费。所以生态好了一些。它主要变更是
- 密封类，```sealed```，可以更细粒度的控制是否允许对类型/接口进行继承和重写
- 一些工具的改进（但有很多都是提案）：```RandomGenerator```，

## 参考
- [Java全栈知识体系：泛型机制详解](https://pdai.tech/md/java/basic/java-basic-x-generic.html)
- [Java中JDK8、JDK11、JDK17，该怎么选择？](https://cloud.tencent.com/developer/article/1977236)
- [Java8到Java17都更新了哪些内容？！收藏了](https://zhuanlan.zhihu.com/p/480293185)