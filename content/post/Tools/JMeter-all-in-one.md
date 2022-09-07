---
title: "JMeter压测工具"
date: 2022-01-06T16:01:53+08:00
categories:
- 计算机科学与技术
- 实用工具
tags:
- 实用工具
- 测试
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/jmeter.jpg
---
JMeter是Apache托管的一款用于进行接口测试的工具。功能比较丰富，还有插件可以进行拓展。
<!--more-->
## 安装
1. 前置安装JDK（配置好JAVA_HOME、CLASSPATH）
1. 去官网下载[二进制包传送门](https://jmeter.apache.org/download_jmeter.cgi)
1. 解压缩，windows下运行bin/jmeter.bat
## 测试计划示例
1. 配置测试相关变量
    <center>
    <img src="/images/tools/jmeter_var.png">配置变量</img>
    </center>
1. 创建线程组
1. 创建控制器（可选）
    - [Jmeter如何控制取样器执行顺序](https://blog.csdn.net/weixin_44102466/article/details/120022428)
1. 配置登录请求（可选）
    - 某些测试可能需要先获取token，需要先发一些其他请求并处理
        <center>
        <img src="/images/tools/jmeter_req.png">创建登录请求</img>
        </center>
    - 从登录请求中获取access_token（可选）
        <center>
        <img src="/images/tools/jmeter_req_extract.png">正则提取token</img>
        </center>
        <center>
        <img src="/images/tools/jmeter_var_script.png">将token设置为全局变量auth</img>
        </center>
1. 测试请求token填充
    <center>
    <img src="/images/tools/jmeter_var_script_usage.png">使用全局变量auth</img>
    </center>
1. 发送待测试请求
1. 创建响应断言
1. 查看结果树
## BeanShell
- 概述：BeanShell是JMeter所使用的脚本语言，BeanShell也支持了很多其他项目，具体可查看[BeanShell的Github主页](https://github.com/beanshell/beanshell)
- 主要特点：
    1. 可以在${}内用符号_拼接数字，来访问一批变量。也有些特殊单词如_matchNr代表此类变量的数量。
    1. 支持对象，对象也基本支持Java语法
    1. 内置：
        - 使用${__P}，${__setProperty}等内置函数读取/设置JMeter变量
        - 使用${__threadNum}等内置变量
- 一段JMeter后处理代码示例：
    ```java
    //log.info("stationCodes: ${stationCodes_2}");
    int i=${stationCodes_matchNr};

    log.info("station Count: " + String.valueOf(i));

    //${__setProperty(stationCodesOutput,${stationCodes},)}
    String s="";
    for(int j=1;j<=i;j++){
        s += vars.get("stationCodes_"+j)+",";
    }
    //String str = Arrays.toString(a);
    //vars.put("test",str);
    //log.info("????"+str);
    //${__setProperty(stationCodesOutput,str,)}
    props.put("test",s);
    //log.info("????: ${__P(stationCodesOutput,)}");
    String value = props.get("test");
    log.info("value : "+value);
    String trimValue = value.substring(0,value.length()-1);
    log.info("trimValue : "+trimValue);
    String[] stationCodesArray = trimValue.split(",");
    for(int i=0;i!=stationCodesArray.length;++i){
        vars.put("stationCodes_"+(i+1),stationCodesArray[i]);
    }
    ```
- 参考：
    - [BeanShell Wiki](https://github.com/beanshell/beanshell/wiki)
    - [Jmeter几种常用函数用法](https://blog.csdn.net/evanzhang_z/article/details/102715619)

## 插件
- JMeter对插件的支持很好，可以根据自己的需求找一些开源的第三方测试插件，主要步骤如下
    - 去jmeter插件官网下载jmeter-plugins-manager.jar
    - 将该jar抱放入jmeter内，/lib/ext目录下
    - 在jmeter内的菜单options->plugins manager菜单项，打开插件管理器
    - 安装你喜欢的插件
- WebSocket插件：websocketsamplers by peter doornbosch
    - 在取样器中和其他测试工具一样
- 参考：
    - [jmeter的websocket插件安装和使用方法](https://www.jianshu.com/p/3ad3497f12c7)
    - [jmeter插件官网](https://jmeter-plugins.org/)