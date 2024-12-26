---
title: "ElasticSearch：合集篇"
date: 2024-01-09T20:59:38+08:00
categories:
- 计算机科学与技术
- 中间件
tags:
- 中间件
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/logo-elasticsearch-32-color.png
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

下面用Console的语法风格，介绍一些在索引创建、映射创建、文档插入方面的基本的语法，注意版本8.11.3。
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

### 文档间关系
正如关系型数据库中数据表之间可以进行关联，ES这种文档数据库，也可以控制文档之间的关系，比如让文档之间的字段相互有关联，或者字段是嵌套的文档等等。

文档之间的关系通常有以下几种形式：
1. 对象类型：允许将一个对象作为文档字段的值。对象内再包含具体的多个字段。也可以形成一个对象数组。例如
    ```json
    // 提交的数据格式
    {
      "address":[
        {
          "city":"paris",
          "steet":"central"
        },
        {
          "city":"new york",
          "steet":"broadway"
        }
      ]
    }

    // 注意在实际的存储中，这段数据更倾向于被处理成
    {
      "address.city":["paris","new york"],
      "address.street":["central","broadway"]
    }
    ```
2. 嵌套文档：对象类型存在一个问题。如果一个字段是对象数组，那么在查询时，可能错误的匹配到了属于多个对象的不同字段值。此时如果将对象改为单独的文档，就可以避免这种问题。嵌套文档将每一个对象放到一个单独的文档中，所有的对象文档组成一个分块，存储在和主文档接近的位置。
    > 例如上面对象类型中的address字段，如果查询时选择城市为paris，且街道为broadway的，这篇文档是会被选中的。但逻辑上这样的匹配是错误的。
    > 这从一个侧面说明了，ES提供的基于分词的查询的算法并不是绝对智能的。他并不会判断一次匹配是否跨越了一个数组类型字段的多个对象边界。
    此时的解决方案是使用嵌套，只需要在设定映射时指定为nested即可，例如
    ```kibana
    PUT doc-relation
    {
      "mappings": {
        "properties": {
          "location":{
            // 注意是用来说明一个属性是嵌套文档
            "type": "nested", 
            "properties": {
              "city":{
                "type":"text"
              },
              "street":{
                "type":"text"
              }
            }
          },
          "title":{
            "type":"text"
          }
        }
      }
    }

    // 此时搜索也需要用nested指定路径
    GET doc-relation/_search
    {
      "query": {
        "nested": {
          "path": "location", 
          "query": {
            "bool": {
              "must": [
                {"match":{"location.city": "beijing"}},
                {"match":{"location.street": "chang an"}}
              ]
            }
          }
        }
      }
    }
    ```
    > 嵌套文档在配置映射时，也可以通过include_in_*字段，来保持允许将文档内的数据，同时索引到原始文档。也就是说可以同时使用嵌套查询，和普通查询。
3. 文档间的父子关系：对象、嵌套关系还是有一个限制。这些文档关系仍然过于紧密。当我们需要修改一部分文档内容时，往往需要修改全部相关的索引文件（比如倒排索引项中的出现次数、位置）。但如果使用完全不同类型的文档去存储数据。在映射中指定文档之间的父子关系。这样在处理文档数据变更时可以完全互相独立处理，无需考虑依赖问题。父文档中只存储父子关系，不存储子文档中的任何数据。父子关系的优缺点都很明显。应当只在有必要的时候进行使用：当索引数据包含一对多的关系，并且其中一个实体的数量远远超过另一个的时候。代码改自[一起学ES](https://cloud.tencent.com/developer/article/2372964?areaId=106001)。
    ```kibana
    // 父文档除了多一个父子关系字段，没什么特别的
    PUT doc-join-relation
    {
      "mappings": {
        "properties": {
          "title":{
            "type": "text"
          },
          "join_field":{
            // join代表父子关系
            "type": "join",
            "relations":{
              // 父文档中的映射名：子文档中的映射名
              "blogs":"comments"
            }
          }
        }
      }
    }

    // 插入也没什么特别的
    PUT doc-join-relation/_doc/1
    {
      "title": "Elasticsearch Join 示例",
      // 记录映射关系
      "join_field": "blogs"
    }

    // 子文档也是在同一个索引下，使用routing来保证到同一分片
    PUT doc-join-relation/_doc/2?routing=1
    {
      "title": "很棒的博客",
      "join_field": {
        "name": "comments",
        // 一个子文档字段只能有一个父文档
        "parent": "1"
      }
    }

    GET doc-join-relation/_search
    {
      "query": {
        // 用has_child来对子文档进行要求
        "has_child": {
          "type": "comments",
          "query": {
            "match_all": {}
          },
          // 随父文档一并返回子文档
          "inner_hits": {}
        }
      }
    }
    ```
    > 注意：使用了父子关系的文档，因为需要对完全不同的索引文件进行处理，显然会在搜索过程中有更差的效率。而且也需要考虑文件的路由问题。必须让父子文档位于一个节点。
5. 反规范化：将一些数据重复存储，避免在查询的时候要多次进行“连接”等操作。但缺点也很明显，数据冗余带来的性能和业务逻辑问题。
6. 应用端的连接：将需要进行连接的数据，分多次查询。并在客户端程序中，对返回数据再进行连接。

文档间关系不影响后续的各种分析、相关性搜索、数据聚集。这些功能都可以在不同的文档间关系上得到应有的实现。

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
所谓聚集，就是在一组搜索结果中，查询某一个统计数据。聚集功能是非常重要的功能。

在实现的过程中，聚集主要分为两个主要的类别：度量型和桶型，以及以此为基础的嵌套
- 度量型：对一组文档进行分析，获得最小值、最大值、方差等数据
- 桶型：用某个标准对文档分类，并统计每一个分类的文档数量。具体还分为单桶聚集（限定在一个桶内）和多桶聚集（桶数会随查询变化）
  - 多桶对性能有一定要求，也需要考虑运行时数据情况，或者限制最大桶数量
  - 单桶有多种：global（全局）、filter（满足过滤条件的）、missing（将所有未通过查询的进行聚集）
- 嵌套：在一个桶型聚集内部，可以再继续进行聚集操作

和倒排索引的步骤相反，数据聚集时往往需要用文档ID，获取到文档字段数据。因此在实际应用中其性能需要特别关注。

下面是一个对数据进行搜索并聚集的例子
```kibana
// 一个简单查询，并作嵌套聚集
POST my-simple-index2/_search
{
  "query": {
    // 允许模糊匹配的方法
    "wildcard": {
      "name": {
        "value": "*"
      }
    }
  }, 
  // aggregations的写法，8.11.3
  "aggs": {
    // 本次聚集的名称，用户自定义
    "test_aggs": {
      // 使用term按字段做桶型聚集，还可以在这里使用avg等进行度量型聚集
      "terms": {
        "field": "age",
        // 设置最少的数量要求
        "min_doc_count":2,
        // 可以设置排除某一个桶
        // 该字段类型根据聚集字段类型变化
        "exclude":[10]
      },
      // 嵌套，对每一个桶内，统计出生日期最大的
      "aggs": {
        "sub_aggs": {
          "max": {
            "field": "bir"
          }
        }
      }
    }
  }
}

```

### 集群
ElasticSearch需要面对海量的文档数据。分布式处理能力是其强大的根本。根据需要，ES可以很方便的扩展集群。关于集群的一些静态的配置在```elasticsearch.yml```中，另外也有一些和索引相关的配置，是在运行期可以修改的。

ES节点发现的方式有广播和单播，主要使用广播。ES作为一个分布式系统，实现了主节点选举、识别错误等功能。其中识别错误是指，主节点对所有节点发出请求，以检测其他节点是否能够正常工作。在实际使用中需要正确的对主节点数量、选举进行配置，避免发生脑裂。

对于集群的健康情况，可以通过```GET _cluster/health```查询得到。一个基本的JSON结果如下。
```json
{
  "cluster_name": "docker-cluster",
  // 因为存在未分配副本分片，健康状态为yellow
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  // 活跃主分片数量
  "active_primary_shards": 43,
  "active_shards": 43,
  "relocating_shards": 0,
  "initializing_shards": 0,
  // 未分配分片数量
  "unassigned_shards": 6,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 87.75510204081633
}
```
同一个文档的副本分片不能和主分片放置在一起，因此对于只有单节点的ES，他们实际上处于“不健康”的状态。当有新节点加入后，ES会尝试将分片在所有节点上进行均匀分配。

对于集群数量的修改，主要有三种：新增节点、删除节点、停用节点。删除节点的风险很高，而且很容易出现yellow“不健康”状态。更好的办法是停用一个节点。在停用节点流程中，ES将会停止分配任何分片到被停用的节点上。

集群健康状态为红色时，代表因为网络或节点故障，有一些数据永久丢失（暂时不能被查询到）。这时就需要恢复对应的节点和网络。

实际上用户主要需要考虑的是对于索引内的分片配置。例如对于如下配置。
```json
// 索引doc-relation
"doc-relation": {
  "settings": {
    "index": {
      "routing": {
        "allocation": {
          "include": {
            "_tier_preference": "data_content"
          }
        }
      },
      // 分片数量
      "number_of_shards": "1",
      // 基于可用节点的数量自动分配副本数量，默认为false（禁用）
      "auto_expand_replicas": "0-3",
      "provided_name": "doc-relation",
      "creation_date": "1705674175334",
      // 副本数量
      "number_of_replicas": "1",
      "uuid": "e5bD7WqsREOPa3YvUcbLAw",
      "version": {
        "created": "8500003"
      }
    }
  }
}
```

另外ES还提供了别名、路由两个功能。先说一下别名。别名是在索引之上的抽象，可以对多个索引绑定相同的别名，而且一个索引也可以绑定多个别名。当然也通过别名可以来查询、过滤文档。
```kibana
PUT doc-join-relation/_alias/myalias
PUT doc-relation/_alias/myalias

// 查看myalias别名的所有索引
GET _alias/myalias
// 查看doc-relation索引的全部别名
GET doc-relation/_alias

// 为别名配置别名过滤器
POST _aliases
{
  "actions": [
    {
      "add": {
        // 指定别名、索引、以及对该索引的过滤方式
        "index": "doc-join-relation",
        "alias": "myalias",
        "filter": {
          "term": {
            "title": "很棒的博客"
          }
        }
      }
    }
  ]
}

// 通过别名来查询
GET myalias/_search
{
  "query": {
    "match_all": {}
  }
}
```

路由很好理解，决定索引分片存储在哪个节点，以及决定去哪个节点查询索引分片数据。默认情况下，ES会使用文档ID的哈希值去进行路由。但是这个完全随机性的路由方式显然不够优秀。对于那些经常在查询中体现相同性质的文档。如果能够将他们路由到同一个节点（比如经常按属性A分类查询，那么就按属性A的值去进行散列并路由），显然能提高查询速度。不过应当注意的是，用户指定路由的情况下，ES会放弃对文档ID的判重，因此需要用户自行保证。而且由于查询操作只会被限定在符合路由的节点上发生，因此也需要用户保证同节点存储了所需要的全部数据。例子如下
```kibana
// 想要看出路由的区别，必须先组成多节点的集群

// 通过URL给出路由值
PUT doc-relation/_doc/1?routing=test
{
  "title":"what",
  "content":"myaliasContent"
}

// 查询指定路由值指向的节点的文档数据
POST myalias/_search?routing=test
{
  "query": {
    "match_all": {}
  }
}

// 查看在test路由值下的分片节点
GET myalias/_search_shards?routing=test


// 配置别名过滤器中的路由字段
POST _aliases
{
  "actions": [
    {
      "add": {
        // 指定别名、索引、以及对该索引的过滤方式
        "index": "doc-join-relation",
        "alias": "myalias",
        "filter": {
          "term": {
            "title": "很棒的博客"
          }
        },
        // 各个别名所代表的索引数据中，只返回满足该路由条件的文档
        "routing":"test"
      }
    }
  ]
}

```


## 性能优化思路
1. 合并请求：减少网络通信性能损失，使用bulk进行批量处理
2. 优化Lucene分段：ES接收文档后，会处理并缓存，再进一步存储到内存中的分段结构（Segments），分段会时不时的写入磁盘。这个过程中几乎每一部都是很耗时的操作。
    - 刷新Refresh：从缓存（不可被查询到）写入到分段中（可被索引）
    - 冲刷Flush：从分段写入磁盘
    - 分段合并：分段是不断从缓存创建的，对多个小分段的处理不如一个大分段的处理。因此合并分段可以提高速度。
3. 权衡索引速度和查询速度
4. 预热和内存：充分利用过滤器缓存、分片缓存。
5. 谨慎使用脚本：如果有需要，可以使用Java编写，并将其以插件形式植入ES的运行时。
6. JVM优化


## 其他
1. cat API：对于许多信息都有可视化效果更好的查询接口。但一定注意，官方设计这个接口的目的**只是为了**人工在命令行或者kibana使用，而不是给应用程序使用。用法例如
    ```kibana
    // 查询全部节点
    GET _cat/nodes

    // 查询全部索引
    GET _cat/indices
    ```


## 参考资料
《ElasticSearch 实战》

[ElasticSearch概述](https://www.cnblogs.com/xfeiyun/p/15887142.html)

[官方文档ElasticSearch Current Reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro.html)

[Kibana用户指南Console](https://segmentfault.com/a/1190000016391736)

[Elasticsearch索引、搜索流程及集群选举细节整理](https://cloud.tencent.com/developer/article/1863678)