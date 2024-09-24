---
title: "设计模式-开坑篇"
date: 2021-08-04T10:56:42+08:00
categories:
- 计算机科学与技术
- 设计模式
tags:
- 系列开坑
- 设计模式
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/design-pattern.svg
---
软件行业发展到现在，大部分需求场景下，优秀的开发模式已有定论。本系列结合场景和代码，总结常用设计模式。
<!--more-->.
## 为什么要看设计模式
- 虽然设计模式经常被人吐槽，但实际上学一点设计模式本质上只是想借鉴前人经验。
- 不仅是代码复用，更要体会经验复用，分解问题的方式方法。
## 如何使用设计模式
1. 列出你的主要需求
1. 列出你未来可能支持的新需求
1. 分离会变动的部分，和不会变动的部分
1. 平衡开发速度和运行速度，寻找合适的设计模式
## 七大原则
1. 开闭原则：允许扩展，禁止修改。
2. 单一职责原则：一个类只做一件事情。
3. 依赖倒转原则：高层模块不应依赖底层模块。即开发应当面向抽象（接口），而不针对具体实现。
4. 迪米特（最小知识原则）：高内聚，低耦合。一个对象应当尽量减少自己的依赖类型、减少和自己有耦合关系的类型。提升模块之间的独立程度。
5. 里氏代换原则：任何可以使用父类的地方，均应可以使用子类。
6. 接口隔离原则：接口应当尽量分化、细化，不应当强迫用户实现一些不会使用到的接口。互相依赖应当建立在最小的接口集合上。
7. 组合/聚合复用原则：尽量使用组合和聚合，少使用继承来完成代码复用。
## 常用设计模式速览
### 创建型
> 如何更优雅的创建对象
- 单例模式（Singleton）：一个类型在程序生命周期内只有一个实例
    ```java
    // 一个比较中庸的写法
    public class Singleton {
        // JVM在加载时创建，线程安全
        private static Singleton instance = new Singleton();

        private Singleton() {}

        // 即使不调用getInstance，此时instance也已经构造出来了
        public static Singleton getInstance() {
            return instance;
        }
    }
    ```
    类似的，cpp中也可以用静态变量来做
    ```cpp
    public class Singleton {
        private:
            Singleton() {}
            Singleton(const Singleton&) = delete;
        public:
            // 线程安全
            Singleton* getSingleton() {
                static Singleton instance;
                return &instance;
            }
    }
    ```
- 工厂方法（Factory）：提供创建实例的接口，由具体的子类工厂决定具体实例化的类型。其实这样做还有一个好处，就是调用者不需要引入很多具体类型（没有直接依赖关系）。
    ```java
    public interface BeanFactory {
        Bean createBean(String type);
    }

    public class FactoryA implements BeanFactory {
        Bean createBean(String type) {
            // ...
        }
    }

    public abstract class  Bean {
    }

    public class A extends Bean {
        //...
    }

    public class B extends Bean {
        //...
    }

    // 使用时以工厂创建
    BeanFactory f = new FactoryA();
    Bean bean = f.createBean("you type info");
    ```
- 抽象工厂（Abstract Factory）：提供一个接口用于创建相关对象或对象家族，而不需要明确指明创建的具体类型。抽象工厂模式和工厂模式经常一同出现。
    ```java
    public interface AbstractFactory {
        BeanA createBeanA();
        BeanB createBeanB();
    }

    public class AbstractFactoryA implements AbstractFactory{        
        BeanA createBeanA() {
            // ...
        }
        BeanB createBeanB() {
            // ...
        }
    }

    //同理
    public class AbstractFactoryB implements AbstractFactory {}

    public abstract class Bean {
        BeanA componentA;
        BeanB componentB;
    }

    public class A extends Bean {
        AbstractFactory componentFactory;
        public A(AbstractFactory factory) {
            componentFactory = factory;
        }

        public prepare() {
            componentA = componentFactory.createBeanA();
            componentB = componentFactory.createBeanB();
        }
    }

    // class A 仍有自己的工厂(见工厂模式)
    public interface BeanFactory;
    // ...
    ```
    > 相比于工厂模式，抽象工厂实际上提供了更高的抽象，即对对象的成员的具体构成，可以进一步使用工厂去控制创建。
- 生成器（Builder）：将复杂对象的创建过程封装起来，允许对象通过多个步骤来创建，并且可以改变创建过程。
    ```java
    public interface AbstractBuilder {
        void initialize();
        void addA(String a);
        void addB(String b);
        void addC(String c);
        // 获取构建完成的对象的接口
        String getBuildResult();
    }

    public class MyStringBuilder {
        // 实现略
    }
    ```
- 原型（Prototype）：通过克隆的方式，快速创建重复的对象，保证性能。该模式应当支持根据调用者的输入条件，选择要复制的实例。
    ```java
    public interface BigBean {}

    public class BigBeanRegistry {
        Map<String,BigBean> BigBeanYarn;

        public BigBean getBigBean(String param) {
            return BigBeanYarn.get(param).clone();
        }
    }
    ```
### 结构型
> 如何更优雅的扩展功能、精简代码
- 组合模式（Composite）：部分整体模式，对象内部有同类型对象的集合，例如展开成树状结构的表达式。这样就能以一致的方式处理单个对象，或者对象组合（树）。
    ```java
    public interface BaseItem {
        String getName();
        double getPrice();
    }

    public class ItemA extends BaseItem {
        public String getName() {
            // ...
        }
        public double getPrice() {
            // ...
        }
    }

    public class ItemMenu extends BaseItem {
        ArrayList items = new ArrayList();
        public void add(BaseItem item) {
            items.add(item);
        }
        public String getName() {
            // 遍历所有内容
            // ...
        }
        public String getPrice() {
            // 遍历所有内容
            // ...
        }
    }
    ```
- 外观（Facade）：优化接口，降低外部使用模块的复杂度。可以用于封装多个类。项目中期优化使用。
    ```java
    public interface Light {}
    public interface Sofa {}
    public interface TV {}
    // ... 各自的具体实现
    public class SunLight implements Light {};
    public class SoftSofa implements Sofa {};
    public class CRT implements TV {};
    // 外观模式封装
    public class HomeFacade {
        Light light;
        Sofa sofa;
        TV tv;
        
        public HomeFacade(Light l,Sofa s,TV t) {
            light = l; sofa = s; tv = t;
        }
        
        public setWatchTVMode() {
            // ... 一套操作
        }

        public setSleepMode() {
            // ... 一套操作
        }
    }
    // 其他封装情况
    public class BarFacade extends HomeFacade {}
    ```
- 代理（Proxy）：增加中间层，为其他对象提供接口以提供对当前被代理对象的访问。一般针对一个类。根据代理的用途，有不同的具体类型，如：远程代理、虚拟代理、保护代理、防火墙代理、缓存代理、智能引用代理、同步代理、复杂隐藏代理、写入时复制代理。Java中原生支持远程代理（RMI）。
    ```java
    // RMI：调用者 -> Stub -> 网络 -> Skeleton -> 执行者
    // 远程接口，调用侧
    public interface MyRemote extends Remote {
        public String helloWorld() throws RemoteException;
    }
    // main中
    MyRemote service = (MyRemote) Naming.lookup("rmi://127.0.0.1/RemoteHello");

    // 远程接口实现，执行侧
    public class MyRemoteImpl extends UnicastRemoteObject implements MyRemote {
        public String helloWorld() {
            return "hello world";
        }

        public MyRemoteImpl() throws RemoteException {

        }

        public static void main(String[] args) {
            try {
                MyRemote service = new MyRemoteImpl();
                Naming.rebind("RemoteHello", service);
            } catch(Exceptino e) {
                e.printStackTrace();
            }
        }
    }
    ```
    再看一段更常见的例子
    ```java
    public interface Action {
        void doSth();
    }

    public class RealActionA implements Action {
        public void doSth() {
            // ...
        }
    }

    public class ActionProxy implements Action {
        Action realObject;

        public void doSth() {
            // ...
            realObject.doSth();
            // ...
        }
    }
    ```
    Java在包java.lang.reflect中提供了动态代理技术，在实际编码中更为常用。下面展示了使用动态代理，对于访问实例方法进行权限控制。
    ```java
    public interface StudentBean {
        String getName();
        void setName(String name);
        double getScore();
        void setScore(double score);
    }

    public class StudentBeanImpl implements StudentBean {
        String name;
        double score;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public double getScore() {
            return score;
        }

        public void setScore(double score) {
            this.score = score;
        }
    }

    public class OwnerInvocationHandler implements InvocationHandler {
        StudentBean person;

        public OwnerInvocationHandler(StudentBean person) {
            this.person = person;
        }

        public Object invoke(Object proxy, Method method, Object[] args) 
                throws IllegalAccessException {
            try {
                if(method.getName().equals("setScore")){
                    // 分数不能由自己设置
                    throw new IllegalAccessException();
                } else {
                    return method.invoke(person, args);
                }
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }
            return null;
        }
    }

    public class TeacherInvocationHandler implements InvocationHandler {
        // 其他都类似，但只允许设置学生的分数
        // 略
    }

    public class Util {
        // 创建代理后的实例
        public static StudentBean getOwnerProxy(StudentBean student) {
            return (StudentBean) Proxy.newProxyInstance(
                student.getClass().getClassLoader(),
                student.getClass().getInterfafces(),
                new StudentInvocationHandler(student));
        }

        public static StudentBean getTeacherProxy(StudentBean student) {
            return (StudentBean) Proxy.newProxyInstance(
                student.getClass().getClassLoader(),
                student.getClass().getInterfafces(),
                new TeacherInvocationHandler(student));
        }
    }
    ```
- 适配器（Adaptor）：为两个已有的不兼容模块间的接口建立连接，使得能够共同工作。非必要不应使用适配器模式，而应当进行重构。项目后期兼容开发。
    ```java
    public interface Duck {
        void fly();
    }

    public interface Mouse {
        void run();
    }

    public class Jerry {
        void run() {
            // ...
        }
    }

    // 对象适配器（继承 + 组合）
    public interface MouseDuck implements Duck {
        Mouse m;

        public MouseDuck(Mouse m) {
            this.m = m;
        }

        void fly() {
            m.run();
        }
    }
    ```
    由于语言限制，java无法实现类适配器（多继承）。如果在C++中，还能做类适配器。
- 装饰器（Decorator）：允许向一个现有的对象的某个接口添加新的扩展功能，而不改变已有结构。由一个抽象类实现一个接口，可以一直扩展下去。在java中，java.io的InputStream类型体系就是装饰器结构。
    ```java
    public abstract class BaseContent {
        String description = "BaseContent";

        public String getDescription() {
            return description;
        }

        public Data doSth();
    }

    // 实现类
    public class ContentA extends BaseContent {
        public ContentA() {
            description="ContentA";
        }

        public Data doSth() {
            // ...
        }
    }

    // 装饰者类
    public class ContentDecorator extends BaseContent {
        public abstract String getDescription();
    }

    // 具体装饰者
    public class ContentDecoratorA extends ContentDecorator {
        // 可以和不同种实现类进行组合，甚至也可以和具体装饰者进行组合
        BaseContent baseContent;

        public ContentDecoratorA(BaseContent content){
            baseContent = content;
        }

        public String getDescription() {
            return baseContent.getDescrption() + "DecoratorA";
        }

        public Data doSth() {
            return baseContent.doSth() /* + ... ;*/;
        }
    }

    // 使用示例
    // 可以嵌套多层装饰
    // 由此就做到了将不同的装饰分开，但可以根据需要组合完成功能描述
    BaseContent a = new ContentA();
    BaseContent a2 = new ContentDecoratorA(a);
    BaseContent a3 = new ContentDecoratorA(a2);
    ```
- 桥接器（Bridge）：把抽象和实现解耦。将多变的部分单独抽象为接口，在实体类中进行使用。由一个抽象类持有一个接口。
    ```java
    // 抽象出来的多变部分(桥接的左半部分)
    public abstract class RemoteContrl {
        // Has-A
        TV implementor;
        public void on();
        public void off();
        public void setChannel(String channel) {
            implementor.tuneChannel(channel);
        }
    }

    public class ConcreteRemote extends RemoteControl {
        // 实现略
    }

    // 抽象出来的多变部分(桥接的右半部分)
    public abstract class TV {
        void on();
        void off();
        void tuneChannel(String channel);
    }

    public class HaierTV implements TV {
        // 实现略
    }
    ```
- 享元（Flyweight）：也叫做蝇量模式，尝试重用现有的同类对象，如果无法匹配则创建新的对象。可以理解为对对象实例的复用。
    ```java
    // 传统方式，每个树单独有一个完整状态的实例
    public class TreeFull {
        Coord position;
        // 大量属性
        public void display();
    }

    // 享元模式，只保留通用的部分
    public class TreeFlyweight {
        // 无属性，或少量属性
        public display(x,y);
    }

    public class TreeManager {
        // 复用
        TreeFlyweight treeMultiplexing;

        public display() {
            // 一个实例用于多次绘制
            treeMultiplexing.display(1,2);
            treeMultiplexing.display(10,-10);
            // ...
        }
    }

    ```
- 过滤器模式（Filter）：一个接口继承体系内，用不同实现过滤同一组对象。过滤操作可以组合成复杂的语义。
    ```java
    public class Person {
        int age;
        String name;
        String gender;
        // 各种get、set方法，略
    }

    public interface PersonFilter {
        List<Person> filter(List<Person> input);
    }

    public class MaleFilter implements PersonFilter {
        // 编写filter过滤男性，略
    }

    public class MinorsFilter implements PersonFilter {
        // 编写filter过滤未成年人，略
    }

    public class AndFilter implements PersonFilter {
        PersonFilter a,b;
        public AndFilter(PersonFilter a, PersonFilter b) {
            this.a = a;
            this.b = b;
        }

        // 且
        public List<Person> filter(List<Person> input) {
            b.filter(a.filter(input));
        }
    }
    ```
### 行为型
> 如何更优雅的完成一些操作
- 迭代器（Iterator）：提供一个获取迭代器的接口，顺序访问集合对象元素。
    ```java
    public interface Iterator {
        boolean hasNext();
        Object next();
    }

    public class MyIterator implements Iterator {
        MyItem[] items;
        int position = 0;
        
        public MyIterator(MyItem[] items) {
            this.items = items;
        }

        public Object next() {
            return items[position++];
        }
        
        public boolean hasNext() {
            return position != items.lenth;
        }
    }
    ```
- 观察者（Observer）：对存在一对多依赖关系的情况，保管相关依赖对象的集合，变更属性则通知依赖它的对象。java对观察者模式有内置支持（Observer & Observable），而且观察者也分为推送（push）和拉取（pull）两种模式，下文仅以自定义代码展示推送方式的基本原理。
    ```java
    public interface Subject {
        void install(Observer o);
        void uninstall(Observer o);
        void notifyObservers();
    }

    public interface Observer {
        void update(String content);
    }

    public interface Display {
        public void display();
    }

    // 可以继续扩展被观察者
    public class SubjectImplA implements Subject {
        private ArrayList observers;
        private String data;
        
        public void install(Observer o) {
            // ...
        }
        public void uninstall(Observer o) {
            // ...
        }
        public void notifyObservers() {
            // ...
            for(Observer o: observers) {
                o.update(data);
            }
        }
    }

    // 观察者模式也能继续扩展观察者
    public class DisplayImplA implements Display, Observer {
        public void update(String data) {
            display()
        }
        public void display() {
            // ...
        }
    }
    ```
- 模板方法（Template）：允许重写部分方法，但这些方法的调用将会以抽象类中定义的逻辑进行。
    ```java
    public interface Drink {
        default void prepare() {
            boilWater();
            brew(); // 冲泡
            pourInCup();
            addCondiments(); // 加配料
            hook();
            if(addTwiceCondiments()){
                addCondiments();
            }
        }

        default void boilWater() {
            // ...
        }

        default void pourInCup() {
            // ...
        }

        // 也可以留一个默认啥也不做的钩子
        default void hook(/*... Args*/) {}

        // 甚至可以留一个可以控制流程的函数
        default bool addTwiceCondiments() {
            return false
        }

        void braw();
        void addCondiments();
    }

    public class Tea implements Drink {
        public void brew() {
            // ...
        }

        public void addCondiments() {
            // ...
        }
    }

    public class Coffe implements Drink {
        public void brew() {
            // ...
        }

        public void addCondiments() {
            // ...
        }
    }
    ```
- 命令模式（Command）：数据驱动，将命令包裹在对象中进行传递，调用对象负责寻找合适的处理对象，将命令传递过去。调用者→命令→接收者，即调用者实际保存若干命令，命令中封装对接收者的实际调用，而调用者不需要关系命令究竟是如何实现的。命令排队，且可撤销。
    ```java
    public interface Command {
        void execute();
        void undo();
    }

    // 一种具体命令的实现
    public class LightOnCommand implements Command {
        Light light;
        public LightOnCommand(Light light) {
            this.light=light;
        }

        public void execute() {
            this.light.on();
        }

        public void undo() {
            this.light.off();
        }
    }

    // 可以组合出批处理命令
    public class BatchCommand implements Command {
        Command[] commands;
        
        public BatchCommand(commands) {
            this.commands = commands;
        }

        public void execute() {
            for(Command c: commands) {
                c.execute();
            }
        }

        public void undo() {
            for(Command c: commands) {
                c.undo();
            }
        }
    }

    // 接收者
    public class RemoteControl {
        List<Command> commands;
        // 仅支持最后一个命令的撤销
        Command lastCommands;

        public RemoteControl() {
            commands = new ArrayList<Command>();
        }

        public void setCommand(Command command) {
            commands.add(command);
        }

        public void onButtonPressed(int index) {
            commands[index].execute();
            lastCommands = commands[index];
        }

        public void onUndoPressed() {
            lastCommands.undo();
        }
    }
    ```
- 状态模式（State）：对象内部保存一个状态对象（每种状态需要实现不同状态下的行为，并且行为会变更依赖对象），根据不同状态变更行为。用来避免过多条件语句。表现为类的行为是受状态控制而变化。可以优雅的实现状态机。
    ```java
    // 假定必须按顺序先做A，再做B
    public interface State {
        void actionA();
        void actionB();
    }

    public class Controller {
        State init;
        State afterA;
        State afterB;
        State currentState;
        public Controller() {
            currentState = init = new StateInit();
            afterA = new StateA();
            afterB = new StateB();
        }
        public void actionA() {
            currentState.actionA();
        }
        public void actionB() {
            currentState.actionB();
        }
        public void setState(State s) {
            currentState = s;
        }
        public State getStateA() {
            return afterA;
        }
        public State getStateB() {
            return afterB;
        }
    }

    public class StateInit implements State {
        Controller controller;
        public void actionA() {
            controller.setState(controller.getStateA());
        }
        public void actionB() {
            // reject
        }
    }

    public class StateB implements State {
        Controller controller;
        public void actionA() {
            // reject
        }
        public void actionB() {
            controller.setState(controller.getStateB());
        }
    }
    ```
- 策略模式（Strategy）：创建统一接口下表示各种策略的对象，使之可被替换。表现上是一个类中接口的行为可以动态变化、替换。
    ```java
    public interface FlyStrategy {
        void fly(){

        }
    }

    public class DuckFly implements FlyStrategy {
        public void fly() { 
            // ...
        }
    }

    public class FakeFly implements FlyStrategy {
        public void fly(){
            // ...
        }
    }

    public interface Duck {
        FlyStrategy flyStrategy;
        
        default void setFlyStrategy(FlyStrategy strategy){
            flyStrategy=strategy;
        }

        default void performFly() {
            flyStrategy.fly();
        }
    }

    public class FlyableDuck implements Duck {
        // ...
    }

    public class FakeDuck implements Duck {
        // ...
    }

    ```
- 职责链（Chain ofResponsibility）：也叫责任链，将请求的发送者和接收者解耦，多个对象都有可能接收并处理同一个请求，直到有对象标记处理完成为止。
    ```java
    public abstract class Handler {
        Handler successor;
        public void handleRequest();
    }

    public class HandlerA extends Handler {
        public HandlerA(Handler successor) {
            this.successor = successor;
        }

        public handleRequest() {
            // ...
            this.successor.handleRequest();
        }
    }
    ```
- 中介者（Mediator）：提供一个中介类处理不同类之间的通信，降低通信的复杂性。比如闹钟、日历、咖啡、淋雨具有复杂关系的操作，可以通过一个中介者来解耦通信和操作关系。
    ```java
    public interface Event;
    public class AlarmEvent implements Event {}
    public class CalendarEvent implements Event {}
    public class CoffeeEvent implements Event {}
    public class ShowerEvent implements Event {}
    
    public class Mediator {
        public void onEvent(Event e) {
            if(e instanceof AlarmEvent) {
                // ...
            } else if(e isntance of CalenderEvent) {
                // ...
            } else {
                // ...
            }
        }
    }

    public class Alarm {
        Mediator m;
        public void onTimeout() {
            m.onEvent(new AlarmEvent());
        }
    }
    ```
- 访问者（Visitor）：针对一些易变且和对象类型无关的操作，在被访问的类里放一个对外接收访问者的接口，将数据结构和数据操作分离。
    ```java
    public abstract class Person {
        // 在对象内部，提供的接受访问者的接口
        public abstract void accept(Action action);
    }

    public class Man extends Person {
        public void accept(Action action) {
            action.getManResult(this);
        }
    }

    public class Woman extends Person {
        public void accept(Action action) {
            action.getManResult(this);
        }
    }

    // 根据需要可以给出针对不同对象的实现
    // 对扩展开放，对修改关闭
    public abstract class Action {
        public abstract void getManResult(Man man);

        public abstract void getManResult(Woman woman);
    }

    public class Good extends Action {
        public void getManResult(Man man) {
            // ...
        }

        public void getManResult(Woman woman) {

        }
    }

    public class Bad extends Action {
        public void getManResult(Man man) {
            // ...
        }

        public void getManResult(Woman woman) {
            // ...
        }
    }
    ```
- 解释器（Interpreter）：评估语言的语法或表达式，用于实现简单文法。（终结符表达式、非终结符表达式）
    ```java
    public interface Expression {
        void interpret(String context);
    }

    // 连续语句
    public class Sequence implements Expression {
        Expression exp1;
        Expression exp2;
        void interpret(String context) {
            // ...
        }
    }

    public class Command implements Expression {
        void interpret(String context) {
            // ...
        }
    }

    // 循环语句(由循环条件变量和表达式构成)
    public class Repetition implements Expression {
        Variable var;
        Expression exp;
        void interpret(String context) {
            // ...
        }
    }

    // 变量
    public class Variable implements Expression {
        void interpret(String context);
    }
    ```
- 备忘录（Memento）：保存对象的状态，以便在需要的时候恢复对象。由备忘录对象、备忘录管理对象构成。
    ```java
    public class MementoObject {
        // 大量属性
        // ...

        public void saveMemeto();
    }

    public class MementoManager {
        MementoObject applicationState;

        public MementoObject getCurrentState() {
            return applicationState;
        }

        public void restoreState(MementoObject m) {
            applicationState = m;
        }
    }
    ```
- MVC：Model代表存取数据的对象，View包含可视化内容，Controller控制数据双向流动
    ```java
    // 播放器例子
    public interface PlayerModelInterface {
        void setSong(String song);
        void getSong(String song);
        // 仅在切歌时通知
        void registerObserver(SongObserver o);
        void removeObserver(SongObserver o);
        // 每一个采样都通知
        void registerObserver(TickObserver o);
        void removeObserver(TickObserver o);
    }

    // MVC中的Model
    // MetaEventListener来自java的MIDI库
    public class MyPlayerModel implements PlayerModelInterface, MetaEventListener {
        ArrayList songObservers = new ArrayList();
        ArrayList tickObservers = new ArrayList();

        void setSong(String song){
            notifySongObservers();
            // 开始播放
            // ...
        }

        void beatEvent() {
            notifyTickObservers();
        }
        
        // 观察者模式其余代码，略
    }

    public interface SongObserver {
        void updateSong(String song);
    }

    public interface TickObserver {
        void updateTick();
    }

    // 观察者接口实现代码，略

    // MVC中的View
    public class MyPlayerView implements ActionListener, SongObserver, TickObserver {
        PlayerModelInterface model;
        ControllerInterface controller;
        // GUI设计
        JButton setSongButton;
        JTextField songTextField;

        public MyPlayerView(ControllerInterface controller, PlayerModelInterface model) {
            this.controller = controller;
            this.model = model;
            // 界面视图需要根据两种信息绘制
            model.registerObserver((SongObserver)this);
            model.registerObserver((TickObserver)this);
        }

        public void updateSong(String song) {
            // ...
        }

        public void updateTick() {
            // ...
        }

        public void actionPerformed(ActionEvent event) {
            if(event.getSource() == setSongButton)) {
                controller.setSong(songTextField.getText());
            }
            // 其他界面控制
            // ...
        }
    }

    public interface ControllerInterface {
        void setSong();
    }

    public class PlayerController implements ControllerInterface {
        PlayerModelInterface model;
        MyPlayerView view;

        public PlayerController(PlayerModelInterface model) {
            this.model = model;
            // 在MVC模式中，控制器创造
            view = new MyPlayerView(this, model);
        }

        public void setSong(String song) {
            model.setSong(song);
        }
    }

    public class MainApplication {
        public static void main(String[] args) {
            PlayerModelInterface model = new MyPlayerModel();
            ControllerInterface controller = new PlayerController(model);
        }
    }
    ```
### 其他
- 空对象（Null Object）：取代空对象实例检测，反应一种不做任何动作的子类情况。
## 术语
1. 耦合性：指模块之间的信息、参数的依赖程度。
1. 内聚性：和耦合相对，指功能相关的程序组合成一个模块的程度。
1. 代理（proxy）和委托（delegate）：代理更为严格，要求若干对象实现同一个接口。委托相对简单，让一个对象扮演另外对象的行为，并在内部引用被委托对象。
## 图例
- 实现：新类型实现了接口
<center><img src="/images/ProgramDesign/ImplementUML.svg" ></center></br>

- 泛化：新类型泛化原有类型
<center><img src="/images/ProgramDesign/GeneralizationUML.svg" ></center></br>

- 组合：被组合类（CPU）是组合类（Computer）不可分割的一部分，这里我们假定处理器焊死在电脑上
<center><img src="/images/ProgramDesign/CompositionUML.svg" ></center></br>

- 聚合：被聚合类（Computer）是聚合类（Classroom）的一部分，两者地位并不平等，被聚合类是单独存在的
<center><img src="/images/ProgramDesign/AggregationUML.svg" ></center></br>

- 依赖：被依赖类（Logger）为依赖类型提供必要能力，但不是依赖类型的成员
<center><img src="/images/ProgramDesign/DependencyUML.svg" ></center></br>

- 关联：被关联类（Student）在关联类型（Teacher）中有保留，两者地位平等，不负责生命周期管理。也存在双向关联
<center><img src="/images/ProgramDesign/AssociationUML.svg" ></center></br>

## 参考资料
- 代码示例主要来源：《Head First设计模式》
- [24种设计模式大全](https://blog.csdn.net/yanlin813/article/details/52664805)
- [设计模式-菜鸟教程](https://www.runoob.com/design-pattern/design-pattern-tutorial.html)
- [桥接模式、外观模式、适配器模式的区别](https://www.cnblogs.com/peida/archive/2008/08/01/1257574.html)
- [设计模式-菜鸟教程](https://www.runoob.com/design-pattern/design-pattern-intro.html)
- [漫画讲解抽象工厂方法](https://developer.aliyun.com/article/904309)