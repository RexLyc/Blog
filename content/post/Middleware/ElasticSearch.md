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
    // 
    "_version": 1,
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

下面用Console的语法风格，介绍一些基本的语法，注意版本8.11.3。ES中主要有URL检索、DSL（领域特定语言）检索（其实就是带上一个描述查询需求的JSON结构体）
```kibana
// 创建索引

// 插入文档
// 更新文档
// 更新文档时执行脚本
// 删除文档

// 冻结索引
// 删除索引

// 按字段搜索

// 批量操作

```

接下来还有一些复杂的搜索
```kibana

```

## 核心原理




## 参考资料
《ElasticSearch 实战》