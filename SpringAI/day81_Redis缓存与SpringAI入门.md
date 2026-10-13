# Day81｜Redis 缓存与 Spring AI 入门

> **内容来源：** `day81.xmind`（含代码截图）  
> **学习主线：** Redis 缓存问题 → 大模型基础 → 部署与调用 → Spring AI 集成  
> **整理说明：** 按原文归纳，代码截图已转为文本；API Key 已脱敏。代码中的模型名称沿用课堂示例，使用时以实际服务支持的名称为准。

## 📑 目录

- [一、Redis 缓存：穿透、雪崩与击穿](#一redis-缓存穿透雪崩与击穿)
- [二、AI 与大模型基础概念](#二ai-与大模型基础概念)
- [三、大模型部署](#三大模型部署)
- [四、大模型接口调用](#四大模型接口调用)
- [五、Spring AI 入门实践](#五spring-ai-入门实践)
- [六、复习速记](#六复习速记)

---

## 一、Redis 缓存：穿透、雪崩与击穿

### 1.1 三类问题对比

| 问题 | 触发场景 | 数据库压力来源 | 主要应对方式 |
| :--- | :--- | :--- | :--- |
| **缓存穿透** | 查询数据库中不存在的数据 | 缓存和数据库都查不到，每次请求仍访问数据库 | 缓存空值、布隆过滤器 |
| **缓存雪崩** | 大量 Key 同时过期，或 Redis 服务不可用 | 大量请求集中访问数据库 | 随机 TTL、高可用部署、多级缓存 |
| **缓存击穿** | 某个热点 Key 失效 | 同一热点数据的大量并发请求访问数据库 | 互斥锁、逻辑过期 |

> 💡 **记忆方法：** 穿透＝数据不存在；雪崩＝大量缓存失效；击穿＝热点缓存失效。

### 1.2 缓存穿透

#### 方案一：缓存空值

当数据库查不到数据时，仍向 Redis 写入空值标记，并设置较短的 TTL。后续请求命中该标记后，可直接返回“无数据”。

| 优点 | 缺点 |
| :--- | :--- |
| 实现简单 | 额外占用缓存空间 |
| 减少重复查询不存在数据的请求 | 数据新增后，空值缓存过期前可能出现短暂不一致 |

#### 方案二：布隆过滤器

在查询 Redis 前，先用布隆过滤器判断请求的数据是否可能存在。

```text
请求 → 布隆过滤器
         ├─ 判断不存在 → 直接拒绝或返回无数据
         └─ 判断可能存在 → 查询 Redis → 未命中时查询数据库
```

- **原理：** 通过位数组（Bit 数组）和多个哈希函数进行判断。
- **优点：** 内存占用较低。
- **缺点：** 实现和维护较复杂，存在一定概率的误判。
- **关键理解：** “可能存在”不代表一定存在，仍需继续查询。

**原文结论：** 实际开发中可混合使用布隆过滤器与空值缓存。

### 1.3 缓存雪崩

**原因：** 大量 Key 在同一时间失效，或 Redis 宕机，导致请求集中访问数据库。

**应对措施：**

1. **随机过期时间：** 在基础 TTL 上增加随机偏移，分散 Key 的过期时刻。
2. **Redis 高可用：** 通过主从复制、哨兵或 Cluster 等部署方式提升可用性。
3. **多级缓存：** 加入本地 JVM 缓存，在 Redis 不可用时承担部分请求。
4. **参数校验：** 提前拦截明显非法的参数，例如 `id <= 0`，减少无效请求。

> **分类补充：** 参数校验是通用防护措施，也有助于减少缓存穿透；它不能直接解决正常 Key 集中过期的问题。

#### Redis 部署方式

| 方式 | 核心作用 | 要点 |
| :--- | :--- | :--- |
| 主从复制 | 数据复制、支持读写分离 | 主节点负责写入；从节点可承担读取；课堂示例为一主多从 |
| 哨兵机制 | 监控与故障转移 | 主节点宕机后，从从节点中选出新的主节点 |
| Cluster 集群 | 数据分片、扩展存储容量 | 将数据分布到多个节点，提升整体容量 |

**注意：** 主从复制本身不等于自动故障转移；哨兵负责监控和切换。

### 1.4 缓存击穿

**原因：** 某个热点 Key 突然失效，大量请求同时触发数据库查询和缓存重建。

#### 方案一：互斥锁

用分布式互斥锁限制缓存重建，同一热点数据由一个请求线程查询数据库并重建缓存，其余线程等待或重试。

- **优点：** 重建流程直观，避免多个线程同时回源。
- **缺点：** 等待和重试增加延迟，影响并发性能。

> **表述补充：** 原文将其优点概括为“强一致”。互斥锁主要保证缓存重建互斥；整体数据一致性还取决于数据库更新与缓存更新策略。

#### 方案二：逻辑过期

不依靠 Key 的 TTL 删除热点缓存，而是在 Value 中保存逻辑过期时间。

```text
读取缓存 → 检查逻辑过期时间
             ├─ 未过期 → 返回缓存数据
             └─ 已过期 → 返回旧数据，并异步重建缓存
```

- **优点：** 请求可直接返回旧数据，减少等待，响应性能较好。
- **缺点：** 实现较复杂，允许短暂返回旧数据，偏向最终一致性。

| 对比维度 | 互斥锁重建 | 逻辑过期 |
| :--- | :--- | :--- |
| 请求体验 | 可能等待缓存重建 | 可先返回旧数据 |
| 数据时效 | 倾向等待重建后的数据 | 允许短暂旧数据 |
| 实现复杂度 | 相对较低 | 相对较高 |

---

## 二、AI 与大模型基础概念

### 2.1 术语速览

| 术语 | 中文名称 | 核心含义 |
| :--- | :--- | :--- |
| AI | 人工智能 | 让机器具备感知、学习、推理和决策等能力 |
| LLM | 大语言模型 | 在海量文本上训练的神经网络模型，擅长理解和生成语言 |
| Token | 词元 | 模型处理文本的基本单位，可对应一个词、字或文本片段 |
| Prompt | 提示词 | 输入给模型的内容，用于说明任务、背景和要求 |
| RAG | 检索增强生成 | 检索外部资料，将资料和问题一起交给模型生成答案 |
| Function Calling | 函数／工具调用 | 模型输出结构化调用信息，由外部程序执行工具 |
| Agent | 智能体 | 能规划、决策、调用工具并多轮执行任务的大模型应用 |
| Fine-tuning | 微调 | 在预训练模型基础上，用专属数据继续训练并更新权重 |

### 2.2 Token、编码与解码

大模型通过分词器将文本转换为 Token ID，再基于这些数字进行计算。

```text
输入文本 → 分词器 → Token 序列 → Token ID 序列
                                      ↓
                                   模型计算
                                      ↓
输出文本 ← 解码 ← 模型生成的 Token ID 序列
```

- **词汇表：** 保存文本片段与数字编号的映射。
- **Token：** 词汇表中的文本片段。
- **Token ID：** 该片段对应的数字编号。
- **编码：** 将文本转换为 Token ID。
- **解码：** 将生成的 Token ID 转回文本。

### 2.3 RAG、工具调用、Agent 与微调

| 能力 | 主要解决的问题 | 基本流程 |
| :--- | :--- | :--- |
| RAG | 为回答提供外部资料 | 用户问题 → 检索知识库 → 资料与问题组成 Prompt → 生成答案 |
| Function Calling | 让应用执行外部操作 | 模型生成函数名与参数 → 程序执行 → 结果交回模型 → 生成答案 |
| Agent | 完成需要多步骤执行的目标 | 规划 → 执行／调用工具 → 观察结果 → 继续决策 |
| Fine-tuning | 适配任务、风格或领域 | 准备专属数据 → 继续训练 → 更新模型权重 |

> **理解重点：** 工具调用由外部程序执行；RAG 提供检索资料；微调通过训练改变模型权重。

---

## 三、大模型部署

### 3.1 本地部署：Ollama

**课堂流程：**

1. 下载并安装 Ollama。
2. 在模型广场选择模型，获取运行命令。
3. 执行命令，下载并运行模型。

```bash
ollama run deepseek-r1:1.5b
```

- `1.5b` 表示约 **15 亿参数**。
- 更大的模型通常需要更多内存、显存等硬件资源。
- 模型效果还受训练数据、架构和任务影响，不能只看参数量。
- 本地部署可使数据在本地处理，便于掌握数据流向。

### 3.2 云端服务

原文举例：**阿里云百炼**。

| 对比项 | 本地部署 | 云端服务 |
| :--- | :--- | :--- |
| 计算资源 | 使用本地硬件 | 使用服务商资源 |
| 部署工作 | 安装工具并下载模型 | 开通服务并配置访问凭证 |
| 数据处理位置 | 本地环境 | 服务商环境 |
| 调用凭证 | 原文中的本机 Ollama 示例无需 API Key | 通常需要 API Key |

> **表述补充：** 原文将本地／云端概括为“安全性高／低”。更准确的比较是数据处理位置和控制边界不同，安全性还取决于配置与服务条款。

---

## 四、大模型接口调用

### 4.1 调用本地 Ollama

原图中的请求使用 Ollama 原生聊天接口。

```http
POST http://localhost:11434/api/chat
Content-Type: application/json
```

```json
{
  "model": "deepseek-r1:1.5b",
  "messages": [
    {
      "role": "user",
      "content": "你是谁"
    }
  ],
  "stream": false
}
```

- `model`：指定本地模型。
- `messages`：传入对话消息。
- `stream: false`：使用非流式响应。
- 该本机示例无需 API Key，回复内容位于 `message.content`。

### 4.2 调用 DeepSeek

原图中的请求使用兼容 OpenAI 格式的聊天接口，并需要 API Key。

```http
POST https://api.deepseek.com/chat/completions
Content-Type: application/json
Authorization: Bearer <YOUR_API_KEY>
```

```json
{
  "model": "deepseek-flash",
  "messages": [
    {
      "role": "system",
      "content": "你是一个智能助手，名字叫做小团团"
    },
    {
      "role": "user",
      "content": "你是谁"
    }
  ]
}
```

原图中回复内容位于 `choices[0].message.content`。`deepseek-flash` 沿用原文示例，本文未验证该模型名称当前是否可用。

### 4.3 Prompt 消息角色

| 角色 | 含义 | 用途 |
| :--- | :--- | :--- |
| `system` | 系统提示词 | 定义助手身份、职责和回答风格 |
| `user` | 用户消息 | 传入用户的问题或任务 |
| `assistant` | 模型回复 | 表示模型回答，或记录历史模型消息 |

**原文要点：** DeepSeek、阿里云百炼等服务提供兼容 OpenAI API 格式的调用方式；具体接口地址、模型名称和支持能力需按所选服务配置。

---

## 五、Spring AI 入门实践

### 5.1 引入 Maven 依赖

先通过 **BOM** 统一管理 Spring AI 组件版本，再引入 OpenAI 模型 Starter。

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>${spring-ai.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-openai</artifactId>
    </dependency>
</dependencies>
```

> 原文件未给出 `spring-ai.version` 的具体版本；需在项目属性中指定与项目兼容的版本。

### 5.2 配置 application.yml

```yaml
spring:
  ai:
    openai:
      base-url: https://api.deepseek.com
      api-key: ${DEEPSEEK_API_KEY}
      chat:
        options:
          temperature: 0.7  # 控制生成结果的随机性，较小值通常更确定
          model: deepseek-flash
```

| 配置项 | 作用 |
| :--- | :--- |
| `base-url` | 模型服务的基础地址 |
| `api-key` | 访问凭证，此处改为从环境变量读取 |
| `temperature` | 调节生成随机性 |
| `model` | 使用的模型名称，需匹配服务端支持的模型 |

### 5.3 装配 ChatClient

按原文示例，引入 Starter 并配置后，Spring Boot 自动装配 `OpenAiChatModel` 和 `ChatClient.Builder`。通过 Builder 创建 `ChatClient` Bean。

```java
@Configuration
public class SpringAIConfig {

    @Bean
    public ChatClient chatClient(ChatClient.Builder builder) {
        return builder.build();
    }
}
```

**核心关系：** `ChatClient` 基于 `ChatModel`，提供更方便的链式调用方式。

### 5.4 同步非流式调用

等待模型完整生成答案后，一次性返回文本。

```java
@RequestMapping("/chat")
public String chat(
        @RequestParam(defaultValue = "讲个笑话") String prompt) {
    return chatClient
            .prompt()
            .user(prompt)
            .call()
            .content();
}
```

调用链：`prompt()` 创建请求 → `user()` 设置用户消息 → `call()` 发起同步调用 → `content()` 提取文本。

### 5.5 流式调用

模型生成内容时分片返回，示例返回类型为 Reactor 的 `Flux<String>`。

```java
@RequestMapping(value = "/chat/stream", produces = "text/html;charset=UTF-8")
public Flux<String> chatStream(
        @RequestParam(defaultValue = "讲个笑话") String prompt) {
    return chatClient
            .prompt()
            .user(prompt)
            .stream()
            .content();
}
```

- `.stream()`：获取流式响应。
- `.content()`：提取各分片的文本内容。
- 原文强调设置响应类型与 UTF-8 编码，避免中文乱码。
- 为方便与非流式示例并列，整理时将此路径改为 `/chat/stream`；原文两段示例均使用 `/chat`。

| 对比项 | 非流式 | 流式 |
| :--- | :--- | :--- |
| 调用方法 | `.call()` | `.stream()` |
| 返回类型 | `String` | `Flux<String>` |
| 展示方式 | 完整生成后一次性展示 | 逐段展示模型生成内容 |

> 以上为核心方法片段，使用时需要在控制器中注入 `ChatClient` 并添加相应导入。

### 5.6 设置默认系统提示词

通过 `.defaultSystem()` 设置助手的默认身份、职责和回复风格。

```java
@Bean
public ChatClient chatClient(ChatClient.Builder builder) {
    return builder
            .defaultSystem("""
                    你是IT教育机构的在线客服助手。
                    你的职责：解答学员关于课程、报名、上课时间、收费、资料、上课方式的咨询。
                    请以友好、乐于助人、愉快的方式解答学生的各种问题。
                    """)
            .build();
}
```

该示例用于替换前面的基础 `chatClient` Bean 方法。

**系统提示词的组成：** 身份定位 + 职责范围 + 回答风格。

---

## 六、复习速记

- **缓存穿透：** 不存在的数据反复回源 → 空值缓存、布隆过滤器。
- **缓存雪崩：** 大量缓存失效或 Redis 不可用 → 随机 TTL、高可用、多级缓存。
- **缓存击穿：** 热点 Key 失效 → 互斥重建或逻辑过期。
- **本地模型：** Ollama 运行模型，课堂示例使用 `/api/chat`。
- **云端调用：** 配置接口地址、模型名称和 API Key。
- **RAG：** 检索资料后再生成；**工具调用：** 程序执行模型提出的调用；**微调：** 训练并更新权重。
- **Spring AI：** BOM → Starter → YAML → `ChatClient` → 调用。
- **调用口诀：** `call()` 等完整结果，`stream()` 分片返回，`defaultSystem()` 设置默认系统提示词。