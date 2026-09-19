# Day 73｜RabbitMQ 重试与延迟消息、Elasticsearch 基础

> 本篇分为两部分：**RabbitMQ 消费可靠性与延迟消息**，以及 **Elasticsearch 搜索与 Mapping 入门**。内容根据 `day73.xmind` 整理，示例配置以原笔记为准。

---

## 一、RabbitMQ：消费者重试与消息处理

### 1. 为什么需要消费者本地重试

消费者处理消息时若抛出业务异常，直接将消息重新入队（`requeue`）可能形成“取出 → 失败 → 入队”的无限循环。可以启用 Spring AMQP 的**消费者本地重试**：先在消费者进程内有限次重试，次数用尽后再决定如何处置消息。

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1
        acknowledge-mode: auto
        retry:
          enabled: true
          initial-interval: 1000ms
          multiplier: 1
          max-attempts: 3
          stateless: true
```

| 参数 | 作用 |
|---|---|
| `prefetch: 1` | 每个消费者一次只预取少量未确认消息，便于按处理能力分配任务 |
| `acknowledge-mode: auto` | 由 Spring AMQP 根据监听方法的执行结果自动确认 |
| `enabled: true` | 启用消费者失败后的本地重试；原笔记指出默认关闭 |
| `initial-interval: 1000ms` | 初次重试前等待 1 秒 |
| `multiplier: 1` | 后续等待时间不增长 |
| `max-attempts: 3` | 最多尝试 3 次，包含首次消费 |
| `stateless: true` | 使用无状态重试；原笔记建议业务涉及事务时考虑 `false` |

> 本地重试会延长单条消息的处理时间。重试次数和间隔应结合业务耗时设置，不宜无限放大。

### 2. 重试耗尽后如何处理

原笔记列出三种思路：

| 处理方式 | 结果 | 适用考虑 |
|---|---|---|
| 拒绝并不重新入队 | 原队列中的消息被移除；笔记将其列为默认行为 | 不希望失败消息反复阻塞正常消费 |
| `NACK` 并重新入队 | 消息可能再次投递 | 适合短暂故障，但须避免无限循环 |
| 重新发布（Republish）到错误交换机 | 原消息确认，失败消息转入单独处理链路 | 便于隔离、排查和人工补偿 |

选择重新发布时，可配置 `MessageRecoverer`：

```java
@Bean
public MessageRecoverer messageRecoverer(RabbitTemplate rabbitTemplate) {
    return new RepublishMessageRecoverer(
            rabbitTemplate, "error.direct", "error");
}
```

这段代码表示：重试用尽后，将失败消息以路由键 `error` 发送到 `error.direct`。错误交换机和对应队列也需要提前声明并绑定，否则仍可能无法接收失败消息。

### 3. 保证业务幂等

消息可能因重试或重复投递而被处理不止一次，因此消费者的业务操作应具备**幂等性**：同一条业务消息执行多次，最终效果仍与执行一次相同。

1. **消息 ID 去重**：为消息生成全局唯一 ID；消费前检查该 ID 是否已经成功处理，处理成功后再记录。原笔记给出的配置是：

   ```java
   jackson2JsonMessageConverter.setCreateMessageIds(true);
   ```

2. **业务状态判断**：依据订单号、支付状态等业务唯一标识判断是否已经完成操作。例如订单已经支付，就不再重复扣款。

> 消息 ID 本身只能用于识别，仍需配合持久化的去重记录或业务唯一约束，才能在服务重启后继续防重。

---

## 二、RabbitMQ：延迟消息

延迟消息是指生产者发出消息后，不立即交给消费者，而是经过指定时间再投递。原笔记介绍了两种实现方式。

### 4. 方案一：TTL + 死信交换机

```text
生产者 → normal.direct → normal.queue（TTL 30 秒）
                            ↓ 过期后成为死信
消费者 ← dlx.queue ← dlx.direct
```

普通队列中的消息到期后，由队列配置的死信交换机 `dlx.direct` 转发到 `dlx.queue`，最终交给消费者。在图示配置里，死信转发使用的路由键需要与死信队列的绑定键匹配。

**特点：**需要配置普通交换机、普通队列，以及死信交换机、死信队列和相应绑定。若不同 TTL 的消息混放在同一队列，后面的消息可能要等队首消息移走后才会被处理，造成实际延迟偏差。

### 5. 方案二：RabbitMQ 延迟消息插件

先安装并启用延迟消息插件，再声明延迟交换机、队列和绑定关系：

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "delay.queue", durable = "true"),
        exchange = @Exchange(
                name = "delay.direct",
                type = ExchangeTypes.DIRECT,
                durable = "true",
                delayed = "true"),
        key = "delay"
))
public void listen(String message) {
    // 处理到期消息
}
```

发送时为单条消息设置延迟，单位为毫秒：

```java
rabbitTemplate.convertAndSend(
        "delay.direct",
        "delay",
        "hello delay message",
        message -> {
            message.getMessageProperties().setDelay(30_000);
            return message;
        }
);
```

上例表示延迟约 **30 秒** 后再向绑定队列投递。

| 对比项 | TTL + 死信交换机 | 延迟消息插件 |
|---|---|---|
| 组件配置 | 需要额外的死信交换机与队列 | 需要安装插件并声明延迟交换机 |
| 延迟控制 | 依赖队列中消息到期后的死信转发 | 可为每条消息单独设置延迟 |
| 时间偏差 | 混合 TTL 时可能受队首消息影响 | 不依赖普通队列队首过期判断 |

> 延迟投递不等于“消费者恰好在指定毫秒执行”：实际处理时间仍受 Broker、队列积压和消费者负载影响。

---

## 三、Elasticsearch 基础

### 6. Elasticsearch 是什么

Elasticsearch（ES）是基于 Apache Lucene 构建的分布式 RESTful 搜索引擎，采用文档型数据组织方式，也是 Elastic Stack（常称 ELK）的核心组件之一。原笔记采用 Docker 安装，具体命令需参照配套安装文档。

### 7. 核心概念与 MySQL 类比

| Elasticsearch | 作用 | 便于理解的 MySQL 类比 |
|---|---|---|
| `index`（索引） | 一组同类文档的集合 | 表 |
| `document`（文档） | 一条 JSON 数据 | 一行记录 |
| `field`（字段） | 文档中的一个属性 | 一列 |
| `mapping`（映射） | 定义字段类型及索引方式 | 表结构 |

> 这些只是学习时的类比，ES 的文档模型与关系型表并不完全相同。

### 8. 正排索引与倒排索引

两者都是需要额外空间的数据组织方式，但查询方向不同：

| 类型 | 映射方向 | 更适合 |
|---|---|---|
| 正排索引 | 文档 → 字段/内容 | 已知文档 ID，读取文档内容 |
| 倒排索引 | 词项 → 文档列表 | 已知关键词，查找包含该词的文档 |

例如文档 1 包含“消息可靠性”，文档 2 包含“消息延迟”：按文档查看内容是正向读取；搜索“消息”并定位文档 1、2，则依靠倒排索引。

### 9. IK 中文分词器

原笔记指出，ES 默认分词器对中文场景不够友好，因此可安装 IK 中文分词器；安装时应选择与 ES 版本相匹配的插件。

| 模式 | 特点 | 常见用途 |
|---|---|---|
| `ik_smart` | 粗粒度分词 | 搜索查询时减少切分 |
| `ik_max_word` | 细粒度分词 | 写入时构建更丰富的检索词项 |

IK 还支持自定义**扩展词库**与**停用词库**，可根据项目中的专有名词和无意义词调整分词效果。

### 10. Mapping 常用属性

| 属性 | 作用 | 要点 |
|---|---|---|
| `type` | 定义字段数据类型 | 如 `text`、`keyword`、数值、布尔、日期、对象 |
| `index` | 是否为字段建立倒排索引 | 默认通常为 `true`；关闭后该字段不能按普通倒排索引方式检索 |
| `analyzer` | 指定分词器 | 主要用于需要分词的 `text` 字段 |
| `properties` | 定义对象的子字段 | 常用于 `object` 类型 |

#### 常见字段类型

- **字符串**：`text` 会分词，适合全文检索；`keyword` 不分词，适合精确值。
- **整数**：`byte`、`short`、`integer`、`long`。
- **小数**：`float`、`double`。
- **其他**：`boolean`、`date`、`object`。

写入 `text` 字段时，`analyzer` 决定如何切词并建立倒排索引。原笔记建议建索引时采用细粒度分词，以保留更多可搜索的词项。

---

## ✅ 复习清单

- [ ] 能解释消费者本地重试与重新入队的区别
- [ ] 能设置 `max-attempts`、重试间隔和确认模式
- [ ] 能说明重试耗尽后拒绝、重新入队、重新发布的不同结果
- [ ] 能用消息 ID 或业务唯一键实现幂等处理
- [ ] 能画出 TTL + 死信交换机的延迟消息链路
- [ ] 能声明延迟交换机，并为消息设置延迟毫秒数
- [ ] 能区分 ES 的索引、文档、字段与 Mapping
- [ ] 能解释正排索引和倒排索引的查询方向
- [ ] 能区分 `ik_smart` 与 `ik_max_word`
- [ ] 能选择 `text`、`keyword` 等常见字段类型
