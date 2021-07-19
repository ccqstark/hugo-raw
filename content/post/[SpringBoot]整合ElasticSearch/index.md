---
title: "[SpringBoot]整合ElasticSearch"
date: 2021-02-04T23:01:00+08:00
draft: true
image: springboot-es.png
slug: "springboot_es"
tags:
    - ElasticSearch
    - SpringBoot
categories:
    - JavaEE
--- 

## 导入依赖

我们使用springboot操作es要用到对应的data相关starter

```xml
<!-- elasticsearch的starter依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
<!-- 将对象转为json传入source时要用 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.75</version>
</dependency>
```
> ⚠️ 各springboot的版本对应特定的elasticsearch版本，引入上面的依赖时会自动下载对应版本的rest-high-level-client，使用时尽量使得版本对应，避免潜在问题。

版本对应表如下：

![%E4%BD%BF%E7%94%A8SpringBoot%E6%93%8D%E4%BD%9CElasticSearch%201f30cd75c9374f088578e4c58ba60976/20201221111458835.png](%E4%BD%BF%E7%94%A8SpringBoot%E6%93%8D%E4%BD%9CElasticSearch%201f30cd75c9374f088578e4c58ba60976/20201221111458835.png)

我使用的这里用Springboot`2.4.1`，所以对应的elasticsearch是`7.9.3`版本

## 配置类

config目录下新建es的配置类`ElasticSearchClientConfig.java`

```java
@Configuration
public class ElasticSearchClientConfig {

  @Bean
  public RestHighLevelClient restHighLevelClient() {
      final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
      credentialsProvider.setCredentials(AuthScope.ANY,
              new UsernamePasswordCredentials("elastic", "[密码]"));
      RestHighLevelClient client = new RestHighLevelClient(
              RestClient.builder(
                      new HttpHost("[ip]", 9200, "http"))
                      .setHttpClientConfigCallback(httpClientBuilder -> {
                          httpClientBuilder.disableAuthCaching();
                          return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
                      }));
      return client;
  }
}
```

这里有用到x-pack基础安全功能，所以配置了用户和密码。如果没有用户和密码，参照官方文档连接代码如下：

```java
RestHighLevelClient client = new RestHighLevelClient(
        RestClient.builder(
                new HttpHost("localhost", 9200, "http"),
                new HttpHost("localhost", 9201, "http")));
```

## 使用客户端连接

使用时`@Autowired`一下，用自定义名字时就`@Qualifier`一下，不然就得对应上面的方法名

```java
@Autowired
@Qualifier("restHighLevelClient")
private RestHighLevelClient client;
```

## 索引相关操作

### 创建索引

```java
void testCreateIndex() throws IOException {
    // 1. 创建索引请求
    CreateIndexRequest request = new CreateIndexRequest("test");
    // 2. 执行请求
    CreateIndexResponse createIndexResponse =
            client.indices().create(request, RequestOptions.DEFAULT);

    System.out.println(createIndexResponse);
}
```

### 判断索引是否存在

```java
void testExistIndex() throws IOException {
    GetIndexRequest request = new GetIndexRequest("test2");
    boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
    System.out.println(exists);
}
```

### 删除索引

```java
void testDeleteIndex() throws IOException {

    DeleteIndexRequest request = new DeleteIndexRequest("test");

    AcknowledgedResponse delete = client.indices().delete(request, RequestOptions.DEFAULT);

    System.out.println(delete.isAcknowledged());

}
```

## 文档相关操作

### 添加文档

```java
void testAddDocument() throws IOException {
    // 创建对象
    User user = new User("ccqstark", 20);
    // 创建请求
    IndexRequest request = new IndexRequest("test");

    // 规则 put /test/_doc/1
    request.id("1");

    // 设置超时时间，可以不写，有默认参数
    request.timeout(TimeValue.timeValueSeconds(1));
    request.timeout("1s");

    // 将数据放入请求
    request.source(JSON.toJSONString(user), XContentType.JSON);

    // 客户端发送请求
    IndexResponse indexResponse = client.index(request, RequestOptions.DEFAULT);

    System.out.println(indexResponse.toString());
    System.out.println(indexResponse.status()); // 对应返回的状态
}
```

### 批量插入文档

```java
void testBulkRequest() throws IOException {

    BulkRequest bulkRequest = new BulkRequest();
    // 数据量多的话超时时间可以设置长一点
    bulkRequest.timeout("10s");

    List<User> userList = new ArrayList<>();
    userList.add(new User("ccq1", 1));
    userList.add(new User("ccq2", 2));
    userList.add(new User("ccq3", 3));
    userList.add(new User("ccq4", 4));
    userList.add(new User("ccq5", 5));

    // 批处理请求
    for (int i = 0; i < userList.size(); i++) {
        bulkRequest.add(
                // 批量更新或批量删除，就在这里修改对应的请求即可. 不指定id会自动生成随机不重复id
                new IndexRequest("test")
                        .id("" + (i + 1))
                        .source(JSON.toJSONString(userList.get(i)), XContentType.JSON));
    }

    BulkResponse bulkResponse = client.bulk(bulkRequest, RequestOptions.DEFAULT);

    // 是否失败，false代表成功
    System.out.println(bulkResponse.hasFailures());
}
```

### 判断文档是否存在

```java
void testIsExists() throws IOException {
    GetRequest getRequest = new GetRequest("test", "1");

    // 不获取返回的_source上下文了
    getRequest.fetchSourceContext(new FetchSourceContext(false));
    // 排序的字段
    getRequest.storedFields("_none_");

    boolean exist = client.exists(getRequest, RequestOptions.DEFAULT);
    System.out.println(exist);
}
```

### 获取文档信息

```java
void testGetDocument() throws IOException {
    // index     id
    GetRequest getRequest = new GetRequest("test", "1");

    GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);

    // 返回的全部内容，和用命令获取的结果一样
    System.out.println(getResponse);
    // 以map获取结果的source
    System.out.println(getResponse.getSourceAsMap());
    // 以string获取结果source
    System.out.println(getResponse.getSourceAsString());
}
```

### 更新文档

```java
void testUpdateDocument() throws IOException {

    UpdateRequest updateRequest = new UpdateRequest("test", "1");
    updateRequest.timeout("1s");

    User user = new User("ccq java", 18);
    updateRequest.doc(JSON.toJSONString(user), XContentType.JSON);

    UpdateResponse updateResponse = client.update(updateRequest, RequestOptions.DEFAULT);

    System.out.println(updateResponse.status());
}
```

### 删除文档

```java
void deleteDocument() throws IOException {

    DeleteRequest deleteRequest = new DeleteRequest("test", "3");

    DeleteResponse deleteResponse = client.delete(deleteRequest, RequestOptions.DEFAULT);

    System.out.println(deleteResponse.status());
}
```

## 高级查询

```java
void testSearch() throws IOException {

    // SearchRequest 搜索请求
    // SearchSourceBuilder 搜索条件构造
    SearchRequest searchRequest = new SearchRequest("test");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    // 不同查询方式都在这里设置
    // QueryBuilders.termQuery() 精确查询
    // QueryBuilders.matchAllQuery() 匹配所有
    TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("name", "ccq");
    sourceBuilder.query(termQueryBuilder);

    sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));

    // 分页
    sourceBuilder.from(0);
    sourceBuilder.size(10);

    // 高亮
    HighlightBuilder highlightBuilder = new HighlightBuilder();
    // 设置高亮的字段
    highlightBuilder.field("name");
    // 多个匹配字高亮
    highlightBuilder.requireFieldMatch(true);
    // 设置高亮标签
    highlightBuilder.preTags("<span style=\"color:#ffd73b\">");
    highlightBuilder.postTags("</span>");
    sourceBuilder.highlighter(highlightBuilder);

    // 执行搜索
    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

    System.out.println(JSON.toJSONString(searchResponse.getHits()));
    System.out.println("=========================================");
    for (SearchHit hit : searchResponse.getHits()) {
        // 获取高亮字段
        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        HighlightField highlightName = highlightFields.get("name");
        // 原来的结果
        Map<String, Object> sourceAsMap = hit.getSourceAsMap();
        // 用高亮替换原来的，这里要判断一个是否为空，因为有可能没有高亮结果
        if (highlightName != null) {
            Text[] fragments = highlightName.fragments();

            // 只有当要高亮搜索当字段是数组类型fragments才会有多个元素，如果是单字段就去第0个就行
            sourceAsMap.put("name", fragments[0]);
        }
        System.out.println(sourceAsMap);
    }
}
```

参考自`狂神`的ElasticSearch教程：
[【狂神说Java】ElasticSearch7.6.x最新完整教程通俗易懂](https://www.bilibili.com/video/BV17a4y1x7zq?p=12)