---
title: "Android开发合集"
date: 2022-07-28T17:01:23+08:00
categories:
- 计算机科学与技术
- 安卓
tags:
- 滚动更新
- 安卓
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/android.jpeg
---
项目中偶尔需要一些Android开发，在此记录一下。API版本普遍为编译32，最低支持23。
<!--more-->
## 概述
Android是基于Linux开发的一款优秀的操作系统，尤其适用于在手机端使用，目前也是用户数量最多的操作系统。本文从实用角度出发，给出一些使用的项目代码记录。

## Android Studio安装
1. 最简单粗暴的办法，在HTTP Proxy设置中，直接使用Auto-detect proxy settings，然后用本机的任何上网代理即可。这也是**最稳定**的办法。
    - 但一定要在**第一次安装运行前就准备好代理**，否则可能出现包安装不完整导致的一系列非常奇怪的问题。
    - android studio的代理（SDK）和gradle代理（各类第三方库）并不是同一回事，在settings.gradle中还可以选择所使用的代码仓库的URL
1. 或者你也可以选择配置镜像：由于Android SDK存在版权问题，这里要搜索下可以使用的镜像源（清华TUNA在撰写本文时并不能使用）。可以使用中科院开源协会，填写如下图。
![Android Studio Proxy设置](/images/postImage/AndroidStudioProxy.jpg)
1. 由于Android Studio更新较快，新版本可能出现问题，建议在出现难以解决的问题的时候，可以使用低版本重试

## 编译篇
### SDK
### SDK Build Tool
### Gradle配置
1. 概述
    1. 全局配置是gradle.properties，位于用户目录的.gradle/下
    1. 项目配置是gradle-wrapper.properties，位于项目下
1. 代理：注意http和https都**必须写上**
    ```python
    #代理服务器IP/域名
    systemProp.http.proxyHost=127.0.0.1
    #代理服务器端口
    systemProp.http.proxyPort=10809
    #代理服务器IP/域名
    systemProp.https.proxyHost=127.0.0.1
    #代理服务器端口
    systemProp.https.proxyPort=10809
    ```
1. build.gradle示例
```java
plugins {
    id 'com.android.application'
}

android {
    compileSdk 32

    defaultConfig {
        applicationId "com.afclab.pr_tpuservice"
        minSdk 23
        targetSdk 32
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"

        // 自定义字段，将会最终写入到BuildConfig.java中
        buildConfigField("String","BUILD_TIME","\""+getBuildDate()+"\"")
    }


    // apk签名配置
    signingConfigs {
        release {
            storeFile file('../yourKey.jks')
            storePassword 'yourPassword'
            keyAlias 'yourKeyAlias'
            keyPassword 'yourKeyPassword'
        }
    }

    // 不同构建类型的配置
    buildTypes {
        debug {
            signingConfig signingConfigs.release
        }

        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {

    implementation 'androidx.appcompat:appcompat:1.4.2'
    implementation 'com.google.android.material:material:1.6.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
    // 在线下载的依赖
    implementation 'com.github.licheedev:Android-SerialPort-API:2.0.0'
    // 本地下载好的依赖
    implementation fileTree(dir: '..\\3rdParty', include: ['*.aar', '*.jar'], exclude: [])
    implementation 'com.google.code.gson:gson:2.9.1'
    implementation 'com.squareup.okhttp:okhttp:2.7.5'
    // 对本地其他项目的依赖
    implementation project(path: ':pr-common')
}

// 自定义函数要写在做外侧
String getBuildDate() {
    TimeZone.setDefault(TimeZone.getDefault().getTimeZone("GMT+8"));
    Date date =new Date();
    return date.toString();
}
```

### 调用外部C++库
1. 主要方式
    - 低难度：提供Java接口、so库。这意味着JNI代码已经写好了，使用者只需将so放到指定位置，调用Java接口即可
    - 高难度：C/C++接口、so库。需要用户编写编译规则和JNI代码
1. 坑：
    - 外部库的名字可以变动,调用加载时指定正确的名称即可，但是路径必须是固定的，而且要和so对应的CPU架构一致，如armeabi、armeabi-v7a、x86。
1. 参考
    - [菜鸟教程：JNI入门教程](https://www.runoob.com/w3cnote/jni-getting-started-tutorials.html)
    - [ANDROID动态加载 使用SO库时要注意的一些问题](https://segmentfault.com/a/1190000005646078)
## 原理
1. 四大组件：活动（Activity）、服务（Service）、广播接收者（BroadCast Receiver）、内容提供者（Content Provider）
## 代码应用
### 数据存储
- 五种主要方式
    - SharePreferences：本质是xml文件
    - SQLite
    - Content Provider：唯一可以用于共享的办法，用一个uri标记资源，如content://路径/id
    - File：一般用于大文件
    - 网络存储
### ListView
- ListView容易出现卡顿问题，一定要做其中的View优化
- 最好准备完整的数据之后再进行setAdapter，减少触发数据变动
    - 而且在回调中的数据变动，不一定会立刻影响到界面绘制
- 参考：
    - [Android最常用的控件ListView](https://blog.csdn.net/indeedes/article/details/119530068)
### 权限获取

### 开机自启动
- 流程：配置监听权限、设置监听类、监听系统广播并启动
- 参考：[设置程序开机自启动](https://blog.csdn.net/CSDNHAY/article/details/120785513)
### Activity Manager Service
- 参考:
    - [Android7.1 AMS概述](https://www.freesion.com/article/1752426241/)

### 跨线程通信
- 关键词：
    - Handler：核心方法如sendMessage、handleMessage，用于发送、处理一个消息
    - Looper：执行Handler的handleMessage所在的事件循环线程
    - Message、Bundle：发送消息的数据结构
- 基本流程
    - \[可选\]自定义Handler，尤其是handleMessage函数
    - 创建Handler、创建Looper，启动Looper
    - 将Handler发给不同的线程
    - 各线程内自己组Message并发送
- 示例代码
    - 待补充
### Camera2和TextureView
- 关键词
    - 待补充
    - 补充一张Camera2的类图
- 基本流程
    - 为TextureView添加SurfaceViewListener
    - 由于TextureView的SurfaceView在Listener的onAvailable阶段才可用，所以初始化需要放到该位置
    - 根据需要调用capture拍照、或者setRepeatingRequest预览
- 难点
    - 记得控制画面长宽
    - 相机权限获取
    - 保证使用到的各种所有变量都是类成员变量，尤其是Surface，不能作为函数局部变量，可能导致被GC回收从而引发异常
- 参考
    - [CAMERA2 API 采集视频并SURFACEVIEW、TEXTUREVIEW 预览](https://www.freesion.com/article/3644114052/)
    - [Camera2 Camera1](https://yeungeek.github.io/2020/01/24/AndroidCamera-Orientation/)
    - [预览同时ImageReader获取图片](http://codingsky.com/doc/2022/4/19/842.html)
    - [基于Camera2实现边录制视频边实时分析图片](https://blog.csdn.net/m0_37697747/article/details/122077631)
    - [Android Camera2 全屏预览+实时获取预览帧进行图像处理](https://blog.csdn.net/qq_38743313/article/details/101557079?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-3-101557079-blog-122077631.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-3-101557079-blog-122077631.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=6)：这一篇讲了一些坑和处理方法
    - [ImageReader 丢帧卡顿](https://blog.csdn.net/xuhui_7810/article/details/104402300)
    - [ImageReader Surface 垃圾回收问题](https://qa.1r1g.com/sf/ask/2340657301/)

### 无界面Service及通信
- 基本流程
    - 根据需要继承Service、IntentService，实现必要的函数，并提供Binder、AIDL（Android Interface Define Language）
        ```java
        // Service主体
        public class TPUService extends Service {
            private String TAG = "TPUService";

            @Override
            public void onCreate() {
                super.onCreate();
                Log.i(TAG, "onCreate");
            }

            @Nullable
            @Override
            public IBinder onBind(Intent intent) {
                Log.i(TAG, "onBind");
                // 在此处提供接口的具体实现
                return new TPUServiceInterface.Stub() {
                    @Override
                    public String ping() throws RemoteException {
                        return "pong";
                    }

                    @Override
                    public boolean reset() throws RemoteException {
                        return false;
                    }

                    @Override
                    public boolean init() throws RemoteException {
                        return false;
                    }

                    @Override
                    public String readCard() throws RemoteException {
                        return null;
                    }
                };
            }
        }

        // 接口主题（.aidl文件），该文件会被Android Studio自动转为java文件
        interface TPUServiceInterface {
            String ping();
            boolean reset();
            boolean init();
            String readCard();
        }
        ```
    - 由于无界面，启动上一般需要设置开机启动、或者允许由其他程序启动，则其AndroidManifest.xml一般形如
        ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.afclab.pr_tpuservice">
            <!-- 自启动权限 -->
            <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
            <application
                android:allowBackup="true"
                android:icon="@mipmap/ic_launcher"
                android:label="@string/app_name"
                android:roundIcon="@mipmap/ic_launcher_round"
                android:supportsRtl="true"
                android:theme="@style/Theme.PRTPUService">
                <!-- 接收启动完成广播信号 -->
                <receiver
                    android:name="com.afclab.pr_tpuservice.OnBootBroadCast"
                    android:enabled="true"
                    android:exported="true">
                    <intent-filter android:priority="1000">
                        <action android:name="android.intent.action.BOOT_COMPLETED"></action>
                    </intent-filter>
                </receiver>
                <!-- exported设置为true，允许其他应用启动 -->
                <service android:name=".TPUService" android:exported="true"/>
            </application>

        </manifest>
        ```
    - 自定义广播信号处理类，接收广播，并运行自定义Service，代码形如
        ```java
        public class OnBootBroadCast extends BroadcastReceiver {
            @Override
            public void onReceive(Context arg0, Intent arg1) {
                Intent mBootIntent = new Intent(arg0, TPUService.class);
                arg0.startService(mBootIntent);
            }
        }
        ```
    - 受安卓系统限制（Android3.1+），初次安装后如果不启动一次，则并不会真正监听启动信号。如果不由外部应用启动，则需要通过adb，手动启动一次，此后监听将会正式生效。
    - 客户端使用
        ```java
        public class MainActivity extends AppCompatActivity {
            private String TAG = "MainActivity";
            private TPUServiceInterface mService = null;
            // 连接
            private ServiceConnection mConnection = new ServiceConnection() {
                @Override
                public void onServiceConnected(ComponentName componentName, IBinder iBinder) { //绑定Service成功后执行
                    mService = TPUServiceInterface.Stub.asInterface(iBinder);//获取远程服务的代理对象，一个IBinder对象
                    try {
                        Log.i(TAG, mService.ping()); //调用远程服务的接口。
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                }

                @Override
                public void onServiceDisconnected(ComponentName componentName) {
                    mService = null;
                }
            };

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
                Button connectButton = findViewById(R.id.service_connect);
                connectButton.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        // 必须要正确提供包名、服务名，并传递给Intent
                        ComponentName componentName = new ComponentName("com.afclab.pr_tpuservice", "com.afclab.pr_tpuservice.TPUService");
                        Intent intent = new Intent();
                        intent.setComponent(componentName);
                        boolean result = bindService(intent, mConnection, BIND_AUTO_CREATE);
                        Log.i(TAG, "bind result: " + result);
                    }
                });
            }
        }
        ```
- 参考
    - [Android进程间通信](https://www.jianshu.com/p/e64b0a9472db)
    - [Android Service的跨进程通信实战&Service/AIDL远程调用过程解析（Android Q）](https://blog.csdn.net/u011578734/article/details/106000423)
## ADB
1. 概述：Android Debug Bridge，即ADB，安卓调试桥，可以通过ADB完成各种安装、调试、调用系统shell等能力。由于Android Studio有良好的调试工具，个人建议只把adb作为辅助手段，但是对于批量安装应用等场景，adb还是更有优势
1. 一些指令
    ```bash
    # 主机侧
    adb root # 进入root
    adb shell # 进入shell
    adb connect x.x.x.x # 连接置顶ip的安卓设备
    adb tcpip 5555 # 尝试打开对端5555端口连接
    adb install -r xxx.apk # 覆盖安装
    adb shell am start com.你的包路径/com.你的Activity # 启动普通程序
    adb shell am startservice com.你的包路径/com.你的Service # 启动服务
    adb shell screenrecord /存储/路径.mp4 # 录制屏幕
    adb shell input ... # 模拟各类输入事件（键鼠等）
    adb shell logcat >> xxx.txt # 将日志追加输出到指定文件
    adb -s (ip):5555 shell # 指定连接

    # 安卓shell侧
    setprop service.adb.tcp.port 5555
    pm list packages # 查看已安装的包
    pm install 你的包路径 # 安装
    pm uninstall 包名 # 卸载
    am force-stop 包名 # 杀程序

    # adb shell命令也都可以用在这里
    # .. 可以使用其他部分linux指令
    ```
1. 设置永久网络调试接口：adb实际上默认就是开启的，网络端口也是，可以直接尝试连接（首次连接也会需要安卓端确认）
    - 参考：[adb网络连接调试，重启之后失效](https://blog.csdn.net/ezconn/article/details/103358710)

## 实用第三方库
> repositories一般会添加：maven(jitpack.io),jcenter，方便查找正确的包
1. 串口通信：com.github.licheedev:Android-SerialPort-API
## 坑
1. 新建项目提示类似于：plugin com.android.application not found in any repositories
    - 尚不清楚原理，但是大概率和gradle有关，建议删除默认gradle目录(C:/Users/你的名字/.gradle/)，完全重新下载。
    ![删除gradle目录](/images/postImage/delete_gradle_to_redownload.jpg)
    - 其他的可能性包括：仓库中确实没有该版本、gradle和插件版本不兼容、gradle和gradle所用java版本不兼容（Gradle7.0+要求java11）、网路代理错误等
    - [参考](https://metapx.org/plugin-with-id-com-android-application-not-found/)
1. 设置整个Layout的背景是图片动画，会造成绘制掉帧
    - 不应将图片直接放到drawable下面，而应当放到mipmap下
    - 当然实际上也不推荐直接用图片做背景
1. 安卓设备连接上USB设备时，必须使用弹出式的权限获取窗口来获取权限，不能自动授权。该权限是动态获取的，在AndroidManifest中无法进行静态获取。
1. Gson和Fastjson在对于byte[]的序列化方面，并不兼容，尽量使用同一种序列化方式。
## 参考资料
- [2022最新Android基础视频教程](https://www.bilibili.com/video/BV19U4y1R7zV)
- [Google官方文档](https://developer.android.google.cn/reference)
    - Android Studio已经将文档融合进Android SDK源代码中，只需要安装项目所使用版本的SDK时同时安装其源代码即可