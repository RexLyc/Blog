---
title: "IBM WebSphere MQ"
date: 2021-07-27T16:43:16+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/IBM.jpg
---
由于工作原因，需要使用到IBMMQ，由于相关资料较少，所以在此特地整理一篇。尽力涵盖安装、配置、使用。
<!--more-->
## 安装
最离谱的事情就是安装，尚且不知道从哪里能一步步找到到Linux的服务器安装包连接，以及Windows的Explorer客户端的安装包。
- 网上查到的安装包网址：
    - [下载无需安装的压缩包](http://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/messaging/mqadv/)
- Linux
    - 找到压缩包之后（名称如：WS_MQ_V8.0.0.2_LINUX_ON_X86_64_RELEASE.tar.gz），解压。
    - server目录下，./mqlicense.sh -accept
    - rpm安装各个包
        - e.g.：rpm -ivh MQSeriesRuntime-xxxx.rpm
    - 各包内容
        - Runtime（MQSeriesRuntime），运行时环境，必须安装（客户端和服务器端都需要）
        - Server（MQSeriesServer），运行队列管理器和提供消息队列服务，服务器端需要
        - Client（MQSeriesClient），MQ的很小的功能子集，连接Server组件，不提供队列管理器
        - SDK（MQSeriesSDK),开发需要，用来编译应用程序
        - Sample programs，示例程序
        - Java messaging（MQSeriesJava），支持Java消息服务功能（JMS）
        - Man pages，帮助文档，提供control命令，MQI命令，MQSC命令帮助
        - Java JRE，必须，Java运行时，支持MQ部分Java编写的功能
        - Message Catalogs，支持的语言
        - Global Security Kit，支持加密安全认证功能，必须先安装JRE
        - Telemetry Service（MQSeriesXRService），遥感通信MQTT协议，支持传感器等设备连接通信
        - MQ Explorer，管理监控MQ
        - Managed File Transfer，文件传输
        - Advanced Message Security，更高级的安全保护，必须安装JRE和Security Kit
        - AMQP Service，提供AMQP通道，支持MQ Light APIs
    - 配置用户环境
        - \# passwd mqm（rpm安装过程中已经创建了一个名为mqm的用户和组，设置密码来解锁使用）
        - /etc/profile修改环境变量，并source
            <br/> &emsp; MQ_HOME=/opt/mqm/bin
            <br/> &emsp; PATH=$MQ_HOME:$PATH
            <br/> &emsp; export PATH
        - su - mqm（切换用户）
        - 将mqm的主目录下，文件、目录修改为mqm用户名和用户组
- Windows（仅Explorer）
    - 下载到安装包压缩包，形如：ms0t_mqexplorer_9100_windows_x86_64.zip
    - 解压缩，双击setup.exe安装
## 配置
- 创建队列管理器：
<br/> &emsp; crtmqm -q YOUR_MQM_NAME
- 开启队列管理器：
<br/> &emsp; strmqm YOUR_MQM_NAME
- 查看队列管理器状态：
<br/> &emsp; dspmq
- 打开队列管理器（此步往下都在管理器中执行）：
<br/> &emsp; runmqsc YOUR_MQM_NAME
- 定义服务器连接通道：
<br/> &emsp; DEFINE CHANNEL(YOUR_CONN_NAME) CHLTYPE(SVRCONN) TRPTYPE(TCP) MCAUSER(YOUR_USER_NAME)
- 创建监听：
<br/> &emsp; DEFINE LISTENER(YOUR_LISTENER_NAME) TRPTYPE(TCP) PORT(1414) CONTROL(QMGR)
    - control属性代表启动、停止的控制方式：MANUAL为手动、QMGR为随QM启停、STARTONLY为随QM启动但不随其停止
- 启动监听：
<br/> &emsp; START LISTENER(YOUR_LISTNER_NAME)
- 发送&接收的队列&通道配置：暂时跳过
- 权限问题：
    - 取消验证：
        - 关闭通道鉴权：
        <br/> &emsp; ALTER QMGR CHLAUTH(DISABLED)
        - 禁用连接权限认证：
        <br/> &emsp; ALTER QMGR CONNAUTH('')
        - 刷新安全策略：
        <br/> &emsp; REFRESH SECURITY TYPE(CONNAUTH)
    - 正常验证：
        - 关闭对mqm的禁用
        <br/> &emsp; SET CHLAUTH(*) TYPE(BLOCKUSER) USERLIST(*MQADMIN) ACTION(REMOVE)
        - 从windows等系统访问时会有windows系统用户名称绑定的CLNTUSER，根据自己的情况添加
        <br/> &emsp; SET CHLAUTH(*) TYPE(USERMAP) CLNTUSER('liyicheng') USERSRC(MAP) MCAUSER('mqm') ACTION(ADD)
        - 或者允许从一个指定ip访问时
        <br/> &emsp; SET CHLAUTH(*) TYPE(ADDRESSMAP) ADDRESS(10.8.203.215) USERSRC(MAP) MCAUSER('mqm') ACTION(ADD)
- 结束：end
> 注意：创建队列和通道时，定义名字的或者绑定名字的时候不加‘’的是默认大写的，加上‘’是区分大小写的，这是一个坑。

## IBMMQ基本使用
- 主要概念
    - 队列
        - 本地队列：物理上位于本地队列管理器中的队列
        - 本地传输队列：临时存储目标为远程管理器的消息的队列。队列管理器利用传输队列，分阶段的将消息发送到远程队列。一个本地传输队列是位于网络两侧的两个队列管理器的连接的桥梁。
        - 远程队列：一个不与该应用程序直接相连的队列管理器下的队列。(发送方的远程队列就是接收方的本地队列)
        - [其他队列分类](https://www.cnblogs.com/ealenxie/p/9244776.html)
    - 通道
        - 发送通道：将消息发送到远程队列（给出传输队列、远程队列、远程队列管理器的信息）
        - 接收通道：获取远程发到本地的消息
        - 服务器连接通道：用于客户端和服务器连接
    - QM：队列管理器
    - 队列深度：当前队列中的消息数量
    - 内网连接和外网连接：据说是有区别的，但是尚未接触到。
- 关键文件
    - 日志
        - /var/mqm/qmgrs/YOUR_MQM_NAME/errors/***.log
- 和Spring整合
    - 依赖
    ```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jms</artifactId>
    </dependency>
    <dependency>
        <groupId>javax.jms</groupId>
        <artifactId>javax.jms-api</artifactId>
        <version>2.0.1</version>
    </dependency>
    <dependency>
        <groupId>com.ibm.mq</groupId>
        <artifactId>com.ibm.mq.allclient</artifactId>
        <version>9.1.1.0</version>
    </dependency>
    ```
    - 配置
    ```yml
        #ibm mq 配置信息
        project.mq.host= 172.16.20.51
        project.mq.port= 1414
        #(队列管理器名称)
        project.mq.queue-manager= MQTEST
        #(通道名称)
        project.mq.channel= SERVERCONN
        #创建的MQ用户
        project.mq.username= mqm #填上你的用户名
        #创建的MQ用户连接密码
        project.mq.password= 123456 #如果关闭了验证，这里随意
        #连接超时
        project.mq.receive-timeout= 20000
    ```
    - 配置类代码
    ```java
    package com.liyicheng.springlearn.config;

    import com.ibm.mq.jms.MQQueueConnectionFactory;
    import com.ibm.msg.client.wmq.WMQConstants;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Primary;
    import org.springframework.jms.connection.CachingConnectionFactory;
    import org.springframework.jms.connection.JmsTransactionManager;
    import org.springframework.jms.connection.UserCredentialsConnectionFactoryAdapter;
    import org.springframework.jms.core.JmsOperations;
    import org.springframework.jms.core.JmsTemplate;
    import org.springframework.transaction.PlatformTransactionManager;

    /**
    * @author liyicheng
    * @date 2021-07-28 17:52:57
    **/
    @Configuration
    public class JmsConfig {
        /**
        * 注入连接参数:
        * 建立JmsConfig类，添加注解@Configuration，并将以上属性注入到此类
        */
        @Value("${project.mq.host}")
        private String host;
        @Value("${project.mq.port}")
        private Integer port;
        @Value("${project.mq.queue-manager}")
        private String queueManager;
        @Value("${project.mq.channel}")
        private String channel;
        @Value("${project.mq.username}")
        private String username;
        @Value("${project.mq.password}")
        private String password;
        @Value("${project.mq.receive-timeout}")
        private long receiveTimeout;

        /**
        * 配置连接工厂:
        * CCSID要与连接到的队列管理器一致，Windows下默认为1381，
        * Linux下默认为1208。1208表示UTF-8字符集，建议把队列管理器的CCSID改为1208
        *
        * @return
        */
        @Bean
        public MQQueueConnectionFactory mqQueueConnectionFactory() {
            MQQueueConnectionFactory mqQueueConnectionFactory = new MQQueueConnectionFactory();
            mqQueueConnectionFactory.setHostName(host);
            try {
                mqQueueConnectionFactory.setTransportType(WMQConstants.WMQ_CM_CLIENT);
                mqQueueConnectionFactory.setCCSID(1208);
                mqQueueConnectionFactory.setChannel(channel);
                mqQueueConnectionFactory.setPort(port);
                mqQueueConnectionFactory.setQueueManager(queueManager);
            } catch (Exception e) {
                e.printStackTrace();
            }
            return mqQueueConnectionFactory;
        }

        /**
        * 配置连接认证:
        * 如不需要账户密码链接可以跳过此步，直接将mqQueueConnectionFactory注入下一步的缓存连接工厂。
        *
        * @param mqQueueConnectionFactory
        * @return
        */
        @Bean
        UserCredentialsConnectionFactoryAdapter userCredentialsConnectionFactoryAdapter(MQQueueConnectionFactory mqQueueConnectionFactory) {
            UserCredentialsConnectionFactoryAdapter userCredentialsConnectionFactoryAdapter = new UserCredentialsConnectionFactoryAdapter();
            userCredentialsConnectionFactoryAdapter.setUsername(username);
            userCredentialsConnectionFactoryAdapter.setPassword(password);
            userCredentialsConnectionFactoryAdapter.setTargetConnectionFactory(mqQueueConnectionFactory);
            return userCredentialsConnectionFactoryAdapter;
        }

        /**
        * 配置缓存连接工厂:
        * 不配置该类则每次与MQ交互都需要重新创建连接，大幅降低速度。
        */
        @Bean
        @Primary
        public CachingConnectionFactory cachingConnectionFactory(UserCredentialsConnectionFactoryAdapter userCredentialsConnectionFactoryAdapter) {
            CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory();
            cachingConnectionFactory.setTargetConnectionFactory(userCredentialsConnectionFactoryAdapter);
            cachingConnectionFactory.setSessionCacheSize(500);
            cachingConnectionFactory.setReconnectOnException(true);
            return cachingConnectionFactory;
        }

        /**
        * 配置事务管理器:
        * 不使用事务可以跳过该步骤。
        * 如需使用事务，可添加注解@EnableTransactionManagement到程序入口类中，事务的具体用法可参考Spring Trasaction。
        *
        * @param cachingConnectionFactory
        * @return
        */
        @Bean
        public PlatformTransactionManager jmsTransactionManager(CachingConnectionFactory cachingConnectionFactory) {
            JmsTransactionManager jmsTransactionManager = new JmsTransactionManager();
            jmsTransactionManager.setConnectionFactory(cachingConnectionFactory);
            return jmsTransactionManager;
        }

        /**
        * 配置JMS模板:
        * JmsOperations为JmsTemplate的实现接口。
        * 重要：不设置setReceiveTimeout时，当队列为空，从队列中取出消息的方法将会一直挂起直到队列内有消息
        *
        * @param cachingConnectionFactory
        * @return
        */
        @Bean
        public JmsOperations jmsOperations(CachingConnectionFactory cachingConnectionFactory) {
            JmsTemplate jmsTemplate = new JmsTemplate(cachingConnectionFactory);
            jmsTemplate.setReceiveTimeout(receiveTimeout);
            return jmsTemplate;
        }
    }
    ```
    - 生产者代码
    ```java
    package com.liyicheng.springlearn.producer;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.jms.core.JmsOperations;
    import org.springframework.stereotype.Component;

    import javax.annotation.PostConstruct;

    /**
    * @author liyicheng
    * @date 2021-07-28 17:54:09
    **/
    @Component
    public class SendMessage {

        @Autowired
        JmsOperations jmsOperations;

        //@PostConstruct在服务器加载Servle的时候运行，并且只会被服务器执行一次, @PreDestroy在destroy()方法执行执行之后执行
        @PostConstruct
        public void send(){
            jmsOperations.convertAndSend("Q1", "my message...");
            System.out.println("开始发送消息");
        }

    }
    ```
    - 消费者代码
    ```java
    package com.liyicheng.springlearn.consumer;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.jms.annotation.JmsListener;
    import org.springframework.jms.core.JmsOperations;
    import org.springframework.jms.listener.adapter.MessageListenerAdapter;
    import org.springframework.stereotype.Component;

    import javax.jms.JMSException;
    import javax.jms.Message;
    import javax.jms.TextMessage;

    /**
    * @author liyicheng
    * @date 2021-07-28 17:54:40
    **/
    @Component
    public class ReceiveMessage  extends MessageListenerAdapter {
        @Autowired
        JmsOperations jmsOperations;


        @Override
        @JmsListener(destination = "Q1") //队列
        public void onMessage(Message message) {
            //必须转换如果不转换直接message.tostring消息的传输有限制。
            TextMessage textMessage= (TextMessage) message;  //转换成文本消息
            try {
                System.out.println("MQ_send传来的值为:" +textMessage.getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }

        }

    }
    ```
    
## 参考资料
- [在Linux系统安装IBM WebSphere MQ](https://blog.csdn.net/chengyan0079/article/details/78922889)
- [Linux安装IBM WebSphere MQ以及配置](https://blog.csdn.net/weixin_37539417/article/details/93488229)
- [IBM WebSphere MQ for linux 安装详解](http://www.shterm.cn/176.html)
- [IBM MQ 远程队列的创建与使用](https://blog.csdn.net/ILYPTING/article/details/104749065)
- [SpringBoot整合IBMMQ连接发送和接收](https://blog.csdn.net/u012448904/article/details/90474548  )
- [IBMMQ搭建和远程访问队列管理器时的权限配置](https://www.cnblogs.com/shawWey/p/12204917.html)