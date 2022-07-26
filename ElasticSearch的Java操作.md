---
title: ElasticSearch的Java操作
categories:
  - ElasticSearch
tags:
  - ElasticSearch
top_img: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp2.jpg'
cover: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp2.jpg'
abbrlink: 15202
date: 2021-11-08 23:57:05
updated: 2021-11-19 00:18:15
---

Elasticsearch 软件是由 Java 语言开发的，所以也可以通过 Java API 的方式对 Elasticsearch服务进行访问。

# 环境准备

pom文件导入相关依赖：

```xml
    <dependencies>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.8.0</version>
        </dependency>
        <!-- elasticsearch 的客户端 -->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.8.0</version>
        </dependency>
        <!-- elasticsearch 依赖 2.x 的 log4j -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.8.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.8.2</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.9.9</version>
        </dependency>
        <!-- junit 单元测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```



# 客户端对象

因为早期版本的客户端对象`TransportClient`已经不再推荐使用，且在未来版本中会被删除，所以这里我们采用高级 REST 客户端对象`RestHighLevelClient`：

```java
public class EsTets {

    public static void main(String[] args) throws IOException {
        //创建客户端对象
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost",9200,"http"))
        );

        //关闭客户端连接
        client.close();
    }
}

```

**9200 **端口为 Elasticsearch 的 Web 通信端口，**localhost** 为启动 ES 服务的主机名。执行代码，查看控制台信息：

>23:37:27.621 [main] DEBUG org.apache.http.impl.nio.conn.PoolingNHttpClientConnectionManager - Connection manager is shutting down
>23:37:27.628 [main] DEBUG org.apache.http.impl.nio.conn.PoolingNHttpClientConnectionManager - Connection manager shut down
>
>Process finished with exit code 0

说明启动成功。

---



# 索引操作

## 创建索引

```java
public class CreateIndex {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http"))
        );
        // 创建索引 - 请求对象
        CreateIndexRequest request = new CreateIndexRequest("user");
        // 发送请求，获取响应
        CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);
        // 响应状态
        boolean acknowledged = response.isAcknowledged();
        System.out.println("操作状态 = " + acknowledged);
        client.close();
    }
}
```

## 查询索引

```java
public class SearchIndex {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http"))
        );
        // 查询索引
        GetIndexRequest request = new GetIndexRequest("user");
        // 发送请求，获取响应
        GetIndexResponse response = client.indices().get(request, RequestOptions.DEFAULT);

        System.out.println(response.getAliases());
        System.out.println(response.getMappings());
        System.out.println(response.getSettings());
        client.close();
    }
}
```

## 删除索引

```java
public class DeleteIndex {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http"))
        );
        // 删除索引
        DeleteIndexRequest request = new DeleteIndexRequest("user");
        // 发送请求，获取响应
        AcknowledgedResponse response = client.indices().delete(request, RequestOptions.DEFAULT);

        System.out.println(response.isAcknowledged());
        client.close();
    }
}
```

---



# 文档操作

创建数据模型：

```java
package top.nanzx.es.pojo;

import lombok.Data;

@Data
public class User {
    private String name;
    private String sex;
    private Integer age;
}
```

## 创建文档

创建数据，添加到文档中：

```java
public class CreateDoc {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http"))
        );

        IndexRequest request = new IndexRequest("user");
        request.id("1001");

        User user = new User();
        user.setName("nan");
        user.setSex("男");
        user.setAge(23);

        ObjectMapper mapper = new ObjectMapper();
        String userJson = mapper.writeValueAsString(user);

        request.source(userJson, XContentType.JSON);

        IndexResponse response = client.index(request, RequestOptions.DEFAULT);

        System.out.println("_index:" + response.getIndex());
        System.out.println("_id:" + response.getId());
        System.out.println("_result:" + response.getResult());
        client.close();
    }
}
```

## 修改文档

```java
public class UpdateDoc {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http"))
        );

        UpdateRequest request = new UpdateRequest();
        request.index("user").id("1001");

        //设置请求体，对数据进行修改
        request.doc(XContentType.JSON, "sex", "女");

        UpdateResponse response = client.update(request, RequestOptions.DEFAULT);

        System.out.println("_index:" + response.getIndex());
        System.out.println("_id:" + response.getId());
        System.out.println("_result:" + response.getResult());
        client.close();
    }
}

```

## 查询文档

```java
public class SearchDoc {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http"))
        );

        GetRequest request = new GetRequest();
        request.index("user").id("1001");

        GetResponse response = client.get(request, RequestOptions.DEFAULT);

        System.out.println(response.getSourceAsString());
        client.close();
    }
}

```

## 删除文档

```java
public class DeleteDoc {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http"))
        );

        DeleteRequest request = new DeleteRequest("user");
        request.id("1001");

        DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);

        System.out.println("_result:" + response.toString());
        client.close();
    }
}
```

## 批量操作

### 批量创建

```java
public class BatchCreateDoc {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http"))
        );

        BulkRequest request = new BulkRequest();
        request.add(new IndexRequest("user").id("1001").source(XContentType.JSON,"name","zhangsan"));
        request.add(new IndexRequest("user").id("1002").source(XContentType.JSON,"name","lisi"));
        request.add(new IndexRequest("user").id("1003").source(XContentType.JSON,"name","wangwu"));

        BulkResponse response = client.bulk(request, RequestOptions.DEFAULT);

        System.out.println(response.getTook());
        System.out.println(response.getItems());
        client.close();
    }
}
```

### 批量删除

```java
public class BatchDeleteDoc {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("127.0.0.1", 9200, "http"))
        );

        BulkRequest request = new BulkRequest();
        request.add(new DeleteRequest("user").id("1001"));
        request.add(new DeleteRequest("user").id("1002"));
        request.add(new DeleteRequest("user").id("1003"));

        BulkResponse response = client.bulk(request, RequestOptions.DEFAULT);

        System.out.println(response.getTook());

        client.close();
    }
}
```

---



# 高级查询

## 全量查询

```java
public class MatchAllQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        // 查询所有数据
        sourceBuilder.query(QueryBuilders.matchAllQuery());
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 查询匹配
        SearchHits hits = response.getHits();
        System.out.println("took:" + response.getTook());
        System.out.println("timeout:" + response.isTimedOut());
        System.out.println("total:" + hits.getTotalHits());
        System.out.println("MaxScore:" + hits.getMaxScore());
        System.out.println("hits========>>");
        for (SearchHit hit : hits) {
            //输出每条查询的结果信息
            System.out.println(hit.getSourceAsString());
        }
        System.out.println("<<========");
        client.close();
    }
}
```

## 条件查询

```java
public class TermQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        // 条件数据
        sourceBuilder.query(QueryBuilders.termQuery("age",20));
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 查询匹配
        SearchHits hits = response.getHits();
        System.out.println("took:" + response.getTook());
        System.out.println("timeout:" + response.isTimedOut());
        System.out.println("total:" + hits.getTotalHits());
        System.out.println("MaxScore:" + hits.getMaxScore());
        System.out.println("hits========>>");
        for (SearchHit hit : hits) {
            //输出每条查询的结果信息
            System.out.println(hit.getSourceAsString());
        }
        System.out.println("<<========");
        client.close();
    }
}
```

## 分页查询

```java
public class LimitQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        // 分页查询
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        //(当前页码-1)*每页显示数据条数
        //from：当前页其实索引(第一条数据的顺序号)  size：每页显示多少条
        sourceBuilder.from(1).size(2);

        sourceBuilder.query(QueryBuilders.matchAllQuery());
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 查询匹配
        SearchHits hits = response.getHits();
        System.out.println("took:" + response.getTook());
        System.out.println("timeout:" + response.isTimedOut());
        System.out.println("total:" + hits.getTotalHits());
        System.out.println("MaxScore:" + hits.getMaxScore());
        System.out.println("hits========>>");
        for (SearchHit hit : hits) {
            //输出每条查询的结果信息
            System.out.println(hit.getSourceAsString());
        }
        System.out.println("<<========");
        client.close();
    }
}
```

## 查询排序

```java
public class SortQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        // 查询排序
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.sort("age", SortOrder.ASC);

        sourceBuilder.query(QueryBuilders.matchAllQuery());
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 查询匹配
        SearchHits hits = response.getHits();
        System.out.println("took:" + response.getTook());
        System.out.println("timeout:" + response.isTimedOut());
        System.out.println("total:" + hits.getTotalHits());
        System.out.println("MaxScore:" + hits.getMaxScore());
        System.out.println("hits========>>");
        for (SearchHit hit : hits) {
            //输出每条查询的结果信息
            System.out.println(hit.getSourceAsString());
        }
        System.out.println("<<========");
        client.close();
    }
}
```

## 过滤字段查询

```java
public class FilterQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        // 过滤字段查询
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        String[] excludes = {"sex"};
        String[] includes = {};
        sourceBuilder.fetchSource(includes, excludes);

        sourceBuilder.query(QueryBuilders.matchAllQuery());
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 查询匹配
        SearchHits hits = response.getHits();
        System.out.println("took:" + response.getTook());
        System.out.println("timeout:" + response.isTimedOut());
        System.out.println("total:" + hits.getTotalHits());
        System.out.println("MaxScore:" + hits.getMaxScore());
        System.out.println("hits========>>");
        for (SearchHit hit : hits) {
            //输出每条查询的结果信息
            System.out.println(hit.getSourceAsString());
        }
        System.out.println("<<========");
        client.close();
    }
}
```

## 组合查询

```java
public class CombineQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        // 组合查询
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        BoolQueryBuilder boolQueryBuilder = new BoolQueryBuilder();
        //必须包含
        boolQueryBuilder.must(QueryBuilders.matchQuery("age",30));
        // 一定不含
        boolQueryBuilder.mustNot(QueryBuilders.matchQuery("name", "zhangsan"));
        // 可能包含
        boolQueryBuilder.should(QueryBuilders.matchQuery("sex", "男"));

        sourceBuilder.query(boolQueryBuilder);
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 查询匹配
        SearchHits hits = response.getHits();
        System.out.println("took:" + response.getTook());
        System.out.println("timeout:" + response.isTimedOut());
        System.out.println("total:" + hits.getTotalHits());
        System.out.println("MaxScore:" + hits.getMaxScore());
        System.out.println("hits========>>");
        for (SearchHit hit : hits) {
            //输出每条查询的结果信息
            System.out.println(hit.getSourceAsString());
        }
        System.out.println("<<========");
        client.close();
    }
}
```

## 范围查询

```java
public class RangeQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        // 范围查询
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("age");
        // 大于等于
        rangeQuery.gte("20");
        // 小于等于
        rangeQuery.lte("40");

        sourceBuilder.query(rangeQuery);
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 查询匹配
        SearchHits hits = response.getHits();
        System.out.println("took:" + response.getTook());
        System.out.println("timeout:" + response.isTimedOut());
        System.out.println("total:" + hits.getTotalHits());
        System.out.println("MaxScore:" + hits.getMaxScore());
        System.out.println("hits========>>");
        for (SearchHit hit : hits) {
            //输出每条查询的结果信息
            System.out.println(hit.getSourceAsString());
        }
        System.out.println("<<========");
        client.close();
    }
}
```

## 模糊查询

```java
public class FuzzyQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        // 模糊查询
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();


        sourceBuilder.query(QueryBuilders.fuzzyQuery("name", "zhangsan").fuzziness(Fuzziness.ONE));
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 查询匹配
        SearchHits hits = response.getHits();
        System.out.println("took:" + response.getTook());
        System.out.println("timeout:" + response.isTimedOut());
        System.out.println("total:" + hits.getTotalHits());
        System.out.println("MaxScore:" + hits.getMaxScore());
        System.out.println("hits========>>");
        for (SearchHit hit : hits) {
            //输出每条查询的结果信息
            System.out.println(hit.getSourceAsString());
        }
        System.out.println("<<========");
        client.close();
    }
}
```

## 高亮查询

```java
public class HighLightQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        // 高亮查询
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("name", "lisi");
        sourceBuilder.query(termQueryBuilder);

        //构建高亮字段
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.preTags("<font color='red'>");//设置标签前缀
        highlightBuilder.postTags("</font>");//设置标签后缀
        highlightBuilder.field("name");//设置高亮字段
        sourceBuilder.highlighter(highlightBuilder);

        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 查询匹配
        SearchHits hits = response.getHits();
        System.out.println("took:" + response.getTook());
        System.out.println("timeout:" + response.isTimedOut());
        System.out.println("total:" + hits.getTotalHits());
        System.out.println("MaxScore:" + hits.getMaxScore());
        System.out.println("hits========>>");
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
            //打印高亮结果
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            System.out.println(highlightFields);
        }
        System.out.println("<<========");
        client.close();
    }
}
```

## 聚合查询

```java
public class AggregationQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        // 高亮查询
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        MaxAggregationBuilder aggregationBuilder = AggregationBuilders.max("max_age").field("age");

        sourceBuilder.aggregation(aggregationBuilder);
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 打印响应结果
        System.out.println(response);
        client.close();
    }
}
```

## 分组查询

```java
public class GroupQuery {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        SearchRequest request = new SearchRequest("user");
        // 高亮查询
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        TermsAggregationBuilder termsAggregationBuilder = AggregationBuilders.terms("age_groupby").field("age");

        sourceBuilder.aggregation(termsAggregationBuilder);
        request.source(sourceBuilder);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 打印响应结果
        System.out.println(response);
        client.close();
    }
}
```

