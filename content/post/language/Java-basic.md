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


## 参考
[Java全栈知识体系：泛型机制详解](https://pdai.tech/md/java/basic/java-basic-x-generic.html)