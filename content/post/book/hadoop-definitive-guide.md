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
当数据的大小超过一台独立的物理计算机的存储能力时，就有必要对数据进行分区，并分布存储到网络内的多个计算机上。HDFS就是Hadoop生态中，负责完成这一任务的子系统。全称是Hadoop Distributed File System，有时也称为DFS。

当然实际上Hadoop的文件系统概念是一个抽象概念，只需要实现了FileSystem类的都可以作为其文件系统进行使用。除了HDFS外，还有LocalFileSystem、WebHdfsFileSystem、SWebHdfsFileSystem、HarFileSystem、ViewFileSystem、FTPFileSystem等等。因此虽然主要面向HDFS，但是仍然应当主要面向FileSystem、FileContext这些抽象类/接口进行编程。

HDFS是为专门的场景而设计的，即需要以流式的访问模式，来存取超大文件，并能运行于（相对）廉价的商用硬件集群之上。具体来说，有以下几个特点。
1. 目标是支持较多的超大文件，而不是海量的小文件
2. 流式数据访问：一次写入、多次读取，会在数据上进行长时间的分析。不要求低时间延迟
3. 不要求昂贵的高可靠设备，可以在廉价设备上运行。允许设备故障
4. 对多用户，文件任意位置修改等常规的文件系统的功能支持有限（甚至是不支持）

基础指令示例
```bash
# 查看HDFS根目录下的所有文件的块信息
hdfs fsck / -files -blocks

# dfs的参数是常规指令
hdfs dfs -ls
hdfs dfs -cat ./output3/part-00000

# 在namenode节点上，将本地磁盘文件/tmp/temp.txt，写入到hdfs（实际上是namenode服务，端口9000）的/user/root/路径下
hadoop fs -copyFromLocal /tmp/temp.txt hdfs://localhost:9000/user/root/temp.txt
# 从HDFS拷贝回本地路径（实际上，在正确配置了core-site.xml之后，是可以省略hdfs://localhost:9000的）
hadoop fs -copyToLocal /user/root/output3/part-00000 /tmp

hadoop fs -mkdir /user/root/test-dir
hadoop fs -ls /

```

HDFS的核心设计有以下几种概念：
1. 块：和传统文件系统采用文件为单位存储不同。HDFS采用块来对文件系统进行抽象，块是HDFS对数据进行备份和传输的基本单位。文件的元数据也并不需要和块一同存储。一个块默认128MB。在一个块中可能存储多个文件（多个小文件），也可能由多个块共同构成一个大文件。在实现中，还有如下要点
   1. 块缓存：对于经常访问的文件，所在的块可被显式的缓存在datanode的内存中，而且是以堆外块缓存的形式存放（off-heap block cache，在JVM管理范围之外的本地内存）。
    > 块的默认大小，也是一个值得思考和优化的数值。过大的块大小会使得参与Map的节点变少（因为有数据的节点变少了），但是过小的块大小又会使对块的寻址时间变长。
2. 两类节点（计算机）：namenode、datanode
   1. 管理节点namenode：以命名空间镜像文件和编辑日志文件，存储HDFS文件系统树，直接保存在本地磁盘。另外也会记录每个文件的各个块所在的数据节点的信息（非永久，由datanode上报）。
   3. 数据节点datanode：HDFS真正存储数据的工作节点，存取数据块，定期向namenode发送自己目前维护的块信息。
3. 联邦HDFS：由于namenode管理全部文件的块信息，其很容易受到单机内存的限制。因此HDFS也引入了对namenode的扩展，允许不同的namenode分别管理文件系统树的一部分（比如不同的namenode管理不同的目录）。在这种情况下，因为namenode之间相互独立（不通信），而且datanode上的块是通用的（可能存储任意HDFS路径下的文件），所以datanode需要注册到每一个namenode，以使自己所维护的块信息准确存储到对应的namenode上。
4. namenode的容灾方案示例
   1. 方法一：写本地磁盘的同时，同步原子地写入一个远程网络文件系统NFS
   2. 方法二：运行一个辅助namenode。具体可以用定期备份、热备等方式。

**高可用HA**，虽然前文提到了namenode的容灾方案。但是恢复的过程仍然会非常漫长。包括且不限于：将文件系统树加载到内存，重放编辑日志，接收到足够多的datanode的注册。这个过程是冷启动，随着块数的增加而增加。Hadoop2开始，针对该问题引入了活动-备用（active-standby）namenode，在活动namenode失效时，备用会立刻接管。甚至可以根据需要，配置若干个namenode，并由选举产生活动namenode。在这种方案下，活动和备用切换时不会有明显的中断。

HDFS的接口，虽然HDFS提供了HTTP、C等接口，但主要使用应该还是在Java中。值得注意的是，并不是所有文件系统都能支持所有操作。比如文件内容追加```FileSystem.append()```，就不是所有文件系统都支持的。这一点在调用FileSystem的API时会受到异常。下面是几种使用方式。
```java
// 一个简单的办法是使用URL进行访问，但这需要修改URL的工厂
// 通过设置URL的工厂方法，支持对Hadoop文件系统的URL协议
URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());
try{
    // 访问Windows本地文件系统
    // in = new URL("file:///E:/Hadoop/hadoop-3.3.0/data/input/t1.txt").openStream();
    // 访问HDFS
    in = new URL("hdfs://user/root/output3/part-00000").openStream();
    byte[] data= new byte[1024];
    in.read(data);
    String a=new String(data);
    System.out.println("data："+a);

} catch (Exception e){
    e.printStackTrace();
} finally {
    IOUtils.closeStream(in);
}


// 一个更通用的办法是使用FileSystem的API
// fs在创建时可以通过配置来获取当前的文件系统（需配置core-slite.xml等）
FileSystem fs = FileSystem.get(job);
OutputStream out = fs.create(new Path("E:/Hadoop/hadoop-3.3.0/data/input/t3.txt"));
try{
    // 如当前配置了Windows本地文件系统，输入Windows本地路径
    in = fs.open(new Path("E:/Hadoop/hadoop-3.3.0/data/input/t1.txt"));
    // 将流式数据输出到标准输出
    IOUtils.copyBytes(in,System.out,1024,false);
    ((FSDataInputStream)in).seek(0);
    IOUtils.copyBytes(in,out,1024,false);
    ((FSDataInputStream)in).seek(0);
    IOUtils.copyBytes(in,out,1024,false);
} catch (Exception e){
    e.printStackTrace();
} finally {
    // 
    IOUtils.closeStream(out);
}
```

一次对HDFS的读取过程主要有6个步骤，其中最重要的流程就是从namenode读取到所需文件的块信息（块信息存在内存中），并进一步去对应datanode上读取数据，不同的块可以并发去不同的datanode读取。这些过程都是用RPC方式完成的。
![hdfs读取文件过程](/images/book/hadoop/hdfs-data-access.png)

一次对HDFS的写入过程则稍微复杂一些，重点是需要在namenode上创建块信息（要先选择一组最适合的datanode），并在写入完成后更新完善块信息。HDFS客户端只负责将块数据流式的传输给第一个datanode，后续datanode将会链式传递数据（并链式确认）以保证将数据发到所有需要存储块副本的datanode。
![hdfs写入文件过程](/images/book/hadoop/hdfs-data-write.png)

**数据一致性**，作为一个分布式系统，HDFS当然也需要考虑数据的一致性模型。为了性能，HDFS牺牲了强一致性。即当前块正在写入内容不保证立即可见，当前块写入完成后对新的读取可见。

如果对一致性有要求，可以手动调用以下的FileSystem接口。
1. ```hflush```，立刻刷新写入数据到所有的datanode的写入管道，对新的读取可见。但不保证datanode写磁盘完成。掉电丢失数据。
2. ```hsync```，确保写入磁盘完成。


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

原理概述
1. Hadoop将一次计算的完整执行，称作Job（作业），它包括了输入数据、MapReduce程序，以及各种配置信息。运行期，Hadoop会将Job分为若干个任务Task来执行，具体就是各类Map任务和Reduce任务。它们将会分布在不同的节点上，由Yarn进行调度执行。
1. 对于海量的输入数据，Hadoop会将其切分为登场的小数据块，称为输入分片（input split），在每个分片上构建Map任务。分片的大小划分是一个优化要点，过大的分片可能会造成计算负载的不均衡，过小的分片又会造成管理分片和构建Map任务的时间增加。
1. 数据本地化优化是MapReduce最重要的优化之一。也就是说Hadoop应该在由输入数据的HDFS节点上运行对应的Map任务，而不是使用宝贵的网络通信资源。即使发生了必须在无输入数据的节点上运行程序的情况（比如有数据节点任务排满），也应当按照优先同一机架、同一机房等顺序，就近处理。
1. MapReduce的每一步都会落盘，但Map任务产出的一般将会只写入到本地硬盘，而非HDFS，这是因为Map产生的一般都是中间结果，而中间结果在最终会被删除，没有必要做分布式存储。如果该节点在返回计算结果前失败，Yarn调度其他节点重新计算。
2. reduce输出的最终结果会落HDFS，其第一个副本就存储在reduce任务所在的HDFS节点中。
3. 多个reduce任务时，map到reduce的数据传递称为混洗shuffle。在这个阶段，每一个map输出都会按键切分为若干分（和reduce节点数量一致），并分给各个reduce节点。
    ![多map多reduce](/images/book/hadoop/map-reduce.png)
5. 对于无reduce的任务，各个map完全并行，而且输出数据也直接落hdfs。
6. combiner是在map任务之后的一个优化任务，目的是削减map任务节点和reduce任务节点之间数据传输量。它的思路也很好理解，例如对于求极值的问题，在map之后，可以立刻在原节点内，对所有具有相同的键的键值对求一次极值，此后对于一个键，只需要给reduce节点传递一个值。而不是坐等把所有键值都传给reduce去处理。

MapReduce也可以通过Hadoop Streaming的方式进行实现。尤其是对于一些轻量级的MapReduce开发需求，使用Streaming可能更简单。Streaming的原理说到底，就是用标准输入输出作为数据传递的管线，代替原先在Java中设置的Map和Reduce之间的数据传递，由hadoop来完成对整个流程的控制。主要的有点就是**不限语言**，只要它能处理标准输入输出就行。下面是一段Streaming的最简单的例子。
```bash
# 提交streaming任务，mapper用cat，reducer用wc，最终将会对所有文件的数据进行wc
mapred streaming -input input -output output3 -mapper /bin/cat -reducer /usr/bin/wc
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

