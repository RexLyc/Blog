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
    "from":1
}

```





## 核心原理
### 基础结构和流程
1. 文档：以JSON表示的，自包含（同时包含字段和取值）、层次型、结构灵活（可以缺少/多余一些字段）。和关系型数据库不同的是，文档并不是模式化的，ES的文档是无模式的。并不要求文档都有相同的字段，受限于同一种模式。
    > 不要求模式化，并不代表不要求字段类型一致。对于设定的取值应为整型的字段，传递字符串也是非法的。
1. 映射：通过设定，或者自动解析的方式，将一个文档进行逻辑上的划分，分为不同的字段。
2. 索引：索引是映射的容器，一个索引存储在磁盘的同一组文件中，索引存储了映射的所有字段，以及一些索引级别的设置。
3. 分布式数据结构：
   1. 索引分片：为了提高处理速度，在多节点的ES集群中，索引会被分片，不同的节点处理一个索引的一部分。索引分片是ES将数据进行节点间移动的最小单位。分为主分片和分片副本。
   2. 分片副本：对于每个分片而言，为了提高可靠性，会在不同节点存储一些副本。
4. 索引（Indexing）一篇文档：文档被发送到一个随机的主分片，并由该主分片负责对这次的文档进行索引操作。并对该主分片和分片副本进行数据同步。
5. 搜索索引：在索引的完整分片集合中进行搜索，集合中的分片可以是主分片也可以是副本。因此副本不仅作为容错，也提供负载均衡能力。

<!-- TODO：插入流程图示意 -->




## 参考资料
《ElasticSearch 实战》

[官方文档ElasticSearch Current Reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro.html)

[Kibana用户指南Console](https://segmentfault.com/a/1190000016391736)