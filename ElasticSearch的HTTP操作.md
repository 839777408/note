---
title: ElasticSearch的HTTP操作
categories:
  - ElasticSearch
tags:
  - ElasticSearch
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp1.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp1.jpg'
abbrlink: 6184
date: 2021-11-01 22:18:36
updated: 2021-11-8 23:51:44
---



# RESTful

​        REST 指的是一组架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful。Web 应用程序最重要的 REST 原则是，客户端和服务器之间的交互在请求之间是无状态的。从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外，无状态请求可以由任何可用服务器回答，这十分适合云计算之类的环境。客户端可以缓存数据以改进性能。

​        在服务器端，应用程序状态和功能可以分为各种资源。资源是一个有趣的概念实体，它向客户端公开。资源的例子有：应用程序对象、数据库记录、算法等等。每个资源都使用 URI (Universal Resource Identifier) 得到一个唯一的地址。所有资源都共享统一的接口，以便在客户端和服务器之间传输状态。使用的是标准的 HTTP 方法，比如 GET、PUT、POST 和DELETE。 

​        在 REST 样式的 Web 服务中，每个资源都有一个地址。资源本身都是方法调用的目标，方法列表对所有资源都是一样的。这些方法都是标准方法，包括 HTTP GET、POST、PUT、DELETE，还可能包括 HEAD 和 OPTIONS。简单的理解就是，如果想要访问互联网上的资源，就必须向资源所在的服务器发出请求，请求体中必须包含资源的网络路径，以及对资源进行的操作(增删改查)。

---



# 数据格式

Elasticsearch 是面向文档型数据库，一条数据在这里就是一个文档。为了方便理解，我们将 Elasticsearch 里存储文档数据和关系型数据库 MySQL 存储数据的概念进行一个类比：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211101234242.png)

ES 里的 Index 可以看做一个库，而 Types 相当于表，Documents 则相当于表的行。这里 Types 的概念已经被逐渐弱化，Elasticsearch 6.X 中，一个 index 下已经只能包含一个type，Elasticsearch 7.X 中, Type 的概念已经被删除了。



# 索引操作

## 创建索引

对比关系型数据库，创建索引就等同于创建数据库

在 Postman 中，向 ES 服务器发 **PUT** 请求 ：http://127.0.0.1:9200/shopping

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211103233844.png)

```yaml
{
 "acknowledged"【响应结果】: true, # true 操作成功
 "shards_acknowledged"【分片结果】: true, # 分片操作成功
 "index"【索引名称】: "shopping"
}
# 注意：创建索引库的分片数默认 1 片，在 7.0.0 之前的 Elasticsearch 版本中，默认 5 片
```

如果重复添加索引，会返回错误信息：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211103233946.png)

---

## 查看所有索引

在 Postman 中，向 ES 服务器发 **GET** 请求 ：http://127.0.0.1:9200/_cat/indices?v

这里请求路径中的_cat 表示查看的意思，indices 表示索引，所以整体含义就是查看当前 ES服务器中的所有索引，就好像 MySQL 中的 show tables 的感觉，服务器响应结果如下：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211103234806.png)

| 表头           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| health         | 当前服务器健康状态：green(集群完整) 、yellow(单点正常/集群不完整)、red(单点不正常) |
| status         | 索引打开、关闭状态                                           |
| index          | 索引名                                                       |
| uuid           | 索引统一编号                                                 |
| pri            | 主分片数量                                                   |
| rep            | 副本数量                                                     |
| docs.count     | 可用文档数量                                                 |
| docs.deleted   | 文档删除状态（逻辑删除）                                     |
| store.size     | 主分片和副分片整体占空间大小                                 |
| pri.store.size | 主分片占空间大小                                             |

---

## 查看单个索引

在 Postman 中，向 ES 服务器发 **GET** 请求 ：http://127.0.0.1:9200/shopping

查看索引向 ES 服务器发送的请求路径和创建索引是一致的。但是 HTTP 方法不一致。这里可以体会一下 RESTful 的意义，请求后，服务器响应结果如下：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211103235617.png)

```yaml
{
    "shopping"【索引名】: {
        "aliases"【别名】: {},
        "mappings"【映射】: {},
        "settings"【设置】: {
            "index""【设置 - 索引】: {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards"【设置 - 索引 - 主分片数量】: "1",
                "provided_name"【设置 - 索引 - 名称】: "shopping",
                "creation_date"【设置 - 索引 - 创建时间】: "1635953868328",
                "number_of_replicas"【设置 - 索引 - 副分片数量】: "1",
                "uuid"【设置 - 索引 - 唯一标识】: "bEIGo07lTDu5Xo0tQmcwNw",
                "version"【设置 - 索引 - 版本】: {
                    "created": "7150199"
                }
            }
        }
    }
}
```

---

## 删除索引

在 Postman 中，向 ES 服务器发 **DELETE** 请求 ：http://127.0.0.1:9200/shopping

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211104000508.png)

重新访问索引时，服务器返回响应：**索引不存在**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211104000537.png)

---

# 文档操作

## 创建文档

索引已经创建好了，接下来我们来创建文档，并添加数据。这里的文档可以类比为关系型数据库中的表数据，添加的数据格式为 JSON 格式。

在 Postman 中，向 ES 服务器发 **POST** 请求 ：http://127.0.0.1:9200/shopping**/_doc**

请求体内容为：

```yaml
{
 "title":"华为mate40 Pro",
 "category":"华为",
 "images":"https://www-file.huawei.com/-/media/corporate/images/home/logo/huawei_logo.png",
 "price":6999.00
}
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211104001143.png)

```yaml
{
    "_index"【索引】: "shopping",
    "_type"【类型-文档】: "_doc",
    "_id"【唯一标识】: "aFmU5nwBKn8hkLcxQ-bn", #可以类比为 MySQL 中的主键，随机生成
    "_version"【版本】: 1,
    "result"【结果】: "created", #这里的 create 表示创建成功
    "_shards"【分片】: {
        "total"【分片 - 总数】: 2,
        "successful"【分片 - 成功】: 1,
        "failed"【分片 - 失败】: 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```

上面的数据创建后，由于没有指定数据唯一性标识（ID），默认情况下，ES 服务器会随机生成一个。

此处发送请求的方式必须为 **POST**，不能是 **PUT**，否则会发生错误：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211104001226.png)

如果想要自定义唯一性标识，需要在创建时指定：http://127.0.0.1:9200/shopping/_doc/**1**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211104001652.png)

此处需要**注意**：如果增加数据时明确数据主键，那么请求方式也可以为 PUT

---

## 根据ID查看文档

查看文档时，需要指明文档的唯一性标识，类似于 MySQL 中数据的主键查询

在 Postman 中，向 ES 服务器发 **GET** 请求 ：http://127.0.0.1:9200/shopping**/_doc/1**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211104001942.png)

```yaml
{
    "_index"【索引】: "shopping",
    "_type"【文档类型】: "_doc",
    "_id": "1",
    "_version": 1,
    "_seq_no": 2,
    "_primary_term": 1,
    "found"【查询结果】: true, # true 表示查找到，false 表示未查找到
    "_source"【文档源信息】: {
        "title": "华为mate40 Pro",
        "category": "华为",
        "images": "https://www-file.huawei.com/-/media/corporate/images/home/logo/huawei_logo.png",
        "price": 6999.00
    }
}
```

---

## 查看所有文档

在 Postman 中，向 ES 服务器发 **GET** 请求 ：http://127.0.0.1:9200/shopping**/_search**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211104234015.png)

```yaml
{
    "took": 13,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 3,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "Z1mQ5nwBKn8hkLcx8OZT",
                "_score": 1.0,
                "_source": {
                    "title": "华为mate40 Pro",
                    "category": "华为",
                    "images": "https://www-file.huawei.com/-/media/corporate/images/home/logo/huawei_logo.png",
                    "price": 6999.00
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "aFmU5nwBKn8hkLcxQ-bn",
                "_score": 1.0,
                "_source": {
                    "title": "华为mate40 Pro",
                    "category": "华为",
                    "images": "https://www-file.huawei.com/-/media/corporate/images/home/logo/huawei_logo.png",
                    "price": 6999.00
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "1",
                "_score": 1.0,
                "_source": {
                    "title": "华为P50",
                    "category": "华为",
                    "images": "https://www-file.huawei.com/-/media/corporate/images/home/logo/huawei_logo.png",
                    "price": 5999.00
                }
            }
        ]
    }
}
```

---

## 高级查询文档

### 匹配查询

方式一（url参数）：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105232008.png)

方式二（请求体）：

match 匹配类型查询，会把查询条件进行分词，然后进行查询，多个词条之间是 or 的关系

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105232356.png)

---

### 字段匹配查询

multi_match 与 match 类似，不同的是它可以在多个字段中查询。

在 Postman 中，向 ES 服务器发 GET 请求 ：http://127.0.0.1:9200/shopping/_search

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211107165731.png)

---

### 关键字精确查询

term 查询，精确的关键词匹配查询，不对查询条件进行分词。

定义数据 :

```yaml
# POST /student/_doc/1001
{
    "name":"zhangsan",
    "nickname":"zhangsan",
    "sex":"男",
    "age":30
}
# POST /student/_doc/1002
{
    "name":"lisi",
    "nickname":"lisi",
    "sex":"男",
    "age":20 
 }
# POST /student/_doc/1003
{
     "name":"wangwu",
     "nickname":"wangwu",
     "sex":"女",
     "age":40 
 }
# POST /student/_doc/1004
{
     "name":"zhangsan1",
     "nickname":"zhangsan1",
     "sex":"女",
     "age":50 
 }
# POST /student/_doc/1005
{
     "name":"zhangsan2",
     "nickname":"zhangsan2",
     "sex":"女",
     "age":30 
 }
```

在 Postman 中，向 ES 服务器发 GET 请求 ：http://127.0.0.1:9200/shopping/_search

term 查询，精确的关键词匹配查询，不对查询条件进行分词。（自己试了下，中文好像只能匹配第一个字符）

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211107232317.png)

---

### 多关键字匹配查询

terms 查询和 term 查询一样，但它允许你指定多值进行匹配。

如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件，类似于 mysql 的 in

在 Postman 中，向 ES 服务器发 GET 请求 ：http://127.0.0.1:9200/student/_search

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211107232756.png)

---

### 全查询

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105232726.png)

---

### 分页查询

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105234753.png)

---

### 指定字段查询

默认情况下，Elasticsearch 在搜索的结果中，会把文档中保存在_source 的所有字段都返回。

如果我们只想获取其中的部分字段，我们可以添加_source 的过滤

在 Postman 中，向 ES 服务器发 GET 请求 ：http://127.0.0.1:9200/shopping/_search

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105235105.png)

---

### 字段排序

sort 可以让我们按照不同的字段进行排序，并且通过 order 指定排序的方式。desc 降序，asc升序。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105235447.png)

多字段排序：

```yaml
{
    "query":{
        "match_all":{

        }
    },
    "sort":[
        {
            "age":{
                "order":"desc"
            }
        },
        {
            "_score":{
                "order":"desc"
            }
        }
    ]
}
```



---

### 过滤字段

- includes：来指定想要显示的字段

- excludes：来指定不想要显示的字段

在 Postman 中，向 ES 服务器发 GET 请求 ：http://127.0.0.1:9200/student/_search

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211107233333.png)

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211107233447.png)

---

### 组合查询

`bool`把各种其它查询通过`must`（必须 ）、`must_not`（必须不）、`should`（应该，可以为）的方式进行组合

在 Postman 中，向 ES 服务器发 **GET** 请求 ：http://127.0.0.1:9200/student/_search

请求体：

```yaml
{
    "query":{
        "bool":{
            "must":[
                {
                    "match":{
                        "name":"zhangsan"
                    }
                }
            ],
            "must_not":[
                {
                    "match":{
                        "age":40
                    }
                }
            ],
            "should":[
                {
                    "match":{
                        "sex":"男"
                    }
                }
            ]
        }
    }
}
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211107235435.png)

---

### 范围查询

range 查询找出哪些落在指定区间内的数字或者时间。range 查询允许以下字符：

| 操作符 | 说明       |
| ------ | ---------- |
| gt     | 大于>      |
| gte    | 大于等于>= |
| lt     | 小于<      |
| lte    | 小于等于<= |

在 Postman 中，向 ES 服务器发 **GET** 请求 ：http://127.0.0.1:9200/student/_search

请求体：

```yaml
{
    "query":{
        "range":{
            "age":{
                "gte":30,
                "lte":35
            }
        }
    }
}
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108000012.png)

---

### 模糊查询

返回包含与搜索字词相似的字词的文档。

编辑距离是将一个术语转换为另一个术语所需的一个字符更改的次数。这些更改可以包括：

* 更改字符（box → fox） 

- 删除字符（black → lack）
- 插入字符（sic → sick） 
- 转置两个相邻字符（act → cat）

为了找到相似的术语，fuzzy 查询会在指定的编辑距离内创建一组搜索词的所有可能的变体或扩展。然后查询返回每个扩展的完全匹配。

通过 fuzziness 修改编辑距离。一般使用默认值 AUTO，根据术语的长度生成编辑距离。

在 Postman 中，向 ES 服务器发 **GET** 请求 ：http://127.0.0.1:9200/student/_search

```yaml
{
    "query":{
        "fuzzy":{
            "title":{
                "value":"zhangsan"
            }
        }
    }
}
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108210224.png)

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108222458.png)

---

### 高亮查询

在进行关键字搜索时，搜索出的内容中的关键字会显示不同的颜色，称之为高亮。

Elasticsearch 可以对查询内容中的关键字部分，进行标签和样式(高亮)的设置。

在使用 match 查询的同时，加上一个 highlight 属性：

- pre_tags：前置标签

- post_tags：后置标签

- fields：需要高亮的字段
- title：这里声明 title 字段需要高亮，后面可以为这个字段设置特有配置，也可以空

在 Postman 中，向 ES 服务器发 GET 请求 ：http://127.0.0.1:9200/student/_search

```yaml
{
    "query":{
        "match":{
            "name":"zhangsan"
        }
    },
    "highlight":{
        "pre_tags":"<font color='red'>",
        "post_tags":"</font>",
        "fields":{
            "name":{

            }
        }
    }
}
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108230318.png)

---

### 聚合查询

聚合允许使用者对 es 文档进行统计分析，类似与关系型数据库中的 group by，当然还有很

多其他的聚合，例如取最大值、平均值等等。

对某个字段取最大值 max：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108232818.png)

对某个字段取最小值 min：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108232914.png)

对某个字段求和 sum：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108233007.png)

对某个字段取平均值 avg：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108233156.png)

对某个字段的值进行去重之后再取总数 count：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108233307.png)

stats 聚合，对某个字段一次性返回 count，max，min，avg 和 sum 五个指标：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108233515.png)

---

### 桶聚合查询

桶聚和相当于 sql 中的 group by 语句

terms 聚合，分组统计：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211108235019.png)

---

## 修改文档

和新增文档一样，输入相同的 URL 地址请求，如果请求体变化，会将原有的数据内容覆盖

在 Postman 中，向 ES 服务器发 **POST** 请求 ：http://127.0.0.1:9200/shopping**/_doc/1**

请求体内容为: 

```yaml
{
 "title":"华为P50",
 "category":"华为",
 "images":"https://www-file.huawei.com/-/media/corporate/images/home/logo/huawei_logo.png",
 "price":5999.00
}
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211104080217.png)

```yaml
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version"【版本】: 2,
    "result"【结果】: "updated", # updated 表示数据被更新
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}
```

---

## 修改字段

修改数据时，也可以只修改某一给条数据的局部信息

在 Postman 中，向 ES 服务器发 **POST** 请求 ：http://127.0.0.1:9200/shopping/**_update/1**

请求体内容为：

```yaml
{ 
 "doc": {
 "price":3000.00
 } 
}
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105000454.png)

根据唯一性标识，查询文档数据，文档数据已经更新：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105000517.png)

---

## 删除文档

删除一个文档不会立即从磁盘上移除，它只是被标记成已删除（逻辑删除）。

在 Postman 中，向 ES 服务器发 **DELETE** 请求 ：http://127.0.0.1:9200/shopping**/_doc/1**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105000802.png)

删除后再查询当前文档信息：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105000842.png)

如果删除一个并不存在的文档：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105000812.png)

---

## 条件删除文档

一般删除数据都是根据文档的唯一性标识进行删除，实际操作时，也可以根据条件对多条数据进行删除

向 ES 服务器发 **POST** 请求 ：http://127.0.0.1:9200/shopping**/_delete_by_query**

请求体内容为：

```yaml
{
 "query":{
 	"match":{
		 "price":4000.00
 		}
	}
}
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211105223604.png)

```yaml
{
    "took"【耗时】: 38,
    "timed_out"【是否超时】: false,
    "total"【总数】: 2,
    "deleted"【删除数量】: 2,
    "batches": 1,
    "version_conflicts": 0,
    "noops": 0,
    "retries": {
        "bulk": 0,
        "search": 0
    },
    "throttled_millis": 0,
    "requests_per_second": -1.0,
    "throttled_until_millis": 0,
    "failures": []
}
```

---



# 映射操作

有了索引库，等于有了数据库中的 database。

接下来就需要建索引库(index)中的映射了，类似于数据库(database)中的表结构(table)。创建数据库表需要设置字段名称，类型，长度，约束等；索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做映射(mapping)。

## 创建映射

先创建索引，在 Postman 中，向 ES 服务器发 **PUT** 请求 ：http://127.0.0.1:9200/student

再创建映射，在 Postman 中，向 ES 服务器发 **PUT** 请求 ：http://127.0.0.1:9200/student**/_mapping**

请求体内容为：

```yaml
{
 "properties": {
     "name":{
         "type": "text",
         "index": true
     },
     "sex":{
         "type": "text",
         "index": false
     },
     "age":{
         "type": "long",
         "index": false
     }
  }
}
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211107132225.png)

映射数据说明：

- 字段名：任意填写，下面指定许多属性，例如：title、subtitle、images、price

- type：类型，Elasticsearch 中支持的数据类型非常丰富，说几个关键的：
  - String 类型，又分两种：
    - text：可分词
    - keyword：不可分词，数据会作为完整字段进行匹配，全词匹配
  - Numerical：数值类型，分两类
    - 基本数据类型：long、integer、short、byte、double、float、half_float
    - 浮点数的高精度类型：scaled_float
  -  Date：日期类型
  -  Array：数组类型
  - Object：对象

- index：是否索引，默认为 true，也就是说你不进行任何配置，所有字段都会被索引。

  - true：字段会被索引，则可以用来进行搜索

  - false：字段不会被索引，不能用来搜索

- store：是否将数据进行独立存储，默认为 false，原始的文本会存储在 _source 里面，默认情况下其他提取出来的字段都不是独立存储的，是从 _source 里面提取出来的。当然你也可以独立的存储某个字段，只要设置"store": true 即可，获取独立存储的字段要比从 _source 中解析快得多，但是也会占用更多的空间，所以要根据实际业务需求来设置。

- analyzer：分词器，这里的 ik_max_word 即使用 ik 分词器，后面会有专门的章节学习

---



## 查看映射

在 Postman 中，向 ES 服务器发 **GET** 请求 ：http://127.0.0.1:9200/student**/_mapping**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211107160909.png)

---



## 索引映射关联

在 Postman 中，向 ES 服务器发 **PUT** 请求 ：http://127.0.0.1:9200/student2

请求体：

```yaml
{
 "settings": {},
 "mappings": {
     "properties": {
        "name":{
             "type": "text",
             "index": true
        },
        "sex":{
             "type": "text",
             "index": false
        },
        "age":{
             "type": "long",
             "index": false
        }
      }
   } 
 }
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20211107161507.png)

索引需为新建索引，否则会报如下错误：

```yaml
{
    "error": {
        "root_cause": [
            {
                "type": "resource_already_exists_exception",
                "reason": "index [student/L5eMwGFvSFS1G3iFfVUaZQ] already exists",
                "index_uuid": "L5eMwGFvSFS1G3iFfVUaZQ",
                "index": "student"
            }
        ],
        "type": "resource_already_exists_exception",
        "reason": "index [student/L5eMwGFvSFS1G3iFfVUaZQ] already exists",
        "index_uuid": "L5eMwGFvSFS1G3iFfVUaZQ",
        "index": "student"
    },
    "status": 400
}
```

