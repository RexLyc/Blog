---
title: "《Hadoop权威指南（第四版）》读书笔记"
date: 2024-01-22T19:33:41+08:00
categories:
- 读书笔记
- 技术书
tags:
- Hadoop
- 中间件
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/hadoop-logo.jpg
draft: true
---
在大数据处理领域，Hadoop生态凭借多年的运营，基本上已经成为相关实现的事实标准。本文记录为读书笔记。
<!--more-->

> 学习过程中，主要使用docker封装的hadoop 3.2.1版本。本地调试时，使用Windows10、Hadoop 3.3.0.、winutil 3.3.0。

## 准备工作
使用docker compose搭建：参考[Hadoop]({{<relref "/content/post/Tools/wsl.md#Hadoop">}})

## 写在前面
Hadoop是由Apache Lucene创始人Doug Cutting创建的。Doug Cutting一直致力于制作强大的搜索系统。他发起了Lucene（1999）、Nutch（2004）、Hadoop（2006）等项目。在发展过程中，Google的三篇论文（GFS、MapReduce、Big Table）起到了决定性的作用，Hadoop可以视为是三篇论文的最佳开源实现。

书中提到的[SETI@home](https://setiathome.berkeley.edu/)。是一个志愿计算项目，鼓励志愿者将自己的CPU的空闲时间利用起来，计算SETI项目分发的天文数据。这些数据往往需要进行几小时乃至几天的运算。和Hadoop的目标场景在多种维度上都有显著区别。

书中提到的一些示例程序，需要有相应的数据。可以从以下参考资料中获得。
- [提取并整理 NCDC 气象数据](https://zhuanlan.zhihu.com/p/556150264)。在使用ftp匿名登陆时，用户名为anonymous，密码输个邮箱就行。注意需要特殊网络连接（你懂的）。
    ```bash
    # 下载2023年全部数据
    wget -r -nH ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-lite/2023 --ftp-user=anonymous --ftp-password=你的邮箱
    ```
> Hadoop作为大数据处理生态解决方案。其生态中的各个组件也并不是简单接入就能适用于所有业务场景。在技术选型时，要充分对比各种方案。

## HDFS

## Yarn

## I/O操作

## MapReduce
最经典的计算模型，官网的一段例子可以很好的入门理解。
```java
// 最经典的分词统计算法
public class WordCount {

  // Map和Reduce的含义和函数式编程类似
  // Map阶段，产生新的值，将每一个词映射为数字1
  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        // 输出为下一阶段的key和value
        context.write(word, one);
      }
    }
  }

  // Reduce阶段，若干个值生成一个值
  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    // 上一阶段的Key会合并到一个，遍历其所有value
    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);

    // 默认就是文本输入形式（读取输入文件夹的所有文件）
    // job.setInputFormat(TextInputFormat.class);
    // job.setOutputFormat(TextOutputFormat.class);

    // 三个阶段：Map、Combine、Reduce。相邻的两个阶段的输入输出要匹配
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);  // 使用combine需要满足结合律
    job.setReducerClass(IntSumReducer.class);
    // 设置最终的输出的key、value类型，必须和最终的Reduce的输出类型匹配
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    // 设置（多个）输入，设置输出
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```




## 生态
### HBase和Hive

### ZooKeeper

### Spark

### Flink

### Avro

### Flume

### Sqoop

### Pig

### Solr

### 其他


## 一些坑：
1. Hadoop执行程序是需要打包程序，上传到集群上执行的，因此对于MapReduce程序，如果想要调试的话，是不能无配置就直接在本机调试（没有那些Jar）。但本地调试还是很重要的，参考[Windows 10安装Hadoop 3.3.0教程](https://kontext.tech/article/634/bwz7kdmrgv0)。其中关于四个xml文件的配置如下
        ```xml
        <!-- core-site.xml -->
        <configuration>
            <property>
                <name>fs.default.name</name>
                <value>hdfs://0.0.0.0:19000</value>
            </property>
        </configuration>
        <!-- hdfs-site.xml -->
        <configuration>
            <property>
                <name>dfs.replication</name>
                <value>1</value>
            </property>
            <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:///e:/Hadoop/hadoop-3.3.0/data/namenode</value>
            </property>
            <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:///e:/Hadoop/hadoop-3.3.0/data/data</value>
            </property>
        </configuration>
        <!-- yarn-site.xml -->
        <configuration>
            <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
            </property>
            <property>
                <name>yarn.nodemanager.env-whitelist</name>
                <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
            </property>
        </configuration>
        <!-- mapred-site.xml -->
        <configuration>
            <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
            </property>
            <property> 
                <name>mapreduce.application.classpath</name>
                <value>%HADOOP_HOME%/share/hadoop/mapreduce/*,%HADOOP_HOME%/share/hadoop/mapreduce/lib/*,%HADOOP_HOME%/share/hadoop/common/*,%HADOOP_HOME%/share/hadoop/common/lib/*,%HADOOP_HOME%/share/hadoop/yarn/*,%HADOOP_HOME%/share/hadoop/yarn/lib/*,%HADOOP_HOME%/share/hadoop/hdfs/*,%HADOOP_HOME%/share/hadoop/hdfs/lib/*</value>
            </property>
        </configuration>
        ```
    - [winutil](https://github.com/cdarlint/winutils)必须复制到Hadoop的bin目录下
    - 只是想在IDEA中调试运行的话，**不需要真正启动Hadoop**。配置环境变量（HADOOP_HOME、Path中添加bin、sbin）即可。main函数中连配置都**不需要改**。
    - 3.3.0以前会出现文件操作权限问题，参考[Install Hadoop 3.2.1 on Windows 10 Step by Step Guide](https://kontext.tech/article/377/latest-hadoop-321-installation-on-windows-10-step-by-step-guide)
    - 如果真的想要在Windows上运行Hadoop，需要以管理员身份运行start-all.cmd
2. Hadoop的各类依赖在Maven中可能会产生依赖冲突，需要通过插件进行exclude解决。
    - 但有些只是运行时警告，无所谓。

## 参考资料
《Hadoop权威指南 第四版（修订版&升级版》

