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
## 为什么要使用注解
1. 注解能够将一些元数据直接写入代码当中。而不需要另写文档。
1. 代码更加易于阅读，并且拥有了编译期类型检查的能力。
1. 可以用于控制编译期代码的生成。
> 总之，一旦你的代码中出现了重复性的工作，你就可以考虑使用注解来简化、自动化该过程。
## 核心原理
1. 运行期注解：利用Java的反射机制，在运行期对注解进行解析。
1. 编译期注解：逐轮次处理注解，生成新的源文件
1. 字节码工程
## 内置注解
1. 标准注解：
    1. @Override：表示当前方法将覆盖父类中的方法。
    1. @Deprecated：表示当前方法已被废弃，如果被使用，编译器会发出警告。
    1. @SuppressWarnings：关闭不恰当的编译器警告信息
1. 元注解：
    1. @Target：说明注解的作用对象，可选ElementType.CONSTRUCTOR / FIELD / LOCAL_VARIABLE / METHOD / PACKAGE / PARAMETER / TYPE
        - 可以使用逗号分隔，添加多个
        - 不使用@Target则可默认作用于任何目标
        - 可以用于作用对象出现的任何位置，比如作为TYPE注解，则可以出现在泛型类的类型参数中，如:public class Cache<@Immutable V>
    1. @Retention：说明注解的作用级别，可选RetentionPolicy.SOURCE/CLASS(默认)/RUNTIME
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
## 运行时注解示例代码
### 一种创建SQL数据表的注解写法
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
### 事件监听器
1. 注解和使用部分
```java
// 监听器注解，只需要提供监听信号源名称
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ActionListenerFor {
    String source();
}

// 业务代码
public class ControlPanel {
    private JPanel panel;
    private JButton setPanelRedButton;

    public ControlPanel() {
        panel = new JPanel();
        setPanelRedButton = new JButton("Red");
        // some other gui setting
        // ...
        // 在构造函数中对监听事件进行关联
        try {
            ActionListenerInstaller.processAnnotations(this);
        } catch (ReflectiveOperationException e) {
            e.printStackTrace();
        }
    }

    // 事件监听回调
    @ActionListenerFor(source = "setPanelRedButton")
    public void redPanel() {
        panel.setBackground(Color.RED);
    }
}
```
2. 注解处理器部分
```java
// 安装监听回调
public class ActionListenerInstaller {
    // 和swing框架相关的特定关联方法
    public static void addListener(Object source, final Object param, final Method m)
            throws ReflectiveOperationException {
        InvocationHandler handler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                return m.invoke(param);
            }
        };
        Object listener = Proxy.newProxyInstance(null
                , new Class[]{java.awt.event.ActionListener.class}, handler);
        Method adder = source.getClass().getMethod("addActionListener", ActionListener.class);
        adder.invoke(source, listener);
    }

    public static void processAnnotations(Object obj) throws ReflectiveOperationException {
        // 获取当前obj的所属类型
        Class<?> cl = obj.getClass();
        for (Method m : cl.getDeclaredMethods()) {
            // 获取包含的监听注解
            ActionListenerFor listener = m.getAnnotation(ActionListenerFor.class);
            if (listener != null) {
                // 查找监听来源，这里要求监听源必须同在当前类型
                Field f = cl.getDeclaredField(listener.source());
                f.setAccessible(true);
                addListener(f.get(obj), obj, m);
            }
        }
    }
}
```
## 编译时注解处理
1. 原有工具的英文名称为Annotation Processing Tool（APT），目标是直接处理源文件。java8之后该部分已经直接迁移到javac内部。
1. 处理原理：注解处理器将会从最初的源文件开始，逐轮次处理注解，并**产生新的源文件**，直到不再有新的源文件产生。然后再进行传统的java源文件编译。
1. 使用方法
    1. 选择RetentionPolicy.SOURCE
    1. 继承AbstractProcessor类，并实现处理器processor函数。
        - 通常需要声明支持处理的注解，如某个包下面（com.xxx.xxx)，或者全部（*）
    1. 通过Messager来打印信息，该信息将会输出于IDEA的build窗口中，一定要使用rebuild。
    1. 返回true代表处理过，返回false则还会交给其他处理器尝试处理。
1. 调用编译：javac -processor ProcessorClassName1,ProcessorClassName2, ... sourceFiles
    - 注意这里需要先生成处理类的字节码，无论实用原生javac，还是maven、gradle，都需要先编译处理器。**你得先有一只组装鸡，才能有蛋**。因此实际上的使用例如
        ```bash
        # 方法一，纯手动
        # 从包起始目录执行
        javac ./your/package/name/MyProcessor.java
        javac -verbose -processor your.package.name.MyProcessor \
            ./your/package/name/MainClass.java \
            ./your/package/name/TestClass.java \
            ...
        
        # 方法二（推荐），打jar包并使用META-INF/services/
        # 先编译打包
        javac ./your/package/name/MyProcessor.java
        echo "your.package.name.MyProcessor" > \
            META-INF/services/javax.annotation.processing.Processor
        # 将META-INF一并打包（注意META-INF和包顶层目录同级）
        jar -cvf MyProcessor.jar ./your/package/name/MyProcessor.class META-INF/
        # 使用
        javac -verbose -cp MyProcessor.jar ./your/path/to/ToBeProcessed.java
        ```
    - 和maven搭配使用，需要配置的编译内容（但是目前还没编写一个有效的例子）
        ```xml
        <!-- pom.xml -->
        <!-- 暂时不使用IDEA内置的Setting中的Annotation Processor，使用pom.xml就可以了 -->
        <!-- 注解处理器项目配置 -->
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                    <configuration>
                        <!-- 必加，处理器项目不应进行注解处理 -->
                        <proc>none</proc>
                        <annotationProcessors>
                            <annotationProcessor>
                                ToStringAnnotationProcessor
                            </annotationProcessor>
                        </annotationProcessors>
                    </configuration>
                </plugin>
            </plugins>
        </build>

        <!-- pom.xml -->
        <!-- 需要使用注解处理器的项目配置 -->
        <!-- 把你的处理器项目列为依赖 -->
        <dependencies>
            <dependency>
                <groupId>org.lyclab</groupId>
                <artifactId>lyc-annotation-processor</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                    <configuration>
                        <annotationProcessors>
                            <annotationProcessor>
                                ToStringAnnotationProcessor
                            </annotationProcessor>
                        </annotationProcessors>
                    </configuration>
                </plugin>
            </plugins>
        </build>
        ```
1. 编译期和运行期的一些处理区别：
    1. 编译期处理只能使用语言模型API，即mirror API来分析源码级的注解。即编译器产生的源码树结构。
        - 起这个名字是因为，镜子能够起到反射的作用。
    1. getAnnotation受限，很多场景需要使用getAnnotationMirror。
        - **尚不清楚具体限制场景**
1. java的SPI思想（待完善）
    - SPI风格：客户代码继承并实现接口，服务方负责调用）
        - 接口和使用者位于同侧
        - 天然适合插件开发
        - 使用META-INF/resources/services/javax.****对AnnotationProcessor进行配置时，就是在使用SPI了
    - API风格：服务方负责继承并实现接口，客户代码进行调用
        - 接口和实现位于同侧
1. 示例代码（为若干同父类的类型自动创建工厂方法）
```java
// 注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface Factory {
    // 填写父类
    Class type();
    // 填写子类
    String id();
}

// 处理器
@SupportedAnnotationTypes("lyc.annotation.Factory")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class FactoryAnnotationProcessor extends AbstractProcessor {

    Messager messager;
    Filer filer;
    Elements elementUtil;
    Types typeUtil;


    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        messager = processingEnv.getMessager();
        filer = processingEnv.getFiler();
        // 可以使用utils简化很多步骤
        elementUtil = processingEnv.getElementUtils();
        typeUtil = processingEnv.getTypeUtils();
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations
            , RoundEnvironment roundEnv) {
        try {
            messager.printMessage(Diagnostic.Kind.NOTE, "Begin Annotation " +
                    "Process");
            // 存储处理过的类型，用于统一生成
            Map<String, TypeElement> factoryElements = new HashMap<>();
            String qualifiedName = null;
            // 如果没有需要处理的类型，直接退出
            if (roundEnv.getElementsAnnotatedWith(Factory.class).isEmpty())
                return true;
            for (Element element :
                    roundEnv.getElementsAnnotatedWith(Factory.class)) {
                messager.printMessage(Diagnostic.Kind.NOTE,
                        "element: " + element.toString());
                // 一些必要的检查：处理目标必须是类型
                if (element.getKind() != ElementKind.CLASS) {
                    messager.printMessage(Diagnostic.Kind.NOTE, "Factory " +
                            "Annotation should only describe class.");
                    return false;
                }
                // 一些必要的检查：处理目标不能是抽象类
                if (element.getModifiers().contains(Modifier.ABSTRACT)) {
                    messager.printMessage(Diagnostic.Kind.NOTE, "Factory " +
                            "Annotation should describe a non abstract class.");
                    return false;
                }
                // // 一些必要的检查：id 应当为 type子类
                // 这里就需要对java的一些惯性思维，父类只能由一个，而接口可以有多个，都需要检查
                // 使用mirror api，获取注解信息
                List<? extends AnnotationMirror> anns =
                        element.getAnnotationMirrors();
                // 提取注解中的type字段，并转为TypeMirror
                // 这一段实际可以用elementUtil代替
                TypeMirror superClassType = null;
                for (AnnotationMirror annotationMirror : anns) {
                    if (annotationMirror.getAnnotationType()
                        .toString().equals(Factory.class.getName())) {
                        for (Map.Entry<? extends ExecutableElement, ?
                                extends AnnotationValue> entry
                                : annotationMirror.getElementValues().entrySet()) {
                            // 一定要记得添加toString
                            if (entry.getKey().getSimpleName().toString().equals("type")) {
                                superClassType =
                                        (TypeMirror) entry.getValue().getValue();
                            }
                        }
                    }
                }
                // 检查id是否是type的子类
                TypeElement typeElement = (TypeElement) element;
                if (superClassType instanceof DeclaredType) {
                    qualifiedName =
                            ((TypeElement) ((DeclaredType) superClassType).asElement()).
                            getQualifiedName().toString();
                    messager.printMessage(Diagnostic.Kind.NOTE,
                            "qualifiedName: " + qualifiedName);
                    // 分接口和类型分别处理
                    if (((DeclaredType) superClassType).asElement().getKind()
                            == ElementKind.INTERFACE) {
                        messager.printMessage(Diagnostic.Kind.NOTE, "Super is" +
                                " Interface");
                        if (!typeElement.getInterfaces()
                            .contains(((DeclaredType) superClassType)
                                .asElement().asType())) {
                            // 使用ERROR输出会直接判定失败
                            messager.printMessage(Diagnostic.Kind.ERROR
                                , "Super Interface not match");
                            return false;
                        }
                    } else {
                        messager.printMessage(Diagnostic.Kind.NOTE, "Super is Class")
                        TypeMirror superClassMirror = ((TypeElement) element).getSuperclass();
                        // 循环向上检查父类
                        while (superClassMirror.getKind() != TypeKind.NONE) {
                            if (superClassMirror.toString()
                                .equals(superClassType.toString())) {
                                break;
                            }
                        }
                        if (superClassMirror.getKind() == TypeKind.NONE) {
                            messager.printMessage(Diagnostic.Kind.ERROR
                                , "Super class not match");
                            return false;
                        }
                    }
                } else {
                    messager.printMessage(Diagnostic.Kind.ERROR
                        , "super class should be class | interface.");
                }
                messager.printMessage(Diagnostic.Kind.NOTE, "super check pass");
                // 添加到待生成
                factoryElements.put(element.getSimpleName().toString(), (TypeElement) element);
            }
            messager.printMessage(Diagnostic.Kind.NOTE, "Generating");
            generateClassCode(factoryElements, qualifiedName);
            messager.printMessage(Diagnostic.Kind.NOTE, "Finish Annotation Process");
        } catch (Exception e) {
            // 有时会有空消息
            if (e.getMessage() != null)
                messager.printMessage(Diagnostic.Kind.NOTE,
                        "my output: " + e.getMessage());
            StringBuilder builder = new StringBuilder();
            // 有时会有空stacktrace
            if (e.getStackTrace() != null) {
                for (StackTraceElement element : e.getStackTrace()) {
                    builder.append("my output: " + element.toString());
                }
            }
            String output = builder.toString();
            if (output.isEmpty())
                output = "Exceptions without any message";
            messager.printMessage(Diagnostic.Kind.ERROR, output);
        } finally {
            messager.printMessage(Diagnostic.Kind.NOTE, "finish!!");
        }
        return true;
    }

    private void generateClassCode(Map<String, TypeElement> elements, String qualifiedName)
     throws IOException {
        String suffix = "Factory";
        // 获取父类名称
        TypeElement superClassName = elementUtil.getTypeElement(qualifiedName);
        // 生成工厂名称
        String factoryClassName = superClassName.getSimpleName() + suffix;
        // 工厂名称（全名）
        String qualifiedFactoryName = qualifiedName + suffix;
        // 创建package
        PackageElement pkg = elementUtil.getPackageOf(superClassName);
        String packageName = pkg.isUnnamed() ? null :
                pkg.getQualifiedName().toString();
        // JavaPoet库，先创建函数
        MethodSpec.Builder method = MethodSpec.methodBuilder("create")
                .addModifiers(Modifier.PUBLIC)
                .addParameter(String.class, "id")
                .returns(TypeName.get(superClassName.asType()));
        method.beginControlFlow("if(id==null)")
                .addStatement("throw new IllegalArgumentException($S)", "id " +
                        "is null")
                .endControlFlow();
        for (Map.Entry<String, TypeElement> entry : elements.entrySet()) {
            method.beginControlFlow("if(id.equals($S))", entry.getKey())
                    .addStatement("return new $L()",
                            entry.getValue().getQualifiedName())
                    .endControlFlow();
        }
        method.addStatement("throw new IllegalArgumentException($S+id)",
                "unknown id = ");
        // 再创建类型
        TypeSpec typeSpec = TypeSpec
                .classBuilder(factoryClassName)
                .addModifiers(Modifier.PUBLIC)
                .addMethod(method.build())
                .build();
        // 写入文件
        JavaFile.builder(packageName, typeSpec).build().writeTo(filer);
    }
}

```
8. 问题与解决：
    1. 生成的内容不一定能正确在代码编辑器中作为普通类型使用。
        - 出现Java File outsize of the source root。需要手动给出生成代码的根目录
        <center><img src="/images/JavaSeries/APTDebug_MarkGenRoot.png"></center>
9. Debug:
    1. 对注解处理器的Debug设置非常重要，在这里简述一下配置方式（IDEA）
        1. 为注解处理器项目添加远程Debug配置
            <center><img src="/images/JavaSeries/APTDebug_Configuration.png">配置APTDebug</center>

        1. 为IDEA配置VM选项
            <center><img src="/images/JavaSeries/APTDebug_VMOptions.png">-Dcompiler.process.debug.port=8000</center>
        
        1. 重启IDEA，使VM选项生效
        1. 开启Build Debug Process
            <center><img src="/images/JavaSeries/APTDebug_EnableDebugBuild.png"></center>

        1. 使用方式：
            1. clean来一套
            1. 先rebuild待处理程序，此时build过程会持续等待远程Debug端口
            1. 以Debug方式运行APTDebug
            1. 开始调试吧
## 字节码工程
1. 在字节码级别上进行处理，是在源码和运行时之外的第三种处理情况。处理字节码文件是相当复杂的事情，一般需要借助一些特殊类库，如AMS。
1. 字节码速学：
    1. 字节码以类为单位。
    1. 阅读可使用javap -verbose xxx.class，但更建议使用Idea的JClasslib插件阅读。
    1. 文件内容依次为：
        | 类型 | 名称 | 数量 |
        | --- | --- | --- |
        | u4 | magic | 1 |
        | u2 | minor_version | 1 |
        | u2 | major_version | 1 |
        | u2 | constant_pool_count | 1 |
        | cp_info | constant_pool | constant_pool_count - 1 |
        | u2 | access_flags | 1 |
        | u2 | this_class | 1 |
        | u2 | super_class | 1 |
        | u2 | interfaces_count | 1 |
        | u2 | interfaces | interfaces_count |
        | u2  | fields_count | 1 |
        | field_info | fields | fields_count |
        | u2 | methods_count | 1 |
        | method_info | methods | methods_count |
        | u2 | attributes_count | 1 |
        | attribute_info | attributes | attributes_count |

    1. 其中：
        - cp_info将包含方法名、字段名等等各类常量。而this_class/super_class的值实际是对cp_info的索引。
        - method_info字段中将会包含代码。实际上很多字段都是常量池的索引。
    1. 描述符标识字符（用于描述方法和字段的类型信息）
        | 标识字符 | 含义 | 标识字符 | 含义 |
        | --- | --- | --- | --- | --- |
        | B | byte | J | long | 
        | C | char | S | short |
        | D | double | Z | boolean |
        | F | float | V | void |
        | I | int | L | 类类型 |
        | 前置[ | 一层数组 | 前置() | 代表方法的参数列表 |
    1. Code属性
        - 包含属性长度、操作数栈最大深度、局部变量所需存储空间、字节码长度、指令字节流、异常表、其他属性等
    1. 字节码指令简介（T替换为各种类型）
        - 加载型(Tload)：如iload、fload、dload、aload、iload_\<n\>...
            - 常量加载型：bipush、sipush、ldc、aconst_null、Tconst...
        - 存储型(Tstore)：如istore、fstore、dstore、astore、istore_\<n\>...
        - 运算型：Tadd、Tmul、Tdiv、Trem、Tneg、Tshl、Tor、Tand、Tinc、dcmpg...
        - 类型转换型（宽化自动，窄化是有指令的）：i2b、i2c、i2s...
        - 对象创建和访问型：new、newarray、get/putfield、get/putstatic、Taload、Tastore、arraylength、instanceof、checkcast...
        - 操作数栈管理型：pop、pop2、dup、dup2、swap...
        - 控制型：ifeq、iflt、tableswitch、lookupswitch、goto...
        - 方法调用和返回指令：invokevirtual、invokeinterface、invokespecial、invokestatic、invokedynamic、Treturn、return
        - 异常处理：athrow
        - 同步指令：monitorenter（以栈顶元素作为锁）、monitorexit、方法级的同步则是隐式的（ACC_SYNCHRONIZED标识）
    1. 从字节码定义中也可以看出，Java天生就存在一些限制，比如：单文件的常量数上限一定是小于65535的。还有设计上取巧的地方，比如大部分byte、short都没有单独指令，都是和int一起做的，类型自动转换。
1. 注意点
    1. 使用Idea的话，待处理的工程必须Rebuild，即必须删除原有class。毕竟字节码处理之后，源代码build不会刷新字节码文件。
    1. AnnotationVisitor不会visit使用默认值的注解元素。
    1. 修改字节码需要对JVM和字节码有较深入理解。其水平相当于你可以跳过Java而直接编写字节码程序。
    1. 需要确保注解将会出现在字节码处理阶段。
1. 独立处理示例：
```java
// 注解
public @interface LogEntry {
    String logger();
}


// 字节码工程，注解处理器
public class EntryLogger extends ClassVisitor {
    private String className;

    public EntryLogger(ClassWriter writer, String className) {
        // 字节码处理和输出初始化
        super(Opcodes.ASM5, writer);
        this.className = className;
    }

    // 本注解处理只针对函数进行
    @Override
    public MethodVisitor visitMethod(int access, String methodName
            , String desc, String signature, String[] exceptions) {
        
        System.out.println("visitMethod: " + methodName + ", desc: "
                + desc + ", signature: " + signature + ", access: " + access);
        // 根据父类classvisitor，创建methodVisitor
        MethodVisitor mv = cv.visitMethod(access, methodName, desc, signature, exceptions);
        
        // 返回MethodVisitor包装类，内部实现处理逻辑
        return new AdviceAdapter(Opcodes.ASM5, mv, access, methodName, desc) {
            private String loggerName;

            // 注解处理（获取注解信息部分）
            @Override
            public AnnotationVisitor visitAnnotation(String annotation, boolean b) {
                System.out.println("visitAnnotation: " + annotation);

                // 按照键值对儿，获取注解信息
                return new AnnotationVisitor(Opcodes.ASM5) {
                    @Override
                    public void visit(String key, Object value) {
                        System.out.println("AnnotationVisitor: " + key + ", object " + value);
                        // 当前注解仅处理LogEntry，此处获取注解中配置的logger名称
                        if (annotation.equals("Lannotation/bytecode/LogEntry;")
                                && key.equals("logger")) {
                            loggerName = value.toString();
                        }
                    }
                };
            }

            // 进入该函数的字节码段，并操纵字节码
            @Override
            protected void onMethodEnter() {
                System.out.println("Enter Method, loggerName: " + loggerName);
                // 仅当能获取到loggerName时，才代表该函数有此注解，需要处理
                if (loggerName != null) {
                    // 增加字节码：将loggerName压栈
                    visitLdcInsn(loggerName);
                    // 增加字节码：调用静态函数java.util.logging.Logger.getLogger
                    //            后续参数代表了函数信息（参数类型和返回值）
                    visitMethodInsn(INVOKESTATIC, "java/util/logging/Logger", "getLogger",
                            "(Ljava/lang/String;)Ljava/util/logging/Logger;", false);
                    // 增加字节码：压入日志信息
                    visitLdcInsn("from entry logger annotation");
                    // 增加字节码：调用虚函数info，打印日志
                    visitMethodInsn(INVOKEVIRTUAL, "java/util/logging/Logger", "info",
                            "(Ljava/lang/String;)V", false);
                    // 清空loggerName
                    loggerName = null;
                }
            }
        };
    }
}

// 测试类
public class TestClass {
    @LogEntry(logger = "global",)
    public static void helloWorld(){
        System.out.println("hello world");
    }
}

// 主类，启动入口
public class Application {

    public static void main(String[] args) {
        TestClass myTest = new TestClass();
        myTest.helloWorld();
        // 用法，给定需要处理的class文件
        if (args.length == 0) {
            System.out.println("USAGE: java annotation.bytecode.EntryLogger classfile");
            exit(1);
        }
        Path path = Paths.get(args[0]);
        try {
            // 打开该字节码、处理并输出
            ClassReader reader = new ClassReader(Files.newInputStream(path));
            ClassWriter writer = new ClassWriter(
                    ClassWriter.COMPUTE_MAXS | ClassWriter.COMPUTE_FRAMES
            );
            // 必要的路径处理
            EntryLogger entryLogger = new EntryLogger(writer,
                    path.toString().replace(".class","")
                            .replaceAll("[/\\\\]","."));
            System.out.println("origin path: " + path.toString());
            System.out.println("path change to : " + path.toString().replace(".class","")
                    .replaceAll("[/\\\\]","."));
            reader.accept(entryLogger,ClassReader.EXPAND_FRAMES);
            Files.write(Paths.get(args[0]),writer.toByteArray());

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
5. 在加载时处理
    1. 字节码工程可以延期到加载时进行，这样更自然，不会修改原有class文件，不过也确实会降低加载性能。
    1. java 1.5引入了java agent。通过加载时调用额外的代理来完成加载时的处理。
    1. 基本步骤
        - 编写一个实现了注解处理和代理的java程序，填写代理信息到清单文件中
        - 编译生成jar包。
        - 运行时调用方式为：java -javaagent:XXX.jar=xxx -classpath xxx
    1. 代码暂略
    

## 参考内容
- [cnblogs编译期生成](https://www.cnblogs.com/LQBlog/p/14208046.html)
- [csdn编译期生成](https://blog.csdn.net/kaifa1321/article/details/79683246)
- [javac选项](https://www.cnblogs.com/itxiaok/p/10356513.html)
- [idea和编译期生成注解](https://zhuanlan.zhihu.com/p/95015043)
- [编译期生成注解：jar和maven](https://segmentfault.com/a/1190000020122395)
- [编译期生成注解：以工厂模式为例](https://blog.csdn.net/qq_20521573/article/details/82321755)
- [Java Element.getAnnotationMirrors方法代码示例](https://vimsky.com/examples/detail/java-method-javax.lang.model.element.Element.getAnnotationMirrors.html)
- [深入理解JVM（七）——插件化注解处理器](https://www.sukaidev.top/2021/06/25/19e6281/)
- [Adavanced Java-Annotation Processing](https://www.youtube.com/watch?v=HaCXOYptHqE)
- [JavaSE8手册](https://docs.oracle.com/javase/8/docs/api/)
- [ASM手册](https://asm.ow2.io/javadoc/index.html)
- [字节码查看方式](https://www.cnblogs.com/javaguide/p/13810777.html)
- [Java Agent Premain](https://www.jianshu.com/p/0bbd79661080)
- 《深入理解Java虚拟机（JVM高级特性与最佳实践）》