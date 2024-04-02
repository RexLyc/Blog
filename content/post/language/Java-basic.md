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
## Jar包结构
参考[一文读懂jar包的小秘密](http://www.flydean.com/java-jar-in-detail/)
一般来说有三个大类
1. META-INF目录：从名字看也知道，是记录了各类元信息的目录
2. class：存放字节码的目录
3. BOOT-INFO目录：Spring Boot应用程序从Boot-INF文件夹加载。
3. 资源文件目录


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

1. 基于反射、接口的动态代理：动态代理是Java中非常重要的特性之一。动态代理能支持对所有实现了同一个接口的类型，完成加载，代理对象的创建。参考[为什么JDK动态代理只能代理接口，不能直接代理类？CGlib为什么可以代理类？](https://blog.csdn.net/lx1315998513/article/details/120641124)。下面摘抄这种情况的一个用例。
    ```java
    // 接口
    public interface UserService {
        String query();
    }

    // 实现类
    public class UserServiceImpl implements UserService{
        
        @Override
        public String query() {
            System.out.println("query");
            return null;
        }
    }

    // 动态代理
    public class UserServiceInvocationHandler implements InvocationHandler {
        private Object target;

        public UserServiceInvocationHandler(Object target) {
            this.target = target;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("invocation handler");
            // 通过反射调用目标对象的方法
            return method.invoke(target, args);
        }
    }

    public class MainApplication {

        public static void main(String[] args) {
            // 指明一个类加载器，要操作class文件，怎么少得了类加载器呢
            ClassLoader classLoader = MainApplication.class.getClassLoader();
            // 为代理对象指定要是实现哪些接口，这里我们要为UserServiceImpl这个目标对象创建动态代理，所以需要为代理对象指定实现UserService接口
            Class[] classes = new Class[]{UserService.class};
            // 初始化一个InvocationHandler，并初始化InvocationHandler中的目标对象
            InvocationHandler invocationHandler = new UserServiceInvocationHandler(new UserServiceImpl());
            // 创建动态代理
            UserService userService = (UserService) Proxy.newProxyInstance(classLoader, classes, invocationHandler);
            // 执行代理对象的方法，通过观察控制台的结果，判断我们是否对目标对象(UserServiceImpl)的方法进行了增强
            userService.query();
        }
    }
    ```
    上文中的```userService```就是代理对象，该对象的动态类型实际上并不是UserService，而是实现了UserService接口，同时也继承了Proxy类型的一个类型的对象。```class Proxy0 extends Proxy implements UserService```。这一步是运行时生成的，所以称为动态代理。正是因为Java只允许单继承，所以这里的被代理类型，必须是一个实现了某个接口的类型。

## 类加载
和其他语言不通，Java从基础上就提供了对类型的运行期加载机制。这个机制就像C/C++语言中，动态链接一样的重要且常见。参考[JVM 基础 - Java 类加载机制](https://pdai.tech/md/java/jvm/java-jvm-classload.html)、[JDK8以后废弃扩展类加载器的原因](https://blog.csdn.net/qq_38003038/article/details/122733985)、[搞定JVM面试之JVM 类加载器](https://snailclimb.gitee.io/2019/08/25/java/jvm/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8/)

1. 类的加载: 查找并加载类的二进制数据。
2. 验证: 确保被加载的类的正确性。文件格式、魔数、版本、字节码语义分析、字节码验证、引用验证。验证可以关闭。
3. 准备: 为类的静态变量分配内存，并将其初始化为默认值
4. 解析: 把类中的符号引用转换为直接引用。类似于重定位。
5. 初始化：为类的静态变量赋予正确的初始值，执行初始化块。这一过程会推迟到类被真正主动使用。

目前加载器有四个层级：
1. 系统级Bootstrap，负责调用rt.jar，由于是C++编写，因此无法在Java内打印得到该加载器
2. 扩展（<=1.8）/ 平台级Platform（>1.8）：负责加载各种jar包，扩展java能力
3. 应用程序App：负责加载```$CLASSPATH```下的所有jar
4. 用户自定义

加载的入口有三种：启动时JVM加载、```Class.forName()```、```ClassLoader.loadClass()```。

加载机制有四种：
1. 全盘负责，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入
2. 父类委托，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类
3. 缓存机制，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效
4. 双亲委派机制, 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。
    > 类似C/C++的动态库搜索。使用双亲委派机制，会优先查找更高级别的jar包，最后才是当前应用的```$CLASSPATH```。避免出现重复字节码。也保证了 Java 的核心 API 不被篡改。

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

## 并发
### 锁
参考[不可不说的Java锁事](https://tech.meituan.com/2018/11/15/java-lock.html)。先从Java提供的锁机制来看：主要有三种synchronized、Lock、AtomicXXX。

syncronized是Java最历史悠久的同步机制，在Java1.5之前，没有做优化，其性能较低（重量锁），在1.6之后，它上锁时有一个尝试的顺序：无锁、偏向锁（已被废弃）、轻量级锁、重量级锁。当一个线程企图访问临界区时，会从这个顺序逐渐升级。其中轻量级锁是指一个自旋锁，线程会尝试等待一小段时间，如果无法获得锁，就转换为重量级锁，并阻塞，等待其他线程释放锁。

Lock是Java提供的一个类型，位于java.util.concurrent下，它解决了synchronized的一些缺陷：可以得知是否上锁成功，在线程选择睡眠时可以释放锁（syn只能在异常和结束时释放），sync也不可中断（只能死等其他线程释放锁），sync也是非公平的。在Lock下，Java提供ReentrantLock，也是可重入锁。另外还有ReentrantReadWriteLock，它提供了读写分离的两种锁，是共享锁（前面的都是独占锁），提高了多读的效率。

> 注意synchronized也是可重入（可递归锁）的，对于同一个对象，如果当前已经获得了锁，尝试再上锁是可以成功的。可以避免一些死锁的情况。

> 非公平锁的吞吐率其实要高于公平锁。公平锁需要维护队列，在发现已有等待的线程时会去排队，保证按顺序唤醒，而非公平锁有机会减少唤起线程的开销，它在申请时不去排队而会在流程中多次尝试获取锁。

AtomicXXXX则是Java基于CAS指令提供的原子类。提供了无锁机制（在一定程度上可以代替有锁，但不绝对）。原子类有自己的问题，比如ABA问题、只支持单一内置变量、内存序等。

Semaphore，AQS提供的另一种同步类型。Semaphore是共享锁。

Java中的大部分同步类（Lock、Semaphore、ReentrantLock等）都是基于AbstractQueuedSynchronizer（简称为AQS）实现的。AQS支持共享和独占，也支持公平和非公平。参考[AQS原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)、[AQS原理](https://javaguide.cn/java/concurrent/aqs.html)。它的核心是Node数据结构（存储线程、请求锁方式等信息），和一个等待队列（虚拟）。队列中的阻塞等待和唤醒过程中的锁分配，是通过 CLH（Craig，Landin，and Hagersten） 队列和锁实现的。

所谓的虚拟队列，是因为该队列并无实体，每一个Node两两用prev/next相连就构成了，Node保存线程的引用，以及节点在队列中的状态（等待锁、占用锁），在外部用一个volatile标记的int值state来保管同步状态（用CAS写）。各个Node都可以尝试去修改state。

> 注意volatile能保证可见性和禁止指令重排，但是不能保证原子性。线程在使用volatile变量时，**只是每次都会读取**到线程本地再使用，但是如果修改，则显然写回时不能保证。

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
- instanceof模式匹配改进，支持变量定义```if(xx instanceOf String s)```
- record关键字，对只需要getter/setter类型，可以直接使用

### Java17
2021年发布，据Oracle的说法至少在发布的三年内免费。所以生态好了一些。它主要变更是
- 密封类，```sealed```，可以更细粒度的控制是否允许对类型/接口进行继承和重写
- 一些工具的改进（但有很多都是提案）：```RandomGenerator```，

### Java18-20
大部分都是预览和孵化功能，有些会在Java21中正式发出
- 设定UTF-8为默认字符集
- 内置简单Web服务器
- 反射改进

### Java21
2023年9月发布，一次非常重大的改进。最主要内容是
- 虚拟线程
- ZGC
- instanceof模式匹配对record的支持
- 增强swtich，支持类型模式匹配，例如```case Integer i -> { ;}```
此外还有一些预览功能：外部方法和内存API、instanceof对未命名变量```_```的支持、字符串模板、main方法入口查询改进


## 参考
- [Java全栈知识体系：泛型机制详解](https://pdai.tech/md/java/basic/java-basic-x-generic.html)
- [Java中JDK8、JDK11、JDK17，该怎么选择？](https://cloud.tencent.com/developer/article/1977236)
- [Java8到Java17都更新了哪些内容？！收藏了](https://zhuanlan.zhihu.com/p/480293185)