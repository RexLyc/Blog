---
title: "ElasticSearch：合集篇"
date: 2024-01-09T20:59:38+08:00
categories:
- 计算机科学与技术
- 数据中间件
tags:
- 数据中间件
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/logo-elasticsearch-32-color.png
draft: true
math: true
---
采用Java语言编写的一个开源搜索引擎，具有良好的性能，灵活的数据存储格式，丰富的搜索功能。
<!--more-->

> ElasticSearch经典书籍大多较陈旧，注意学习新版本特性。

## 概述
特性
- 使用JSON作为数据存储格式
- 支持地理位置查询，文本混合查询
- 统计、日志数据、可视化功能均非常优秀

示例项目：Wikipedia、StackOverflow、Github均在搜索中使用了ES

ELK套件：
- ElasticSearch（数据库）：存储、搜索
- Logstash（服务器端数据处理管道）：收集、解析、处理
- Kibana（数据分析和可视化平台）：前端，搜索、查看、交互、存放数据

基本概念
- 通用概念
  - 正排索引：按文档ID-关键字列表，组织的索引。当搜索一个关键字时，需要遍历每一个文档的关键字列表
  - 倒排索引：按关键字-所在文档ID，组织的索引。搜索关键字能立刻获得所在文档
  - 文档相关性得分：在搜索过程中，根据搜索关键字和索引数据计算的文档相关性分数，用于确定搜索结果的优先级
  - TF-IDF（词频-逆文档频率）算法：
    - 词频：一个关键字在当前文档词频越高得分越高
    - 逆文档频率：多个关键字中，在所有文档中出现频率**低**的，该关键字在所有关键字中的权重**更高**。往往意味着这个词不是语气词、助词。
- ES相关
  - Lucene：可以说是ElasticSearch的前身和底层，一个文档搜索引擎。
  - 全文检索（Full-Text Retrieval）：扫描文章中的每一个词，为其建立索引，说明出现的次数和位置。只处理文本、结果有相关度排序。
  - 接近实时性：搜索时长通常在数秒内
  - 索引（Indice）：一个索引是一些有相似特征的文档的集合。每个索引有一个唯一的表示名称。ES默认根据插入数据，自动创建类型、映射。
    - 处理数据，创建倒排索引的步骤也叫索引（Indexing）
  - 映射：定义索引中的类型的数据结构。映射中包括字段名、字段数据类型、字段索引类型。
    - 动态映射：由ES自行分析数据，并创建映射中的各个字段
    - 静态映射：人工创建
  - 字段类型：ES支持的字段类型很丰富，字符串、数字、日期、布尔、二进制、范围数据、数组、对象、嵌套、地理数据、其他专用数据
  - 文档：可被索引的基础信息单元，采用JSON表示。是一条完整的数据。
  - 废弃
    - 类型：原本位于索引-文档中间的抽象层，意味着不同语义类型的文档。容易误用，被删。V8之后即使使用，类型名也必须是_doc，且只能有一个。

  > 不要用关系型数据库的思维去套用到ES中

```json
// 基于ElasticSearch 8.11.3
// 索引文件示例
{
  // 索引名称
  "my-simple-index": {
    "aliases": {},
    // 映射文件，原始数据的各个字段类型，及其字段属性（类型等）
    "mappings": {
      "properties": {
        "id": {
          "type": "long"
        },
        "name": {
          "type": "text"
        }
      }
    },
    // 索引配置
    "settings": {
      "index": {
        "routing": {
          "allocation": {
            "include": {
              "_tier_preference": "data_content"
            }
          }
        },
        "number_of_shards": "1",
        "provided_name": "my-simple-index",
        "creation_date": "1704893459772",
        "number_of_replicas": "1",
        "uuid": "6Ltq7b8kRnu7XpzaE9p_NA",
        "version": {
          "created": "8500003"
        }
      }
    }
  }
}

// 文档的相关元数据
{
    // 所属索引
    "_index":"",
    // 所属类型，基本就是_doc了
    "_type":"",
    // 文档ID，插入时可以指定，或者由ES生成
    "_id":"",
    // 文档更新时的递增号
    "_version": 1,
    // 序列号，用于确保索引不会查询到旧文档
    "_seq_no": 1,
    // 在查询时反馈，表示是否搜索到
    "found": true,
    "_source": {
        // 文档原始JSON数据
    },
    "references": [],
    "updated_at": "2023-xx-xx ..."
}
```

参考：[Elasticsearch索引与mapping映射](https://www.cnblogs.com/xfeiyun/p/15887524.html)

## 搭建
单机docker搭建：参考[ElasticSearch&Kibana]({{<relref "/content/post/Tools/wsl.md#elasticsearch">}})

此外，对于ElasticSearch，有一些插件值得关注。例如
1. 分词器：将输入的连续文本进行分词的模块，自带的标准分词器对于中文等语言的分词效果并不好（他会把所有的字单独拆开），通过为ES安装相关插件来解决。
  ```bash
  # 在elasticsearch安装目录下
  ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.11.3/elasticsearch-analysis-ik-8.11.3.zip
  ```
  对分词器使用和效果测试在后面会提到。


集群搭建：To Do

## 使用方式
接口调用方式
1. 在编程语言中使用相关接口
2. 使用Kibana提供的Console，类似cURL的语法，相对更简洁明了。实际上将cURL指令拷贝到Console中，会自动转换过来
    ```kibana
    # 官方例子，注意如果向Console拷贝时，行尾不要有用来取消换行的\
    curl -X POST "${ES_URL}/search-lyclearn/_search?pretty" -d'
    {
        "query": {
            "query_string": {
            "query": "snow"
            }
        }
    }'
    ```
3. 任何能够使用RESTful API的方式

下面用Console的语法风格，介绍一些在索引创建、映射创建、文档插入方面的基本的语法，注意版本8.11.3。。
```kibana
// 创建索引
PUT my-simple-index
{
    "mappings":{
        "properties":{
            "id":{
                "type":"long"
            },
            "name":{
                "type":"text",
                // 对分词器进行配置
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word"
            },
            "age":{
                "type": "double"
            }
        }
    }
}

// 对已存在索引，更新mapping，增加字段
PUT my-simple-index/_mapping
{
  "properties": {
      "money":{
        "type":"double"
      }
    }
}

// 查看索引信息
GET my-simple-index
// 查看映射信息
GET my-simple-index/_mapping

// 插入文档，索引/_doc/文档ID
PUT my-simple-index/_doc/1
{
  "id":234,
  "name":"liyicheng",
  "age":12
}

// 更新文档，仍然是PUT
PUT my-simple-index/_doc/1
{
  "id":666,
  "name":"liyicheng",
  "age":12
}

// 更新文档时执行脚本
POST my-simple-index/_update/1
{
  "script" : {
    "source": "ctx._source.id += params.count",
    "lang": "painless",
    "params" : {
      "count" : 4
    }
  }
}

// 删除文档
get my-simple-index/_doc/1

// 冻结索引
// 删除索引
DELETE my-simple-index

// 批量操作
// 批量更新（插入），注意格式必须满足一行一个，不能展开，否则会序列化错误
PUT my-simple-index/_bulk
{"index":{"_id":"1"}}
{"name":"John Doe","age":23,"bir":"2012-12-12"}
{"index":{"_id":"2"}}
{"name":"Jane Doe","age":24,"bir":"2012-12-12"}

// 也可以不提供index，由ES自行填充
PUT my-simple-index/_bulk
{"index":{}}
{"name":"rex lyc","age":266}

// 更新一个文档的同时，删除一个文档
POST my-simple-index/_bulk
{"update":{"_id":"1"}}
{"doc":{"age":20}}
{"delete":{"_id":"2"}}

// 测试分词器，注意需要安装插件
post _analyze
{
  "text":"测试分词器",
  "analyzer":"ik_smart"
}

```

相比较于插入，搜索的用法和功能都更多变一些。ES中主要有URL检索、DSL（领域特定语言）检索（推荐，其实就是带上一个描述查询需求的JSON结构体）。ES的查询操作有两大种类，查询（query）和过滤（filter），使用时应当先过滤（只筛选符合的，性能高），再进行匹配（会计算得分，慢）。而且ES会对过滤进行一定的缓存，进一步加快速度。
下面记录一些基础的搜索用法。
```kibana
// URL检索，按age升序
GET my-simple-index/_search?sort=age:asc

// 带有较多选项的查询
GET my-simple-index/_search
{
  // 查询条件
  "query": {
    // bool 表达式
    "bool": {
      // should代表 或||
      "should": [{
          // 区间查询
          "range": {
            "age": {
              "gte": 10,
              "lte": 2000
            }
          }
        },
        {
          // 单个字段，要求desc字段内容为测试
          "term": {
            "desc": {
              "value": "测试"
            }
          }
        }
      ]
    }
  },
  // 只返回每个文档的该字段
  // 满足查询条件，但没有该字段的记录也会被统计，这里只改变返回的展示
  "_source": ["desc"], 
  // 返回的排序
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ],
  // 每页的数量和当前页数
  "size":10,
  "from":1
}


// bool查询，和过滤器一同使用
GET my-simple-index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "desc": {
              "value": "测试"
            }
          }
        }
        ],
        // 过滤器内的语法和匹配是类似的
        "filter": [
          {
            "range": {
              "age": {
                "gte": 10,
                "lte": 2000
              }
            }
          }
        ]
    }
  },
  // 没有该字段的记录也会被统计，这里只改变返回的展示
  "_source": ["desc"], 
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ],
  "size":10,
  "from":1,
  // 可以使用explain来要求解释查询过程
  // 影响性能，应当仅用于调试
  "explain": true
}

```





## 核心原理
流程参考建议阅读博客，[ElasticSearch概述：索引和文档插入](https://www.cnblogs.com/xfeiyun/p/15887142.html)、[Elastic搜索流程](https://cloud.tencent.com/developer/article/1863678)。下面再简短总结。

### 基础结构和流程
1. 文档：以JSON表示的，自包含（同时包含字段和取值）、层次型、结构灵活（可以缺少/多余一些字段）。和关系型数据库不同的是，文档并不是模式化的，ES的文档是无模式的。并不要求文档都有相同的字段，受限于同一种模式。
    > 不要求模式化，并不代表不要求字段类型一致。对于设定的取值应为整型的字段，传递字符串也是非法的。
1. 映射：通过设定，或者自动解析的方式，将一个文档进行逻辑上的划分，分为不同的字段。
2. 索引：索引是映射的容器，一个索引存储在磁盘的同一组文件中，索引存储了映射的所有字段，以及一些索引级别的设置。倒排索引的核心数据组成主要有两部分，
    1. 单词词表：存储所有分词到倒排列表的映射关系。数据量比较大，一般采用B+树实现。
    2. 倒排列表：由倒排列表项组成，包含文档ID、词频、位置、偏移等字段。
3. 分布式数据结构：
   1. 分片：shard
   2. 索引分片：为了提高处理速度，在多节点的ES集群中，索引会被分片，不同的节点处理一个索引的一部分。索引分片是ES将数据进行节点间移动的最小单位。分为主分片和分片副本。
   3. 分片副本：对于每个分片而言，为了提高可靠性，会在不同节点存储一些副本。
4. 索引（Indexing）一篇文档：文档被发送到一个随机的主分片，并由该主分片负责对这次的文档进行索引操作。并对该主分片和分片副本进行数据同步。
   1. 索引前的具体步骤要跟复杂一些：预处理、索引存在及其处理、文档路由、主分片进行索引文档
   2. 索引后的写入阶段实际上也跟复杂一些（涉及到对内存、磁盘的控制）：Index Buffer、Transaction Log、Segment
5. 搜索索引：在索引的完整分片集合中进行搜索，集合中的分片可以是主分片也可以是副本。因此副本不仅作为容错，也提供负载均衡能力。接收到请求的节点作为协调者，开始协调一次搜索流程。
   1. 主要流程：路由和负载均衡、段级搜索、分片级搜索、协调者收集数据、最终处理

<!-- TODO：插入流程图示意 -->

### 分析器
当一个文档发送到ES的指定分片时，就开始了这篇文档的分析流程。一般来说分析器流程有三个子模块，在创建索引的时候也可以在```setting```中对这三个部分进行定制。
- 字符过滤器：纠正字符，替换一些符号，使之称为正确、标准的单词
- 标准分词器：输入的文本数据不会被直接使用，二十需要被处理为分词（在ES中也叫做token）。负责根据空格、换行、破折号等符号，将长文本分割为分词。
- 分词过滤器：分词过滤器在使用时会形成分词过滤器链，按顺序被调用。在这一步进行分词的小写化，**去掉**一些分词（比如去除一些介词连词、停用词）、**添加**一些分词（比如添加一些同义词）、进一步修改分词等类型的工作。
  
> 查询时有些查询方式也会执行分析步骤，有些则不会，因此需要注意鉴别。

经过了分词过滤器的分词，才会开始进行索引的构建。

一些内置的分析器：标准分析、简单分析、空白分析、停用词分析、关键词分析、模式分析、多语言分析

一些内置的分词器：标准分词、字母分词、关键字分词、模式分词、空白分词、电子邮件分词、路径分词

一些内置的分词过滤器：标准分词过滤、小写分词过滤、长度分词过滤、停用词分词过滤、截断修剪限制分词过滤、颠倒分词过滤、N元语法、词干提取等等

### 相关性搜索
在基础概念一节也提到过，相关性搜索是根据一个打分公式，对文档进行打分，并根据分数优先进行返回。最常用的公式就是TF-IDF公式，即词频-逆文档频率。

词频$TF$很好理解，就是一个分词在文档中出现的次数。实际使用时会进行归一化。

而逆文档频率$IDF$的计算，是用总文档数$|D|$除以分词出现的文档数$|\{j:t_i\in d_j\}|$，再取对数获得的：$log(\frac{|D|}{|\{j:t_i\in d_j\}|})$

总公式为

$score_{q,d}=\sum_{t}^{q}{\sqrt{TF_{t,d}} * IDF^2_{t,d} * norm(d,field) * boost(t)}$

解释一下公式，对于一个给定的查询q，和文档d。
1. 其得分是所有分词的得分总和。
2. 每个分词的得分由四部分相乘：文档内词频的平方根、逆文档频率的平方、该文档字段的归一化因子、该分词的提升权重（可在索引期、查询期配置）。


不过该模型只是ES最主流的评分模型，实际上ES还支持很多其他模型。用来支持对相关性要求迥异的不同场合。

另外由于搜索过程比较复杂，ES也提供了一些用于调试、解释搜索执行过程的功能。
- 方法一：在查询中添加```"explain":true```字段
- 方法二：调用```_explain```接口，如下
    ```kibana
    // 必须指明文档id
    POST my-simple-index/_explain/51T6AY0BC66AG24XNzzh
    {
      "query":{
        "match": {
          "name": "lyc"
        }
      }
    }
    // 返回值会解释是否匹配成功，以及相应原因、数据
    ```

其他查询优化技术
1. 查询再打分```rescore```：有些情况下，会使用一些昂贵的打分脚本来获取更好的匹配。但当文档数量很多的时候，全部执行打分是不现实的。此时可以使用```rescore```字段，来配置再打分步骤，只对部分数据再打分。
2. 定制得分```function_score```：对打分过程中的分词进行统一的定制得分。有一些内置的函数（权重、随机、衰减），也可以使用Groovy编写脚本。
3. 字段数据缓存：字段数据是指一个字段中的所有分词的集合，这个数据结构是用来补充倒排索引在某些算法中的不足。比如字段数据会应用在若干场景：比如按字段排序、聚集，用于定制得分等。字段数据会被尽可能缓存，以提高在后续查询中的速度。
4. 文档值doc_value：是在创建倒排索引的过程中，同时为部分字段创建的一个正排索引（按字段定义，列式存储各字段）。适合做排序和聚合操作。和字段数据缓存搭配使用。默认开启（但并不是每个字段都支持），也就是说ES对于原始数据的存储是有两份的（_source和doc_value）。

### 数据聚集

### 集群

### 

## 参考资料
《ElasticSearch 实战》

[ElasticSearch概述](https://www.cnblogs.com/xfeiyun/p/15887142.html)

[官方文档ElasticSearch Current Reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro.html)

[Kibana用户指南Console](https://segmentfault.com/a/1190000016391736)

[Elasticsearch索引、搜索流程及集群选举细节整理](https://cloud.tencent.com/developer/article/1863678)