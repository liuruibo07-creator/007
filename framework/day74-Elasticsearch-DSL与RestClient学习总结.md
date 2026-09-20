# Day 74｜Elasticsearch DSL 与 RestClient 学习总结

> 本篇梳理 Elasticsearch 的**索引（Index）**和**文档（Document）**操作：先用 REST DSL 理解请求，再用 Java `RestHighLevelClient` 实现相同的增删改查与批量处理。

---

## 📌 知识总览

| 操作对象 | DSL 常用接口 | Java 客户端对象 |
|---|---|---|
| 索引 | `PUT /hotel`、`GET /hotel`、`DELETE /hotel` | `CreateIndexRequest`、`GetIndexRequest`、`DeleteIndexRequest` |
| Mapping 新增字段 | `PUT /hotel/_mapping` | `PutMappingRequest` |
| 文档 | `POST/PUT /index/_doc/{id}`、`GET`、`DELETE` | `IndexRequest`、`GetRequest`、`DeleteRequest` |
| 文档局部更新 | `POST /index/_update/{id}` | `UpdateRequest` |
| 批量文档操作 | 批量请求 | `BulkRequest` |

> **索引**可类比关系型数据库中的表，**文档**可类比一行记录，**Mapping** 描述字段及其类型。这个类比便于入门，但两者的数据模型并不完全相同。

---

## 一、使用 DSL 操作索引

### 1. 创建索引并定义 Mapping

原笔记以酒店数据为例，创建 `hotel` 索引。下面保留主要字段和配置：

```http
PUT /hotel
{
  "mappings": {
    "properties": {
      "id":       { "type": "keyword" },
      "name":     { "type": "text", "analyzer": "ik_max_word", "copy_to": "all" },
      "address":  { "type": "keyword", "index": false },
      "price":    { "type": "integer" },
      "score":    { "type": "integer" },
      "brand":    { "type": "keyword", "copy_to": "all" },
      "city":     { "type": "keyword" },
      "starName": { "type": "keyword" },
      "businenss": { "type": "keyword", "copy_to": "all" },
      "location": { "type": "geo_point" },
      "pic":      { "type": "keyword", "index": false },
      "all":      { "type": "text", "analyzer": "ik_max_word" }
    }
  }
}
```

| 配置 | 含义 |
|---|---|
| `text` + `analyzer` | 对文本分词，适合全文检索；示例使用 `ik_max_word` |
| `keyword` | 保留完整值，适合精确匹配 |
| `index: false` | 不为该字段建立普通检索所需的倒排索引 |
| `copy_to: "all"` | 将字段值复制到 `all` 字段，便于统一检索多个字段 |
| `geo_point` | 存储地理坐标 |

> **字段名核对：**原思维导图写的是 `businenss`，疑似想写 `business`。这里为忠实记录保留原名；实际建索引前应与 Java 模型、导入数据中的字段名核对，否则该字段可能无法按预期写入或检索。

### 2. 查询、删除与更新 Mapping

```http
GET /hotel

DELETE /hotel
```

已有字段的类型等 Mapping 定义通常不能直接修改；如需增加字段，可调用 `_mapping` 接口：

```http
PUT /hotel/_mapping
{
  "properties": {
    "height": { "type": "double" }
  }
}
```

原图中的新增字段示例使用 `/teacher/_mapping`，与酒店示例属于不同索引；这里统一写成 `/hotel/_mapping`，便于串联学习。

---

## 二、使用 DSL 操作文档

以下示例使用 `teacher2` 索引和文档 ID `1`。

### 3. 创建或覆盖文档

```http
POST /teacher2/_doc/1
{
  "info": "java讲师",
  "email": "zs@163.com",
  "name": {
    "firstName": "三",
    "lastName": "张"
  }
}
```

指定相同 ID 再写入完整文档时，可视为全量覆盖；更新时应提供希望保留的完整字段集合：

```http
PUT /teacher2/_doc/1
{
  "info": "java2讲师",
  "email": "zs2@163.com",
  "name": {
    "firstName": "三",
    "lastName": "张"
  }
}
```

### 4. 根据 ID 查询或删除

```http
GET /teacher2/_doc/1

DELETE /teacher2/_doc/1
```

### 5. 局部更新文档

只修改指定字段时使用 `_update`，更新内容放在 `doc` 中：

```http
POST /teacher2/_update/1
{
  "doc": {
    "info": "java讲师"
  }
}
```

| 更新方式 | 接口 | 特点 |
|---|---|---|
| 全量覆盖 | `PUT /index/_doc/{id}` | 用提交的完整文档替换旧内容；遗漏字段可能丢失 |
| 局部更新 | `POST /index/_update/{id}` | 只提交要修改的字段 |

---

## 三、Java RestClient 环境准备

### 6. 引入依赖并保持版本匹配

原笔记使用 `elasticsearch-rest-high-level-client`：

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
</dependency>
```

如果父工程管理了客户端版本，原示例通过下面的属性覆盖为 `7.12.1`，并要求与所连接的 ES 环境版本保持匹配：

```xml
<elasticsearch.version>7.12.1</elasticsearch.version>
```

### 7. 创建客户端

```java
RestHighLevelClient client = new RestHighLevelClient(
        RestClient.builder(
                new HttpHost("192.168.150.102", 9200, "http")
        )
);
```

`192.168.150.102:9200` 是原笔记的教学环境地址，实际使用时应替换为自己的 ES 地址，并在不再使用时关闭客户端。

---

## 四、使用 RestClient 操作索引

### 8. 创建索引

```java
CreateIndexRequest request = new CreateIndexRequest("hotel");
request.source(HOTEL_MAPPING, XContentType.JSON);
client.indices().create(request, RequestOptions.DEFAULT);
```

`HOTEL_MAPPING` 是 Mapping 的 JSON 字符串，可对应前面的 `PUT /hotel` 请求体。

### 9. 查询 Mapping 与判断索引是否存在

```java
GetIndexRequest request = new GetIndexRequest("hotel");
GetIndexResponse response = client.indices().get(request, RequestOptions.DEFAULT);

MappingMetadata mapping = response.getMappings().get("hotel");
log.info("mapping: {}", mapping.getSourceAsMap());

boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
log.info("exists: {}", exists);
```

### 10. 删除索引与新增字段

```java
client.indices().delete(
        new DeleteIndexRequest("hotel"),
        RequestOptions.DEFAULT
);
```

```java
PutMappingRequest request = new PutMappingRequest("hotel");
request.source("{\"properties\":{\"height\":{\"type\":\"double\"}}}", XContentType.JSON);
client.indices().putMapping(request, RequestOptions.DEFAULT);
```

> 删除索引会连同其中的文档一起删除。上面的代码是独立示例，学习时不要在创建和写入数据后立即执行删除。

---

## 五、使用 RestClient 操作文档

### 11. 新增酒店文档

从数据库读取酒店数据，转为 `HotelDoc`，再序列化成 JSON 写入 ES：

```java
Hotel hotel = hotelService.getById(36934L);
HotelDoc hotelDoc = new HotelDoc(hotel);

IndexRequest request = new IndexRequest("hotel")
        .id(hotelDoc.getId().toString());
request.source(JSONUtil.toJsonStr(hotelDoc), XContentType.JSON);

client.index(request, RequestOptions.DEFAULT);
```

### 12. 根据 ID 查询文档

```java
GetRequest request = new GetRequest("hotel", "36934");
GetResponse response = client.get(request, RequestOptions.DEFAULT);

String json = response.getSourceAsString();
HotelDoc hotelDoc = JSONUtil.toBean(json, HotelDoc.class);
log.info("hotelDoc: {}", hotelDoc);
```

使用返回数据前，应根据业务需要判断文档是否存在，以免对空结果执行反序列化。

### 13. 删除与局部更新

```java
client.delete(
        new DeleteRequest("hotel", "36934"),
        RequestOptions.DEFAULT
);
```

```java
UpdateRequest request = new UpdateRequest("hotel", "36934");
request.doc("price", 333);
client.update(request, RequestOptions.DEFAULT);
```

---

## 六、批量操作文档

`BulkRequest` 可以把多条文档操作合并成一次请求，适合批量导入或批量删除。

### 14. 批量导入

```java
BulkRequest request = new BulkRequest();
List<Hotel> hotels = hotelService.list();

for (Hotel hotel : hotels) {
    HotelDoc hotelDoc = new HotelDoc(hotel);
    request.add(new IndexRequest("hotel")
            .id(hotelDoc.getId().toString())
            .source(JSONUtil.toJsonStr(hotelDoc), XContentType.JSON));
}

client.bulk(request, RequestOptions.DEFAULT);
```

### 15. 批量删除

```java
BulkRequest request = new BulkRequest();
request.add(new DeleteRequest("hotel", "36934"));
request.add(new DeleteRequest("hotel", "38609"));

client.bulk(request, RequestOptions.DEFAULT);
```

实际批处理时，还应检查 Bulk 响应中的单项失败结果；整批请求返回并不代表每条文档都成功。

---

## 🔁 DSL 与 Java API 对照速记

| 目标 | DSL | Java 请求对象 |
|---|---|---|
| 创建索引 | `PUT /hotel` | `CreateIndexRequest` |
| 查看索引 | `GET /hotel` | `GetIndexRequest` |
| 删除索引 | `DELETE /hotel` | `DeleteIndexRequest` |
| 新增 Mapping 字段 | `PUT /hotel/_mapping` | `PutMappingRequest` |
| 写入文档 | `POST /hotel/_doc/{id}` | `IndexRequest` |
| 根据 ID 查询 | `GET /hotel/_doc/{id}` | `GetRequest` |
| 删除文档 | `DELETE /hotel/_doc/{id}` | `DeleteRequest` |
| 局部更新 | `POST /hotel/_update/{id}` | `UpdateRequest` |
| 批量处理 | Bulk API | `BulkRequest` |

> **一句话总结：**DSL 用于直接理解和验证 Elasticsearch 请求；RestClient 将同样的索引、Mapping、文档操作封装为 Java 请求对象，方便集成到业务代码中。

## ✅ 复习清单

- [ ] 能写出创建索引与 Mapping 的 DSL
- [ ] 能解释 `text`、`keyword`、`geo_point`、`copy_to` 与 `index: false`
- [ ] 能区分新增 Mapping 字段与修改已有字段
- [ ] 能区分文档全量覆盖和 `_update` 局部更新
- [ ] 能实例化 `RestHighLevelClient` 并配置 ES 地址
- [ ] 能使用 Java API 完成索引与文档的增删查改
- [ ] 能使用 `BulkRequest` 导入或删除多条文档，并检查单项结果
