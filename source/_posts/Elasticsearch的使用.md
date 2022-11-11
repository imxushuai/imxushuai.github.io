---
title: Elasticsearch的使用
photo:
  - 'https://imxushuai-blog.oss-cn-chengdu.aliyuncs.com/elasticsearch-logo.png'
tags:
  - elasticsearch教程
  - elasticsearch入门
  - elasticsearch
  - spring data elasticsearch
  - 全文检索
categories:
  - 全文检索
  - elasticsearch
description: elasticsearch的基本使用。包括原生api的使用以及java客户端的使用
abbrlink: 44334
date: 2019-03-29 15:26:10
---

<center><i>elasticsearch的基本使用。包括原生api的使用以及java客户端的使用</i></center>

![](https://imxushuai-blog.oss-cn-chengdu.aliyuncs.com/elasticsearch-logo.png)

<!-- more -->

## Elasticsearch基本概念

| 概念                 | 说明                                                         |
| :------------------- | ------------------------------------------------------------ |
| 索引（indices)       | indices是index的复数，代表许多的索引，                       |
| 类型（type）         | 类型是模拟mysql中的table概念，一个索引库下可以有不同类型的索引，比如商品索引，订单索引，其数据格式不同。不过这会导致索引库混乱，因此未来版本中会移除这个概念 |
| 文档（document）     | 存入索引库原始的数据。比如每一条商品信息，就是一个文档       |
| 字段（field）        | 文档中的属性                                                 |
| 映射配置（mappings） | 字段的数据类型、属性、是否索引、是否存储等特性               |

另外，在SolrCloud中，有一些集群相关的概念，在Elasticsearch也有类似的：

- 索引集（Indices，index的复数）：逻辑上的完整索引
- 分片（shard）：数据拆分后的各个部分
- 副本（replica）：每个分片的复制

> 要注意的是：Elasticsearch本身就是分布式的，因此即便你只有一个节点，Elasticsearch默认也会对你的数据进行分片和副本操作，当你向集群添加新数据时，数据也会在新加入的节点中进行平衡。

## Elasticsearch API

[Elasticsearch官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)

> 官方文档有更详尽的API介绍和说明

### 索引操作

> 使用http client发送请求即可，且所有数据均已json格式传输

#### 创建索引库

1. Request

   ```http
   PUT test_index
   {
     "settings": {
       "number_of_shards": 5, 
       "number_of_replicas": 1
     }
   }
   ```

   - `PUT`请求
   - `test_index`为索引库名
   - `number_of_shards`集群分片个数
   - `number_of_replicas`每个分片的副本数

2. Response

   ```http
   {
     "acknowledged" : true,
     "shards_acknowledged" : true,
     "index" : "test_index"
   }
   ```

#### 查询索引库

1. Request

   ```http
   GET test_index
   ```

   - `GET`请求

2. Response

   ```http
   {
     "test_index" : {
       "aliases" : { },
       "mappings" : { },
       "settings" : {
         "index" : {
           "creation_date" : "1551211641811",
           "number_of_shards" : "5",
           "number_of_replicas" : "1",
           "uuid" : "yBUaGT3pScWjmfBqoGRjkQ",
           "version" : {
             "created" : "6060199"
           },
           "provided_name" : "test_index"
         }
       }
     }
   }
   ```

   **若查询的索引库不存在，则返回404**

#### 删除索引库

1. Request

   ```http
   DELETE test_index
   ```

2. Response

   ```http
   {
     "acknowledged" : true
   }
   ```

### 配置索引库

#### 新增索引库配置

1. Request格式

   ```http
   PUT 索引库名/_mapping/类型名称
   {
     "properties": {
       "字段名":{
         "type": "text",
         "store": true,
         "index": false,
         "analyzer": "ik_smart"
       }
     }
   }
   ```

   - 类型名称：自定义，标识同一类数据，类似数据库中的数据表的概念

   - 字段名：自定义，标识一类数据中第一个属性，类似数据库中的字段的概念

   - type：数据类型，如：text，long，boolean等

     > 

   - store：默认为false，是否存储该字段的数据

   - index：默认为true，是否为该字段创建索引

   - analyzer：是否分词，值为该字段的分词器

   > 结合示例请求更好理解。
   >
   > 这里只列出常用的一些配置属性，更全的配置属性可查看官方文档。

2. Request

   ```http
   PUT test_index/_mapping/students
   {
     "properties": {
       "name":{
         "type": "text"
       },
       "sex":{
         "type": "text"
       },
       "address":{
         "type": "text",
         "analyzer": "ik_max_word"
       }
     }
   }
   ```

   > 未定义的配置属性采用默认值

3. Response

   ```http
   {
     "acknowledged" : true
   }
   ```

#### 查询索引库配置

1. Request

   ```http
   GET test_index/_mapping
   ```

2. Response

   ```http
   {
     "test_index" : {
       "mappings" : {
         "students" : {
           "properties" : {
             "address" : {
               "type" : "text",
               "analyzer" : "ik_max_word"
             },
             "name" : {
               "type" : "text"
             },
             "sex" : {
               "type" : "text"
             }
           }
         }
       }
     }
   }
   ```

#### 常用配置详解

##### type

Elasticsearch中支持的数据类型非常丰富：

![](https://imxushuai-blog.oss-cn-chengdu.aliyuncs.com/1531712631982.png)

我们说几个关键的：

- String类型，又分两种：

  - text：可分词，不可参与聚合
  - keyword：不可分词，数据会作为完整字段进行匹配，可以参与聚合

- Numerical：数值类型，分两类

  - 基本数据类型：long、interger、short、byte、double、float、half_float
  - 浮点数的高精度类型：scaled_float
    - 需要指定一个精度因子，比如10或100。elasticsearch会把真实值乘以这个因子后存储，取出时再还原。

- Date：日期类型

  elasticsearch可以对日期格式化为字符串存储，但是建议我们存储为毫秒值，存储为long，节省空间。

##### index

index影响字段的索引情况。

- true：字段会被索引，则可以用来进行搜索。默认值就是true
- false：字段不会被索引，不能用来搜索

index的默认值就是true，也就是说你不进行任何配置，所有字段都会被索引。

但是有些字段是我们不希望被索引的，比如商品的图片信息，就需要手动设置index为false。

##### store

是否将数据进行额外存储。

在学习lucene和solr时，我们知道如果一个字段的store设置为false，那么在文档列表中就不会有这个字段的值，用户的搜索结果中不会显示出来。

但是在Elasticsearch中，即便store设置为false，也可以搜索到结果。

原因是Elasticsearch在创建文档索引时，会将文档中的原始数据备份，保存到一个叫做`_source`的属性中。而且我们可以通过过滤`_source`来选择哪些要显示，哪些不显示。

而如果设置store为true，就会在`_source`以外额外存储一份数据，多余，因此一般我们都会将store设置为false，事实上，**store的默认值就是false。**

### 数据操作

#### 新增数据

1. Request

   ```http
   POST test_index/students
   {
     "name":"小明",
     "sex":"male",
     "address":"中国四川成都"
   }
   ```

2. Response

   ```http
   {
     "_index" : "test_index",
     "_type" : "students",
     "_id" : "qxMPLWkBNG9ZoQJNaDll",
     "_version" : 1,
     "result" : "created",
     "_shards" : {
       "total" : 2,
       "successful" : 1,
       "failed" : 0
     },
     "_seq_no" : 0,
     "_primary_term" : 1
   }
   ```

   **`_id`唯一标识此条记录**

3. 自定义id

   - 只需要在请求路径中加上`id`字段即可。

   - 请求示例

     ```http
     POST test_index/students/9527
     {
       "name":"张小明",
       "sex":"male",
       "address":"中国上海浦东新区"
     }
     ```

     **可以在相应结果中，看到起id为自定义的id**

#### 修改数据

1. Request

   ```http
   PUT test_index/students/9527
   {
     "name":"张小明",
     "sex":"male",
     "address":"中国广东深圳"
   }
   ```

   - `PUT`请求
   - 必须制定id

2. Response

   ```http
   {
     "_index" : "test_index",
     "_type" : "students",
     "_id" : "9527",
     "_version" : 5,
     "result" : "updated",
     "_shards" : {
       "total" : 2,
       "successful" : 1,
       "failed" : 0
     },
     "_seq_no" : 4,
     "_primary_term" : 1
   }
   ```

#### 删除数据

1. Request

   ```http
   DELETE test_index/students/9527
   ```

### 数据查询

#### 基本查询

1. 查询指定ID。

   ```http
   // 索引库名/类型名/${ID}
   GET test_index/students/9527
   ```

2. 查询全部

   - 方式1

     ```http
     GET test_index/_search
     ```

   - 方式2

     ```http
     GET test_index/_search
     {
         "query":{
             "match_all": {}
         }
     }
     ```

   - 响应结果

     ```http
     {
       "took" : 1, // 查询花费的时间
       "timed_out" : false, // 是否超时
       // 当前索引库分片信息
       "_shards" : {
         "total" : 5,
         "successful" : 5,
         "skipped" : 0,
         "failed" : 0
       },
       // 搜索结果
       "hits" : {
         "total" : 3, // 搜索的记录数
         "max_score" : 1.0, // 搜索到的记录中的最高得分,可以理解为匹配度或相关性
         "hits" : [
           {
             "_index" : "test_index", // 当前记录所属索引库名
             "_type" : "students", // 当前记录所属类型名
             "_id" : "rBMSLWkBNG9ZoQJNJjlf", // 当前记录id
             "_score" : 1.0, // 当前记录得分
             // 当前记录的源数据
             "_source" : {
               "id" : "9527",
               "name" : "张小明",
               "sex" : "male",
               "address" : "中国上海浦东新区"
             }
           },
           {
             "_index" : "test_index",
             "_type" : "students",
             "_id" : "qxMPLWkBNG9ZoQJNaDll",
             "_score" : 1.0,
             "_source" : {
               "name" : "小明",
               "sex" : "male",
               "address" : "中国四川成都"
             }
           },
           {
             "_index" : "test_index",
             "_type" : "students",
             "_id" : "9527",
             "_score" : 1.0,
             "_source" : {
               "name" : "张小明",
               "sex" : "male",
               "address" : "中国广东深圳"
             }
           }
         ]
       }
     }
     ```

3. 匹配查询

   - Http请求格式

     ```http
     GET test_index/_search
     {
       "query": {
         "match": {
           "address": "四川"
         }
       }
     }
     ```

   - 响应结果

     ```http
     {
       "took" : 6,
       "timed_out" : false,
       "_shards" : {
         "total" : 5,
         "successful" : 5,
         "skipped" : 0,
         "failed" : 0
       },
       "hits" : {
         "total" : 1,
         "max_score" : 0.8630463,
         "hits" : [
           {
             "_index" : "test_index",
             "_type" : "students",
             "_id" : "qxMPLWkBNG9ZoQJNaDll",
             "_score" : 0.8630463,
             "_source" : {
               "name" : "小明",
               "sex" : "male",
               "address" : "中国四川成都"
             }
           }
         ]
       }
     }
     ```

4. 精确查询

   - Http请求

     ```http
     GET test_index/_search
     {
       "query": {
         "term": {
           "FIELD": {
             "value": "VALUE"
           }
         }
       }
     }
     ```

     > FIELD ：需要精确查询的字段
     >
     > VALUE ：要查询的值
     >
     > **注意：精确值一般为数字，时间，布尔等，或者未被分词的字符串**

   - 多精确值查询

     ```http
     GET test_index/_search
     {
       "query": {
         "term": {
           "FIELD": {
             "value": ["VALUE1","VALUE2","VALUE3","VALUE4"]
           }
         }
       }
     }
     ```

     > 匹配查询多个精确值

#### 高级查询

##### 布尔组合查询

> 布尔组合查询把各种其它查询通过`must`（与）、`must_not`（非）、`should`（或）的方式进行组合

- Http请求

  ```http
  GET test_index/_search
  {
      "query":{
          "bool":{
          	"must":     { "match": { "name": "张" }},
          	"must_not": { "match": { "address":  "上海" }},
          	"should":   { "match": { "address": "广东" }}
          }
      }
  }
  ```

- 响应结果

  ```http
  {
    "took" : 9,
    "timed_out" : false,
    "_shards" : {
      "total" : 5,
      "successful" : 5,
      "skipped" : 0,
      "failed" : 0
    },
    "hits" : {
      "total" : 1,
      "max_score" : 0.5753642,
      "hits" : [
        {
          "_index" : "test_index",
          "_type" : "students",
          "_id" : "9527",
          "_score" : 0.5753642,
          "_source" : {
            "name" : "张小明",
            "sex" : "male",
            "address" : "中国广东深圳"
          }
        }
      ]
    }
  }
  ```

##### 范围查询

>  范围查询可以查询那些落在指定区间内的数字或者时间
>
> 新增测试数据:
>
> ```http
> POST test_index/students
> {
>   "name":"小红",
>   "sex":"female",
>   "address":"中国四川成都",
>   "age":16
> }
> ```
>
> elasticsearch可以自动添加映射，我们可以直接在请求中添加一个`mapping`中没有定义的字段，比如上面请求中的`age`字段

- Http请求

  ```http
  GET test_index/_search
  {
      "query":{
          "range": {
              "age": {
                  "gte":  10,
                  "lte":   17
              }
      	}
      }
  }
  ```

  | 操作符 |   说明   |
  | :----: | :------: |
  |   gt   |   大于   |
  |  gte   | 大于等于 |
  |   lt   |   小于   |
  |  lte   | 小于等于 |

- 响应结果

  ```http
  {
    "took" : 22,
    "timed_out" : false,
    "_shards" : {
      "total" : 5,
      "successful" : 5,
      "skipped" : 0,
      "failed" : 0
    },
    "hits" : {
      "total" : 1,
      "max_score" : 1.0,
      "hits" : [
        {
          "_index" : "test_index",
          "_type" : "students",
          "_id" : "rhNeMWkBNG9ZoQJNXDmy",
          "_score" : 1.0,
          "_source" : {
            "name" : "小红",
            "sex" : "female",
            "address" : "中国四川成都",
            "age" : 16
          }
        }
      ]
    }
  }
  ```

##### 模糊查询

>  模糊查询允许用户搜索词条与实际词条的拼写出现偏差，但是偏差的编辑距离不得超过2
>
> 添加数据：
>
> ```http
> POST test_index/students
> {
>   "name":"xiaowang",
>   "sex":"female",
>   "address":"中国北京西二旗",
>   "age":18
> }
> ```

- Http请求

  ```http
  GET test_index/_search
  {
      "query":{
          "fuzzy": {
            "name": "xiaowajh"
          }
      }
  }
  ```

  > 故意输入错误的值：`xiaowajh`

- 响应结果

  ```http
  {
    "took" : 28,
    "timed_out" : false,
    "_shards" : {
      "total" : 5,
      "successful" : 5,
      "skipped" : 0,
      "failed" : 0
    },
    "hits" : {
      "total" : 1,
      "max_score" : 0.65353876,
      "hits" : [
        {
          "_index" : "test_index",
          "_type" : "students",
          "_id" : "rxN3MWkBNG9ZoQJN9zlb",
          "_score" : 0.65353876,
          "_source" : {
            "name" : "xiaowang",
            "sex" : "female",
            "address" : "中国北京西二旗",
            "age" : 18
          }
        }
      ]
    }
  }
  ```

  > 输入错误的值，依旧能查询到。
  >
  > **注意：值的偏差不能大于两个字符，且对中文的支持不太友好**

#### 结果过滤（_source）

1. 直接使用字段名过滤
   - Http请求

     ```http
     GET test_index/_search
     {
       "_source": ["name", "address"], 
       "query": {
         "match_all": {}
       }
     }
     ```

     > 使用 `_source` 控制结果集中包含的数据

   - 响应结果

     ```http
     {
       "took" : 8,
       "timed_out" : false,
       "_shards" : {
         "total" : 5,
         "successful" : 5,
         "skipped" : 0,
         "failed" : 0
       },
       "hits" : {
         "total" : 3,
         "max_score" : 1.0,
         "hits" : [
           {
             "_index" : "test_index",
             "_type" : "students",
             "_id" : "rBMSLWkBNG9ZoQJNJjlf",
             "_score" : 1.0,
             "_source" : {
               "address" : "中国上海浦东新区",
               "name" : "张小明"
             }
           },
           {
             "_index" : "test_index",
             "_type" : "students",
             "_id" : "qxMPLWkBNG9ZoQJNaDll",
             "_score" : 1.0,
             "_source" : {
               "address" : "中国四川成都",
               "name" : "小明"
             }
           },
           {
             "_index" : "test_index",
             "_type" : "students",
             "_id" : "9527",
             "_score" : 1.0,
             "_source" : {
               "address" : "中国广东深圳",
               "name" : "张小明"
             }
           }
         ]
       }
     }
     ```

     > 结果返回的数据只包含了`address`和`name`

2. 使用关键字过滤

   ```http
   GET test_index/_search
   {
     "_source": {
         "includes":["name", "address"]
     }, 
     "query": {
       "match_all": {}
     }
   }
   ```

   > `includes`和`excludes`两个关键字也能完成搜索结果的过滤

#### 过滤（filter）

>  所有的查询都会影响到文档的评分及排名。如果我们需要在查询结果中进行过滤，并且不希望过滤条件影响评分，那么就不要把过滤条件作为查询条件来用，而是使用`filter`代替

- Http请求

  ```http
  GET test_index/_search
  {
      "query": {
        "constant_score": {
          "filter": {
            "match":{
              "address":"四川"
            }
          }
        }
      }
  }
  ```

#### 排序（order）

- Http请求

  ```http
  GET test_index/_search
  {
    "query": {
      "match": {
        "sex": "male"
      }
    },
    "sort": [
      {
        "age": {
          "order": "desc"
        }
      }
    ]
  }
  ```

### 聚合统计

聚合可以让我们极其方便的实现对数据的统计、分析。例如：

- 什么品牌的手机最受欢迎？
- 这些手机的平均价格、最高价格、最低价格？
- 这些手机每月的销售情况如何？

实现这些统计功能的比数据库的sql要方便的多，而且查询速度非常快，可以实现实时搜索效果。

#### 基本概念

Elasticsearch中的聚合，包含多种类型，最常用的两种，一个叫`桶`，一个叫`度量`：

> **桶（bucket）**

桶的作用，是按照某种方式对数据进行分组，每一组数据在ES中称为一个`桶`，例如我们根据国籍对人划分，可以得到`中国桶`、`英国桶`，`日本桶`……或者我们按照年龄段对人进行划分：0~10,10~20,20~30,30~40等。

Elasticsearch中提供的划分桶的方式有很多：

- Date Histogram Aggregation：根据日期阶梯分组，例如给定阶梯为周，会自动每周分为一组
- Histogram Aggregation：根据数值阶梯分组，与日期类似
- Terms Aggregation：根据词条内容分组，词条内容完全匹配的为一组
- Range Aggregation：数值和日期的范围分组，指定开始和结束，然后按段分组
- ……

综上所述，我们发现bucket aggregations 只负责对数据进行分组，并不进行计算，因此往往bucket中往往会嵌套另一种聚合：metrics aggregations即度量

> **度量（metrics）**

分组完成以后，我们一般会对组中的数据进行聚合运算，例如求平均值、最大、最小、求和等，这些在ES中称为`度量`

比较常用的一些度量聚合方式：

- Avg Aggregation：求平均值
- Max Aggregation：求最大值
- Min Aggregation：求最小值
- Percentiles Aggregation：求百分比
- Stats Aggregation：同时返回avg、max、min、sum、count等
- Sum Aggregation：求和
- Top hits Aggregation：求前几
- Value Count Aggregation：求总数
- ……

#### 添加测试数据索引库

创建索引：

```json
PUT /cars
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "transactions": {
      "properties": {
        "color": {
          "type": "keyword"
        },
        "make": {
          "type": "keyword"
        }
      }
    }
  }
}
```

**注意**：在ES中，需要进行聚合、排序、过滤的字段其处理方式比较特殊，因此不能被分词。这里我们将color和make这两个文字类型的字段设置为keyword类型，这个类型不会被分词，将来就可以参与聚合



导入数据

```json
POST /cars/transactions/_bulk
{ "index": {}}
{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
{ "index": {}}
{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
{ "index": {}}
{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
{ "index": {}}
{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }
```

#### 聚合为桶

1. Request

   ```http
   GET /cars/_search
   {
       "size" : 0,
       "aggs" : { 
           "popular_colors" : { 
               "terms" : { 
                 "field" : "color"
               }
           }
       }
   }
   ```

   - size： 查询条数，这里设置为0，因为我们不关心搜索到的数据，只关心聚合结果，提高效率
   - aggs：声明这是一个聚合查询，是aggregations的缩写
     - popular_colors：给这次聚合起一个名字，任意。
       - terms：划分桶的方式，这里是根据词条划分
         - field：划分桶的字段

2. Response

   ```http
   {
     "took": 1,
     "timed_out": false,
     "_shards": {
       "total": 1,
       "successful": 1,
       "skipped": 0,
       "failed": 0
     },
     "hits": {
       "total": 8,
       "max_score": 0,
       "hits": []
     },
     "aggregations": {
       "popular_colors": {
         "doc_count_error_upper_bound": 0,
         "sum_other_doc_count": 0,
         "buckets": [
           {
             "key": "red",
             "doc_count": 4
           },
           {
             "key": "blue",
             "doc_count": 2
           },
           {
             "key": "green",
             "doc_count": 2
           }
         ]
       }
     }
   }
   ```

   - hits：查询结果为空，因为我们设置了size为0
   - aggregations：聚合的结果
   - popular_colors：我们定义的聚合名称
   - buckets：查找到的桶，每个不同的color字段值都会形成一个桶
     - key：这个桶对应的color字段的值
     - doc_count：这个桶中的文档数量

   通过聚合的结果我们发现，目前红色的小车比较畅销！

#### 桶内度量

> 前面的例子告诉我们每个桶里面的文档数量，这很有用。 但通常，我们的应用需要提供更复杂的文档度量。 例如，每种颜色汽车的平均价格是多少？
>
> 因此，我们需要告诉Elasticsearch`使用哪个字段`，`使用何种度量方式`进行运算，这些信息要嵌套在`桶`内，`度量`的运算会基于`桶`内的文档进行
>
> 现在，我们为刚刚的聚合结果添加 求价格平均值的度量

1. Request

   ```http
   GET /cars/_search
   {
       "size" : 0,
       "aggs" : { 
           "popular_colors" : { 
               "terms" : { 
                 "field" : "color"
               },
               "aggs":{
                   "avg_price": { 
                      "avg": {
                         "field": "price" 
                      }
                   }
               }
           }
       }
   }
   ```

   - aggs：我们在上一个aggs(popular_colors)中添加新的aggs。可见`度量`也是一个聚合,度量是在桶内的聚合
   - avg_price：聚合的名称
   - avg：度量的类型，这里是求平均值
   - field：度量运算的字段

2. Reponse

   ```http
   ...
     "aggregations": {
       "popular_colors": {
         "doc_count_error_upper_bound": 0,
         "sum_other_doc_count": 0,
         "buckets": [
           {
             "key": "red",
             "doc_count": 4,
             "avg_price": {
               "value": 32500
             }
           },
           {
             "key": "blue",
             "doc_count": 2,
             "avg_price": {
               "value": 20000
             }
           },
           {
             "key": "green",
             "doc_count": 2,
             "avg_price": {
               "value": 21000
             }
           }
         ]
       }
     }
   ...
   ```

   > 可以看到每个桶中都有自己的`avg_price`字段，这是度量聚合的结果。
   >
   > 桶内度量的本质就是嵌套的`aggs`。

#### 高级

##### 阶梯分桶 Histogram

> histogram是把数值类型的字段，按照一定的阶梯大小进行分组。你需要指定一个阶梯值（interval）来划分阶梯大小。
>
> 举例：
>
> 比如你有价格字段，如果你设定interval的值为200，那么阶梯就会是这样的：
>
> 0，200，400，600，...
>
> 上面列出的是每个阶梯的key，也是区间的启点。
>
> 如果一件商品的价格是450，会落入哪个阶梯区间呢？计算公式如下：
>
> ```txt
> bucket_key = Math.floor((value - offset) / interval) * interval + offset
> ```
>
> value：就是当前数据的值，本例中是450
>
> offset：起始偏移量，默认为0
>
> interval：阶梯间隔，比如200
>
> 因此你得到的key = Math.floor((450 - 0) / 200) * 200 + 0 = 400

1. Request

   ```http
   GET /cars/_search
   {
     "size":0,
     "aggs":{
       "price":{
         "histogram": {
           "field": "price",
           "interval": 5000,
           "min_doc_count": 1
         }
       }
     }
   }
   ```

   > 使用 `min_doc_count`来约束最少文档数量为1，这样文档数量为0的桶会被过滤。

2. Response

   ```http
   {
     "took": 15,
     "timed_out": false,
     "_shards": {
       "total": 5,
       "successful": 5,
       "skipped": 0,
       "failed": 0
     },
     "hits": {
       "total": 8,
       "max_score": 0,
       "hits": []
     },
     "aggregations": {
       "price": {
         "buckets": [
           {
             "key": 10000,
             "doc_count": 2
           },
           {
             "key": 15000,
             "doc_count": 1
           },
           {
             "key": 20000,
             "doc_count": 2
           },
           {
             "key": 25000,
             "doc_count": 1
           },
           {
             "key": 30000,
             "doc_count": 1
           },
           {
             "key": 80000,
             "doc_count": 1
           }
         ]
       }
     }
   }
   ```

##### 范围分桶 Range

> 范围分桶与阶梯分桶类似，也是把数字按照阶段进行分组，只不过range方式需要你自己指定每一组的起始和结束大小。



### 扩展：Spring Data Elasticsearch

官方文档地址：[Spring Data Elasticsearch](https://spring.io/projects/spring-data-elasticsearch#overview)

#### 创建demo工程

1. pom.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.xushuai</groupId>
       <artifactId>elasticsearch-demo</artifactId>
       <version>1.0-SNAPSHOT</version>
   
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.0.1.RELEASE</version>
           <relativePath/> <!-- lookup parent from repository -->
       </parent>
   
       <properties>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
           <java.version>1.8</java.version>
       </properties>
   
       <dependencies>
           <!-- elasticsearch启动器 -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
           </dependency>
   
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
           
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
           </dependency>
       </dependencies>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   </project>
   ```

2. 启动类

   ```java
   package fun.xushuai.elasticsearch;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   
   @SpringBootApplication
   public class ElasticsearchApplication {
       public static void main(String[] args) {
           SpringApplication.run(ElasticsearchApplication.class, args);
       }
   
   }
   ```

3. 实体类

   ```java
   package fun.xushuai.elasticsearch.pojo;
   
   import lombok.Data;
   
   @Data
   public class Item {
   
       Long id;
       String title; //标题
       String category;// 分类
       String brand; // 品牌
       Double price; // 价格
       String images; // 图片地址
   }
   ```

4. 创建Test类

   ```java
   package fun.xushuai.elasticsearch;
   
   import fun.xushuai.elasticsearch.pojo.Item;
   import org.junit.Test;
   import org.junit.runner.RunWith;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.data.elasticsearch.core.ElasticsearchTemplate;
   import org.springframework.test.context.junit4.SpringRunner;
   
   /**
    * Spring Data的子项目的使用方式基本上都是大同小异
    */
   @SpringBootTest
   @RunWith(SpringRunner.class)
   public class ElasticsearchTest {
   
       @Autowired
       private ElasticsearchTemplate esClient;
   
   }
   
   ```

   > Spring Data Elasticsearch主要使用`ElasticsearchTemplate`以及`ElasticsearchRepository`的子接口进行操作，

#### 索引操作

##### 创建索引及映射

1. 配置实体类

   ```java
   package fun.xushuai.elasticsearch.pojo;
   
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   import org.springframework.data.annotation.Id;
   import org.springframework.data.elasticsearch.annotations.Document;
   import org.springframework.data.elasticsearch.annotations.Field;
   import org.springframework.data.elasticsearch.annotations.FieldType;
   
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   @Document(indexName = "goods", type = "item", shards = 1)
   public class Item {
       @Id
       Long id;
       
       @Field(type = FieldType.text, analyzer = "ik_smart")
       String title; //标题
       
       @Field(type = FieldType.keyword)
       String category;// 分类
       
       @Field(type = FieldType.keyword)
       String brand; // 品牌
       
       @Field(type = FieldType.Double)
       Double price; // 价格
       
       @Field(type = FieldType.keyword, index = false)
       String images; // 图片地址
   }
   ```

   > 在`Item.java`上添加`@Document`注解，定义索引名，类型名，分片数量，副本数量等信息。
   >
   > 使用`@Filed`注解定义映射信息。

2. 创建索引库

   ```java
       /**
        * 创建索引以及映射规则
        */
       @Test
       public void testCreateIndex() {
           // 创建索引
           esClient.createIndex(Item.class);
           // 创建映射
           esClient.putMapping(Item.class);
       }
   ```

   > 若运行成功，使用`http client`查看结果
   >
   > ![](https://imxushuai-blog.oss-cn-chengdu.aliyuncs.com/20190311225646.png)

##### 删除索引

```java
    /**
     * 删除索引
     */
    @Test
    public void testDeleteIndex() {
        // 删除索引
        esClient.deleteIndex(Item.class);
    }
```

#### 使用ElasticsearchRepository

> Spring Data 的强大之处，就在于你不用写任何DAO处理，自动根据方法名或类的信息进行CRUD操作。只要你定义一个接口，然后继承Repository提供的一些子接口，就能具备各种基本的CRUD功能。
>
> 使用过JPA的话 一定会很熟悉CrudRepository等接口
>
> ```java
> package fun.xushuai.elasticsearch.repository;
> 
> import fun.xushuai.elasticsearch.pojo.Item;
> import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
> 
> public interface EsRepository extends ElasticsearchRepository<Item, Long> {
> }
> 
> ```

##### CRUD

```java
package fun.xushuai.elasticsearch;

import fun.xushuai.elasticsearch.pojo.Item;
import fun.xushuai.elasticsearch.repository.EsRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.elasticsearch.core.ElasticsearchTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Collections;

/**
 * Spring Data的子项目的使用方式基本上都是大同小异
 */
@SpringBootTest
@RunWith(SpringRunner.class)
public class ElasticsearchTest {

    @Autowired
    private ElasticsearchTemplate esClient;

    @Autowired
    private EsRepository esRepository;

    /**
     * 创建索引以及映射规则
     */
    @Test
    public void testCreateIndex() {
        // 创建索引
        esClient.createIndex(Item.class);
        // 创建映射
        esClient.putMapping(Item.class);
    }

    /**
     * 删除索引
     */
    @Test
    public void testDeleteIndex() {
        // 删除索引
        esClient.deleteIndex(Item.class);
    }

    /**
     * 新增/更新 数据
     * 若指定id的记录已存在，则为更新，否则为新增
     */
    @Test
    public void testSave() {
        ArrayList<Item> items = new ArrayList<>();

        items.add(new Item(1L, "小米8", "手机", "小米", 2999d, "https://xiaomi.com"));
        items.add(new Item(2L, "华为8", "手机", "华为", 2999d, "https://xiaomi.com"));
        items.add(new Item(3L, "小米 MIX2", "手机", "小米", 3999d, "https://xiaomi.com"));
        // 批量保存
        esRepository.saveAll(items);
        // 保存单个 esRepository.save(item1);
    }

    /**
     * 查询
     */
    public void testFindOne() {
        // 按ID查询
        esRepository.findById(1L);
        // 按条件查询,需要到接口自定义查询方法
        esRepository.findByTitle("小米");
    }

    /**
     * 删除数据
     */
    @Test
    public void testDelete() {
        esRepository.deleteById(1L);
    }

}

```

##### 原生查询

```java
    /**
     * 借助Spring Data Elasticsearch使用原生查询
     */
    @Test
    public void testNativeSearch() {
        // 得到原生查询生成器对象
        NativeSearchQueryBuilder nativeSearchQueryBuilder = new NativeSearchQueryBuilder();
        // 查询条件
        nativeSearchQueryBuilder.withQuery(QueryBuilders.matchQuery("title", "小米"));
        // 结果过滤 FetchSourceFilter的参数可以任填其一
        nativeSearchQueryBuilder.withSourceFilter(new FetchSourceFilter(new String[]{"title", "price", "images"}, null));
        // 分页 注意：页码从0开始，即0为第一页
        nativeSearchQueryBuilder.withPageable(PageRequest.of(0, 2));
        // 排序 fieldSort：按字段排序 scoreSort：按得分排序
        nativeSearchQueryBuilder.withSort(SortBuilders.fieldSort("price").order(SortOrder.DESC));

        // 调用查询
        Page<Item> result = esRepository.search(nativeSearchQueryBuilder.build());

        /*
         * 获取查询结果:
         *      result.getTotalElements() : 获取总记录数
         *      result.getTotalPages() : 获取总页数
         *      result.getContent() : 获取查询结果
         *      result.getNumber() : 获取当前页码
         *      result.getPageable() : 获取分页对象
         *      result.getSize() : 获取每页记录数
         *      result.getSort() : 获取排序对象
         *
         */
        List<Item> content = result.getContent();
        content.forEach(item -> System.out.println(item.getTitle()));
    }
```

##### 聚合查询

```java
    /**
     * 聚合查询
     */
    @Test
    public void testAggregation() {
        // 得到原生查询生成器对象
        NativeSearchQueryBuilder nativeSearchQueryBuilder = new NativeSearchQueryBuilder();
        // 聚合
        String termAggregationName = "aggregation_brand";
        String avgAggregationName = "aggregation_brand_sum_price";
        // 统计所有brand并得到每个品牌手机的总和
        nativeSearchQueryBuilder.addAggregation(AggregationBuilders.terms(termAggregationName).field("brand")
                .subAggregation(AggregationBuilders.sum(avgAggregationName).field("price")));

        // 调用查询
        AggregatedPage<Item> result = esClient.queryForPage(nativeSearchQueryBuilder.build(), Item.class);

        // 获取查询结果
        Aggregations aggregations = result.getAggregations();
        StringTerms aggregationBrandResult = aggregations.get(termAggregationName);
        List<StringTerms.Bucket> buckets = aggregationBrandResult.getBuckets();
        // 输出结果
        System.out.println("品牌 | 款式数量 | 价格和");
        buckets.forEach(bucket -> {
            // 获取每个品牌手机总和
            InternalSum sumPrice = bucket.getAggregations().get(avgAggregationName);
            double sum = sumPrice.getValue();
            System.out.println(bucket.getKey() + " | " + bucket.getDocCount() + "款 | sum = " + sum);
        });

    }
```

结果:

```text
品牌 | 款式数量 | 价格和
小米 | 2款 | sum = 6998.0
华为 | 1款 | sum = 2999.0
```



