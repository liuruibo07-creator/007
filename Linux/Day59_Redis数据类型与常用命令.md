# Day 59：Redis 数据类型与常用命令

> 本笔记梳理 Redis 的键值结构、五种基本数据类型、常用命令，以及 Java 中使用 Spring Data Redis 的方式。

## 一、Redis 键值结构

Redis 是键值（Key-Value）数据库：

- **Key**：统一为字符串类型；
- **Value**：可使用多种数据结构。

| 分类 | 数据类型 | 含义 |
| --- | --- | --- |
| 基本类型 | `String` | 字符串 |
|  | `Hash` | 哈希表 |
|  | `List` | 列表 |
|  | `Set` | 无序集合 |
|  | `ZSet` | 有序集合 |
| 高级类型 | `Bitmap` | 位图 |
|  | `Geo` | 地理位置 |
|  | `HyperLogLog` | 基数统计 |

---

## 二、通用命令

| 命令 | 说明 | 示例 |
| --- | --- | --- |
| `KEYS pattern` | 查询匹配模式的 Key，支持 `*` 通配符 | `KEYS user:*` |
| `DEL key [key ...]` | 删除一个或多个 Key | `DEL user:1` |
| `EXPIRE key seconds` | 为 Key 设置秒级过期时间 | `EXPIRE token:1 3600` |
| `EXISTS key [key ...]` | 判断 Key 是否存在 | `EXISTS user:1` |
| `TTL key` | 查看剩余过期秒数 | `TTL token:1` |

> [!WARNING]
> `KEYS *` 会遍历所有 Key；生产环境中数据量大时可能阻塞 Redis，建议使用 `SCAN` 渐进式遍历。

---

## 三、五种基本数据类型

### 1. String（字符串）

最基础的数据类型，可存储普通文本、数字、JSON 字符串等。数值内容支持原子递增或递减。

```bash
# 设置与读取
SET user:name "张三"
GET user:name

# 批量设置与读取
MSET user:1 "张三" user:2 "李四"
MGET user:1 user:2

# 整数递增、按步长增减
SET article:views 10
INCR article:views
INCRBY article:views 5
INCRBY article:views -2

# 小数递增
SET product:price 19.9
INCRBYFLOAT product:price 0.5

# 仅当 Key 不存在时设置（常用于分布式锁）
SETNX lock:order "locked"

# 设置值时同时设定过期时间（秒）
SETEX token:1001 3600 "abc123"
```

| 常用命令 | 作用 |
| --- | --- |
| `SET key value` | 新增或覆盖 Key 的值 |
| `GET key` | 获取值 |
| `MSET` / `MGET` | 批量写入 / 读取 |
| `INCR` / `INCRBY` | 整数自增 / 按步长增减 |
| `INCRBYFLOAT` | 浮点数增减 |
| `SETNX` | 仅 Key 不存在时写入 |
| `SETEX` | 写入并设置过期时间 |

### 2. Hash（哈希）

Hash 类似 Java 的 `HashMap`：一个 Redis Key 下包含多个 `field-value` 字段，适合存储对象属性。

```bash
# 设置单个字段、获取字段
HSET user:1001 name "张三"
HSET user:1001 age 20
HGET user:1001 name

# 一次设置多个字段、一次读取多个字段
HSET user:1001 city "上海" level 3
HMGET user:1001 name age city

# 数值字段递增
HINCRBY user:1001 age 1
HINCRBYFLOAT account:1001 balance 99.5

# 仅当字段不存在时设置
HSETNX user:1001 phone "13800000000"

# 查看全部字段和值 / 仅字段 / 仅值
HGETALL user:1001
HKEYS user:1001
HVALS user:1001
```

| 常用命令 | 作用 |
| --- | --- |
| `HSET key field value` | 新增或修改字段 |
| `HGET key field` | 获取字段值 |
| `HMGET key field ...` | 批量获取字段值 |
| `HINCRBY` / `HINCRBYFLOAT` | 数值字段增减 |
| `HSETNX` | 字段不存在时写入 |
| `HGETALL` / `HKEYS` / `HVALS` | 获取全部内容 / 字段 / 值 |

> [!NOTE]
> 旧版本常见的 `HMSET` 已不建议使用；现在可通过一个 `HSET` 同时传入多个字段和值。

### 3. List（列表）

List 类似 Java 的 `LinkedList`，保持元素顺序，可从两端插入、弹出；适合消息队列、最近访问记录等。

```bash
# 从左侧（头部）和右侧（尾部）插入
LPUSH todo:queue "任务A" "任务B"
RPUSH todo:queue "任务C"

# 查看全部元素（0 到 -1 表示全部）
LRANGE todo:queue 0 -1

# 从两侧弹出元素；可指定数量
LPOP todo:queue
RPOP todo:queue 2

# 阻塞式弹出：队列为空时最多等待 30 秒
BLPOP todo:queue 30
BRPOP todo:queue 30
```

| 常用命令 | 作用 |
| --- | --- |
| `LPUSH` / `RPUSH` | 从左 / 右插入元素 |
| `LPOP` / `RPOP` | 从左 / 右弹出元素 |
| `LRANGE key start stop` | 按索引范围查询 |
| `BLPOP` / `BRPOP` | 阻塞式从左 / 右弹出 |

### 4. Set（无序集合）

Set 类似 Java 的 `HashSet`：元素不重复、无固定顺序，适合标签、共同好友、去重等场景。

```bash
# 添加元素；重复元素会被忽略
SADD user:1001:tags "Java" "Redis" "Linux"

# 查询、删除、统计
SMEMBERS user:1001:tags
SISMEMBER user:1001:tags "Redis"
SCARD user:1001:tags
SREM user:1001:tags "Linux"

# 集合运算
SADD user:1002:tags "Java" "MySQL"
SINTER user:1001:tags user:1002:tags
SDIFF user:1001:tags user:1002:tags
SUNION user:1001:tags user:1002:tags
```

| 常用命令 | 作用 |
| --- | --- |
| `SADD` / `SREM` | 添加 / 删除元素 |
| `SCARD` | 返回元素数量 |
| `SISMEMBER` | 判断元素是否存在 |
| `SMEMBERS` | 获取所有元素 |
| `SINTER` / `SDIFF` / `SUNION` | 交集 / 差集 / 并集 |

### 5. ZSet（有序集合）

ZSet（Sorted Set）类似 Java 的 `TreeSet`。每个成员都关联一个 `score`，Redis 会按分数排序；适用于排行榜、按热度排序等场景。

```bash
# 添加成员及分数
ZADD rank:game 98 "张三" 85 "李四" 92 "王五"

# 获取分数、排名与数量（排名从 0 开始）
ZSCORE rank:game "张三"
ZRANK rank:game "李四"
ZCARD rank:game

# 分数自增与删除
ZINCRBY rank:game 5 "李四"
ZREM rank:game "王五"

# 按排名范围获取：升序 / 降序
ZRANGE rank:game 0 -1 WITHSCORES
ZREVRANGE rank:game 0 2 WITHSCORES

# 按分数范围获取：升序 / 降序
ZRANGEBYSCORE rank:game 80 100 WITHSCORES
ZREVRANGEBYSCORE rank:game 100 80 WITHSCORES

# 统计给定分数区间的成员数
ZCOUNT rank:game 80 100
```

| 常用命令 | 作用 |
| --- | --- |
| `ZADD` / `ZREM` | 添加（携带分数）/ 删除成员 |
| `ZSCORE` / `ZRANK` | 查询分数 / 升序排名 |
| `ZCARD` / `ZCOUNT` | 总成员数 / 指定分数范围数量 |
| `ZINCRBY` | 增加成员分数 |
| `ZRANGE` / `ZREVRANGE` | 按排名升序 / 降序查询 |
| `ZRANGEBYSCORE` / `ZREVRANGEBYSCORE` | 按分数升序 / 降序查询 |

---

## 四、高级数据类型速览

| 类型 | 用途 |
| --- | --- |
| `Bitmap` | 用二进制位表示状态，如签到、活跃用户统计 |
| `Geo` | 保存经纬度并进行距离、附近地点查询 |
| `HyperLogLog` | 以较小内存近似统计去重后的数量（基数） |

---

## 五、Redis Java 客户端

常见 Java 客户端：

- **Jedis**：经典、直观的 Redis Java 客户端；
- **Lettuce**：支持异步与响应式编程；
- **Spring Data Redis**：对 Jedis、Lettuce 的 Spring 封装；Spring Boot 2 起默认使用 Lettuce。
### 1. Spring Boot 集成步骤

1. 引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2. 在 `application.yml` 配置 Redis 连接信息。
3. 注入 `RedisTemplate` 或 `StringRedisTemplate` 使用。

### 2. 序列化配置

`RedisTemplate` 默认使用 JDK 序列化，写入的 Key 和 Value 可读性较差，且会额外占用存储空间。常见做法是：

- Key、Hash Key：使用字符串序列化；
- Value、Hash Value：使用 JSON 序列化。

```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        template.setValueSerializer(RedisSerializer.json());
        template.setHashValueSerializer(RedisSerializer.json());
        template.afterPropertiesSet();
        return template;
    }
}
```
### 方式二
如果希望 Redis 中只保存纯字符串形式的 JSON，而不保留 Java 对象类型信息，可以自行完成 JSON 序列化与反序列化，并使用 `StringRedisTemplate`。

---

## 六、复习要点

```text
String：缓存、计数器、分布式锁
Hash：对象属性
List：队列、消息、时间线
Set：标签、去重、关系运算
ZSet：排行榜、按分数排序
```
