---
title: Day84｜Spring AI 知识库、多模态与会话持久化
date: 2026-10-18
tags:
  - SpringAI
  - 知识库
  - 多模态
  - Redis
  - RAG
aliases:
  - day84
  - 知识库与多模态
---

# Day84｜Spring AI 知识库、多模态与会话持久化

> **内容来源：** `day84(1).xmind`（含导图中的代码与配置）  
> **学习主线：** 知识库（PDF 上传下载 + ChatClient 检索）→ 多模态（文本/图片/音频/视频）→ 会话记忆与列表持久化（Redis）  
> **整理说明：** 沿用课程 API 写法，省略部分 import 与依赖注入，需结合课程项目使用。原导图中的 `api-key` 已做脱敏处理。

## 📑 目录

- [一、知识总览](#overview)
- [二、知识库：PDF 上传与下载](#knowledge-upload-download)
- [三、配置 ChatClient：向量库与 Advisor](#chatclient)
- [四、改造 ChatController：会话隔离检索](#chatcontroller)
- [五、多模态：文本、图片、音频、视频](#multimodal)
- [六、会话记忆与会话列表持久化](#persistence)
- [七、复习速记](#review)

---

<a id="overview"></a>

## 一、知识总览

| 模块 | 核心内容 | 要解决的问题 |
| :--- | :--- | :--- |
| 知识库 | PDF 上传下载、FileService、向量化入库 | 让模型能“读到”外部 PDF 文档，并检索作答 |
| ChatClient | 注册向量库 `VectorStore`、注册 `Advisor` | 打通“检索 → 注入提示词 → 生成回答”链路 |
| ChatController | 给 `QuestionAnswerAdvisor` 传 `chatId` 过滤条件 | 每个会话只查自己的文档，避免串库 |
| 多模态 | `qwen3.8-omni-flash`、`Media`、消息格式声明 | 支持用户输入图片、音频、视频等非文本内容 |
| 持久化 | 会话记忆、会话列表、Redis Stack 向量库 | 重启后不丢数据，替代内存版 `SimpleVectorStore` |

> 💡 **一句话串起来：** 知识库负责“喂文档”，Advisor 负责“检索 + 记忆”，多模态负责“扩展输入类型”，Redis 负责“持久化不丢”。

---

<a id="knowledge-upload-download"></a>

## 二、知识库：PDF 上传与下载

### 2.1 文件管理 FileService

FileService 是整个知识库文件管理的核心，提供两个方法：

#### `save(String chatId, Resource resource)` —— 保存文件

1. 将上传的文件保存到本地磁盘；
2. 将 `chatId` 与文件路径的映射关系保存到 `Properties` 集合中；
3. 对文件进行**读取 → 切片 → 向量化**，存储到向量数据库。

#### `Resource getFile(String chatId)` —— 获取文件

1. 从 `Properties` 中根据 `chatId` 查询文件路径信息；
2. 基于路径信息构建 `Resource` 并返回。

> [!note] 持久化思路
> `chatId` 与文件路径的映射关系保存在**内存中的 `Properties` 集合**里。为间接实现持久化，在应用启动/停止时，把该映射关系**写入 `properties` 属性文件**，下次启动时再加载回来。

### 2.2 上传与下载接口

| 操作 | 调用链 | 说明 |
| :--- | :--- | :--- |
| 上传 | `PdfController → uploadPdf → fileService.save` | 接收 PDF 并向量化入库 |
| 下载 | `PdfController → download → fileService.getFile` | 根据 `chatId` 取回原文件 |

### 2.3 修改 Spring Boot 默认文件上传大小限制

> [!warning] 默认限制
> Spring Boot 默认**单个文件最大 1MB**，**单次请求所有文件合计最大 10MB**，上传 PDF 很容易超限。

在 `application.yml` 中手动调大：

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 20MB   # 限制单个文件大小上限
      max-request-size: 30MB # 限制整个 request 请求大小上限
```

### 2.4 跨域下载：暴露响应头

为了让前端在跨域场景下能正确拿到下载文件名，需要暴露 `Content-Disposition` 响应头：

```java
@Configuration
public class MvcConfiguration implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .exposedHeaders("Content-Disposition");
    }
}
```

---

<a id="chatclient"></a>

## 三、配置 ChatClient：向量库与 Advisor

### 3.1 注册向量数据库

```java
@Bean
public VectorStore vectorStore(DashScopeEmbeddingModel embeddingModel) {
    return SimpleVectorStore.builder(embeddingModel).build();
}
```

### 3.2 注册 Advisor

```java
@Bean
public ChatClient chatClient(ChatClient.Builder builder, ChatMemory chatMemory, VectorStore vectorStore) {
    return builder
            // 设置默认的 advisors
            .defaultAdvisors(
                    new SimpleLoggerAdvisor(), // 日志记录
                    MessageChatMemoryAdvisor.builder(chatMemory).build(), // 会话记忆
                    QuestionAnswerAdvisor
                            .builder(vectorStore)
                            .searchRequest(SearchRequest.builder().topK(2).build())
                            .build()
            )
            .build();
}
```

> 💡 三个 Advisor 各司其职：`SimpleLoggerAdvisor` 记录日志、`MessageChatMemoryAdvisor` 提供会话记忆、`QuestionAnswerAdvisor` 负责从向量库检索（`topK(2)` 取最相关的 2 条）。

---

<a id="chatcontroller"></a>

## 四、改造 ChatController：会话隔离检索

### 4.1 给 QuestionAnswerAdvisor 添加参数

```java
@RequestMapping(value = "/chat", produces = "text/html;charset=UTF-8")
public Flux<String> chat(String prompt, String chatId) {
    chatHistoryRepository.save("pdf", chatId);
    return chatClient
            .prompt()
            .user(prompt)
            .advisors(a -> a.param(ChatMemory.CONVERSATION_ID, chatId))
            .advisors(a -> a.param(QuestionAnswerAdvisor.FILTER_EXPRESSION, "chat_id == '" + chatId + "'"))
            .stream()
            .content();
}
```

> [!important] 核心作用
> 每次请求查询知识库时都带上 `chatId` 过滤条件，这样就不会查到其它会话上传的文档，实现**会话级数据隔离**。

---

<a id="multimodal"></a>

## 五、多模态：文本、图片、音频、视频

### 5.1 概念

> 多模态指的是用户的**输入形式支持多种类型**，比如文本、图片、音频、视频。

### 5.2 实现步骤

#### ① 选择支持多模态的模型

选用阿里云通义千问的多模态模型：`qwen3.8-omni-flash`。

#### ② 引入阿里的依赖坐标

```xml
<dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-starter-dashscope</artifactId>
</dependency>
```

#### ③ `application.yml` 配置向量模型

```yaml
spring:
  ai:
    dashscope:
      api-key: sk-***（已脱敏）
      chat:
        options:
          model: qwen3.8-omni-flash
          temperature: 0.7
          multi-model: true
```

#### ④ 修改 ChatController

```java
@RequestMapping(value = "/chat", produces = "text/html;charset=UTF-8")
public Flux<String> chat(@RequestParam(defaultValue = "讲个笑话") String prompt,
                         @RequestParam(defaultValue = "1") String chatId,
                         @RequestParam(value = "files", required = false) List<MultipartFile> files) {
    chatHistoryRepository.save("chat", chatId);
    // 1. 解析多媒体（无文件时为空数组）
    Media[] medias = CollectionUtils.isEmpty(files) ? new Media[0] : files.stream()
            .map(file -> new Media(resolveMimeType(file), file.getResource()))
            .toArray(Media[]::new);
    // 2. 构建用户消息；带媒体时必须声明消息格式，否则框架默认按图片处理
    UserMessage.Builder userBuilder = UserMessage.builder().text(prompt);
    if (medias.length > 0) {
        userBuilder.media(List.of(medias))
                .metadata(Map.of(DashScopeApiConstants.MESSAGE_FORMAT,
                        resolveMessageFormat(medias[0].getMimeType())));
    }
    // 3. 流式调用
    return chatClient
            .prompt(new Prompt(userBuilder.build()))
            .advisors(advisorSpec -> advisorSpec.param(ChatMemory.CONVERSATION_ID, chatId)) // 传入会话 id
            .stream() // 异步流式调用，AI 会分片不断返回结果
            .content();
}

// 解析上传文件的 MimeType，"application/octet-stream" 是通用的二进制 MimeType
private MimeType resolveMimeType(MultipartFile file) {
    String contentType = file.getContentType();
    return contentType != null
            ? MimeType.valueOf(contentType)
            : MimeType.valueOf("application/octet-stream");
}

// 根据 MimeType 获取阿里云的消息格式
private MessageFormat resolveMessageFormat(MimeType mimeType) {
    return switch (mimeType.getType()) {
        case "audio" -> MessageFormat.AUDIO;
        case "video" -> MessageFormat.VIDEO;
        default -> MessageFormat.IMAGE;
    };
}
```

> [!tip] 关键细节
> 带媒体时必须通过 `metadata` 声明消息格式（`MESSAGE_FORMAT`），否则框架默认按**图片**处理，音频/视频会识别错误。

---

<a id="persistence"></a>

## 六、会话记忆与会话列表持久化

### 6.1 会话记忆

> 将 `message` 存入 Redis，**key 使用 `chatId`，value 使用 list 类型**。

- 对话过程中不断向 Redis 写入；
- 因为 Spring AI 提供的 `Message` **没有无参构造方法**，既不支持反序列化、也没有实现 `Serializable` 接口，所以需要**自定义 `Msg` 类来封装消息**。

### 6.2 会话列表

> 将 `chatId` 存入 Redis，**key 使用 `type`，value 使用 set 类型**。

| 数据 | Redis Key | Value 类型 |
| :--- | :--- | :--- |
| 会话消息 | `chatId` | list |
| 会话列表 | `type` | set |

### 6.3 使用持久化向量数据库（Redis Stack）

> [!warning] 为什么换
> 之前使用的是**基于内存的 `SimpleVectorStore`** 向量数据库，不支持持久化；现改用 **Redis Stack 向量数据库**。

#### 实现步骤

**① 安装 Redis Stack**

**② 引入 `vector-store-redis` 依赖**

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-vector-store-redis</artifactId>
</dependency>
```

**③ `application.yml` 配置**

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 30MB
      max-request-size: 40MB
  ai:
    vectorstore:
      redis:
        index-name: spring-ai-index
        initialize-schema: true
        prefix: "doc:"
  data:
    redis:
      host: 192.168.150.102
      port: 6379
```

**④ 配置类定义 `RedisVectorStore`**

```java
/**
 * 手动定义 RedisVectorStore，声明 chat_id 为 TAG 类型的元数据字段。
 * 只有声明过的字段，才能作为过滤表达式的 key（Redis 安全校验要求）。
 */
@Bean
public RedisVectorStore vectorStore(DashScopeEmbeddingModel embeddingModel, JedisConnectionFactory jedisConnectionFactory) {
    JedisClientConfig clientConfig = DefaultJedisClientConfig.builder()
            .ssl(jedisConnectionFactory.isUseSsl())
            .clientName(jedisConnectionFactory.getClientName())
            .timeoutMillis(jedisConnectionFactory.getTimeout())
            .password(jedisConnectionFactory.getPassword())
            .build();
    JedisPooled jedisPooled = new JedisPooled(
            new HostAndPort(jedisConnectionFactory.getHostName(), jedisConnectionFactory.getPort()),
            clientConfig);
    return RedisVectorStore.builder(jedisPooled, embeddingModel)
            .indexName("spring-ai-index")
            .prefix("doc:")
            .initializeSchema(true)
            .metadataFields(RedisVectorStore.MetadataField.tag("chat_id"))
            .build();
}
```

> [!important] 元数据字段声明
> 只有通过 `.metadataFields(...)` **声明过的字段**，才能作为过滤表达式的 key 使用（Redis 安全校验要求），所以这里必须显式声明 `chat_id` 为 TAG 类型。

---

<a id="review"></a>

## 七、复习速记

- **知识库三步走：** 存磁盘 → 存映射（`Properties`）→ 向量化入库（切片 + Embedding）。
- **检索链路：** `QuestionAnswerAdvisor` + `VectorStore`，`topK(2)` 取最相关片段。
- **会话隔离：** 给 Advisor 传 `FILTER_EXPRESSION`，按 `chat_id` 过滤，防串库。
- **多模态四步走：** 选模型 → 加依赖 → 配 yml（`multi-model: true`）→ 改 Controller（`Media` + 消息格式声明）。
- **消息格式：** `audio` → AUDIO、`video` → VIDEO、其余 → IMAGE。
- **持久化：** 会话记忆用 `chatId` + list；会话列表用 `type` + set；向量库用 Redis Stack。
- **自定义 `Msg` 类的原因：** Spring AI 的 `Message` 无参构造缺失、未实现 `Serializable`，无法直接序列化存 Redis。
