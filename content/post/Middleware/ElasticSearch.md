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
- 全文检索（Full-Text Retrieval）：扫描文章中的每一个词，为其建立索引，说明出现的次数和位置。只处理文本、结果有相关度排序。
- 接近实时性：搜索时长通常在数秒内
- 索引：一个索引是一些有相似特征的文档的集合。每个索引有一个唯一的表示名称。ES默认根据插入数据，自动创建类型、映射
- 类型：一个索引中可以有一个类型。定义了一种语义，代表一类数据。
- 映射：定义索引中的类型的数据结构。映射中包括字段名、字段数据类型、字段索引类型。
- 文档：可被索引的基础信息单元，采用JSON表示。

```json
// 索引文件示例
```


## 搭建和基础使用
参考[ElasticSearch&Kibana]({{<relref "/content/post/Tools/wsl.md#elasticsearch">}})

使用方式
1. 在编程语言中使用相关接口
2. 使用Kibana提供的Console，类似cURL的语法，相对更简洁明了。实际上将cURL指令拷贝到Console中，会自动转换过来
    ```kibana
    # 官方例子，注意拷贝时，行尾不要有用来取消换行的\
    curl -X POST "${ES_URL}/search-lyclearn/_search?pretty" -d'
    {
        "query": {
            "query_string": {
            "query": "snow"
            }
        }
    }'
    ```