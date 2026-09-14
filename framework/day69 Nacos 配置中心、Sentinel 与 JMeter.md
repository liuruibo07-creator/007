# Day 69 学习笔记：Nacos 配置中心、Sentinel 与 JMeter

> 本篇整理自 `day69.xmind`，涵盖 **共享配置、配置热更新、Gateway 动态路由、服务雪崩与保护、Sentinel、JMeter 性能测试**。

---

## 📚 目录

- [一、Nacos 共享配置](day69%20Nacos%20配置中心、Sentinel%20与%20JMeter.md#一nacos-共享配置)
- [二、业务配置热更新](day69%20Nacos%20配置中心、Sentinel%20与%20JMeter.md#二业务配置热更新)
- [三、Gateway 动态路由](day69%20Nacos%20配置中心、Sentinel%20与%20JMeter.md#三gateway-动态路由)
- [四、服务雪崩与保护方案](day69%20Nacos%20配置中心、Sentinel%20与%20JMeter.md#四服务雪崩与保护方案)
- [五、Sentinel](day69%20Nacos%20配置中心、Sentinel%20与%20JMeter.md#五sentinel)
- [六、JMeter 性能测试](day69%20Nacos%20配置中心、Sentinel%20与%20JMeter.md#六jmeter-性能测试)
- [七、知识点速查](day69%20Nacos%20配置中心、Sentinel%20与%20JMeter.md#七知识点速查)

---

## 一、Nacos 共享配置

多个微服务通常存在重复配置。可以将公共内容抽取到 Nacos 配置中心，实现集中维护和复用。

### 1.1 抽取公共配置

课程示例将配置拆分为：

| Data ID | 内容 |
| --- | --- |
| `shared-log.yml` | 公共日志配置 |
| `shared-feign.yml` | OpenFeign 公共配置 |
| `shared-jdbc.yml` | 数据源公共配置 |

### 1.2 引入配置中心依赖

```xml
<!-- Nacos 配置中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>

<!-- 读取 bootstrap 配置 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

### 1.3 添加 bootstrap.yml

`bootstrap.yml` 早于 `application.yml` 加载，主要用于配置 Nacos 配置中心连接信息。

```yaml
spring:
  application:
    name: order-service
  cloud:
    nacos:
      # config 与 discovery 默认读取此统一地址
      server-addr: 192.168.150.102:8848
      config:
        file-extension: yml
        shared-configs:
          - data-id: shared-jdbc.yml
          - data-id: shared-log.yml
          - data-id: shared-feign.yml
```

### 1.4 Data ID 命名规则

微服务自身配置的名称由三部分组成：

```text
${spring.application.name}-${spring.profiles.active}.${file-extension}
```

| 部分 | 含义 |
| --- | --- |
| `spring.application.name` | 微服务名称 |
| `spring.profiles.active` | 当前环境标识，可选，如 `dev`、`test` |
| `file-extension` | 配置文件后缀，如 `yml` |

例如，服务名为 `order-service`、环境为 `dev`、后缀为 `yml` 时，对应：

```text
order-service-dev.yml
```

### 1.5 精简 application.yml

公共配置移到 Nacos 后，本地只保留端口和微服务特有配置：

```yaml
server:
  port: 8080

# 当前微服务自定义的数据源参数
cloud:
  datasource:
    host: localhost
    port: 3306
    dbname: cloud_order
    username: root
    password: 1234
```

这些自定义属性的名称需要与 `shared-jdbc.yml` 中使用的占位符相对应。

> 🔐 数据库密码等敏感配置不宜直接写入代码仓库，生产环境应使用更安全的配置与密钥管理方案。

---

## 二、业务配置热更新

配置热更新是指：在 Nacos 修改配置后，微服务无需重启即可读取新值。

### 2.1 Nacos 侧

在 Nacos 配置中心新建与微服务对应的配置文件，Data ID 按服务名、环境和扩展名组成。

### 2.2 微服务侧的两种方式

#### 方式一：`@Value` + `@RefreshScope`

```java
@RefreshScope
@RestController
public class OrderController {

    @Value("${order.timeout}")
    private Integer timeout;
}
```

`@RefreshScope` 使 Bean 在配置变化时重新读取通过 `@Value` 注入的属性。

#### 方式二：`@ConfigurationProperties`

```java
@Data  
@Component  
@ConfigurationProperties(prefix = "order")  
public class OrderProperties {  
    private Integer count;  
}
```

| 方式 | 适合场景 |
| --- | --- |
| `@Value` + `@RefreshScope` | 少量、零散的配置项 |
| `@ConfigurationProperties` | 一组结构化配置，类型安全且便于统一管理 |

---

## 三、Gateway 动态路由

静态路由写在本地 YAML 中，修改后通常需要重启网关。动态路由把规则放入 Nacos，网关监听配置变化并实时更新路由。

### 3.1 Nacos 侧

在 Nacos 中使用 JSON 格式管理路由配置，因为 JSON 便于 Java 代码解析为路由对象。

示例结构：

```json
[
  {
    "id": "user-service",
    "uri": "lb://user-service",
    "predicates": [
      {
        "name": "Path",
        "args": {
          "_genkey_0": "/user/**"
        }
      }
    ]
  }
]
```

### 3.2 网关引入依赖

网关添加 Nacos Config 和 Bootstrap 依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

### 3.3 网关 bootstrap.yml

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    nacos:
      server-addr: 192.168.150.102:8848
      config:
        file-extension: yml
        shared-configs:
          - data-id: shared-log.yml
```

### 3.4 网关 application.yml

```yaml
server:
  port: 10086

spring:
  main:
    web-application-type: reactive

cloud:
  whitePaths:
    - /user/login
    - /user1/**
```

- `web-application-type: reactive`：将 Web 应用类型设置为响应式。
- `whitePaths`：定义无需身份校验的放行路径。

### 3.5 添加路由监听器

动态路由监听器主要使用两个对象：

| 对象 | 作用 |
| --- | --- |
| `NacosConfigManager` | Nacos 配置管理器，可获取 `ConfigService` |
| `RouteDefinitionWriter` | 路由定义写入器，可以新增或删除 `RouteDefinition` |

更新流程：

```text
Nacos 路由 JSON 发生变化
        ↓
ConfigService 监听到变更
        ↓
Java 解析 JSON 为 RouteDefinition
        ↓
删除旧路由并写入新路由
        ↓
发布路由刷新事件
```

---

## 四、服务雪崩与保护方案

### 4.1 服务雪崩

微服务调用链中某个服务出现故障，故障不断向上游扩散，最终导致整条调用链上的服务相继异常，这种现象称为 **服务雪崩**。

```text
服务 A → 服务 B → 服务 C（故障或超时）
   ↑         ↑          ↓
线程堆积 ← 请求堆积 ← 持续阻塞
```

### 4.2 常用保护方案

| 方案 | 作用 |
| --- | --- |
| 请求限流 | 控制进入服务的请求量，防止突发流量打垮服务 |
| 线程隔离 | 为不同下游调用设置线程上限，避免单个下游耗尽全部线程 |
| 服务熔断 | 下游大量失败或超时时暂时切断调用链路，快速失败 |
| Fallback 降级 | 在限流、熔断或异常时返回预设兜底结果 |

四种方案相互配合：限流控制入口流量，隔离限制故障影响范围，熔断阻止继续调用异常服务，降级保证用户仍能得到可接受的响应。

---

## 五、Sentinel

### 5.1 概述

Sentinel 是阿里开源的服务保护组件，主要由两部分组成：

1. **Sentinel Dashboard**：可视化控制台，用于查看资源、配置规则和监控运行状态。
2. **Sentinel Client**：集成在各微服务中的客户端，负责执行限流、熔断等规则。

### 5.2 规则管理流程

```text
运维人员定义规则
        ↓
Sentinel Dashboard
        ↓
规则存储（Nacos / ZooKeeper / Memory 等）
        ↓
规则推送到各微服务中的 Sentinel Client
        ↓
客户端保护对应资源
```

### 5.3 部署 Dashboard

#### 方式一：JAR 部署

Sentinel Dashboard 本质上是一个 Spring Boot 工程，官方提供可执行 JAR：

```bash
java -jar sentinel-dashboard.jar --server.port=8090
```

#### 方式二：Docker 部署

也可以通过 Docker 镜像运行 Dashboard，端口需与微服务配置中的控制台地址一致。

### 5.4 改造微服务

引入依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

配置控制台：

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8090
      eager: true
      http-method-specify: true
```

| 配置 | 说明 |
| --- | --- |
| `transport.dashboard` | Sentinel Dashboard 地址 |
| `eager: true` | 应用启动时立即初始化 Sentinel；默认懒加载 |
| `http-method-specify: true` | 资源名包含 HTTP 方法，便于区分 RESTful 接口 |

---

## 六、JMeter 性能测试

### 6.1 概述

JMeter 是一款性能测试工具，可以模拟大量并发请求，观察接口吞吐量、响应时间和错误率。

### 6.2 安装与配置

- 使用绿色版本，解压即可运行。
- 建议安装到不包含中文的目录。
- 将字符编码修改为 UTF-8。
- 将界面语言设置为中文简体。

### 6.3 基本使用步骤

1. 新建测试计划。
2. 为测试计划添加线程组。
3. 在线程组中添加 HTTP 请求。
4. 为 HTTP 请求添加监听器：
   - 查看结果树
   - 汇总报告
5. 如果接口需要认证或自定义请求头，再添加 HTTP 请求头管理器。

```text
测试计划
└── 线程组
    ├── HTTP 请求
    │   └── HTTP 请求头管理器（按需）
    ├── 查看结果树
    └── 汇总报告
```

> 💡 “查看结果树”适合调试少量请求；高并发压测时应减少重量级监听器，避免 JMeter 自身消耗过多资源而影响测试结果。

---

## 七、知识点速查

| 知识点 | 核心内容 |
| --- | --- |
| Nacos Config | 集中管理微服务配置 |
| `shared-configs` | 引入多个微服务共用的配置文件 |
| `bootstrap.yml` | 优先加载配置中心连接信息 |
| `@RefreshScope` | 让 `@Value` 注入的配置支持刷新 |
| `@ConfigurationProperties` | 绑定一组结构化配置 |
| 动态路由 | 从 Nacos 获取并实时更新 Gateway 路由 |
| `NacosConfigManager` | 获取 Nacos 的 `ConfigService` |
| `RouteDefinitionWriter` | 新增或删除网关路由定义 |
| 服务雪崩 | 单点故障沿调用链扩散为系统性故障 |
| Sentinel | 提供限流、隔离、熔断和降级等保护能力 |
| `eager: true` | Sentinel 随应用启动立即初始化 |
| JMeter | 模拟并发请求并生成性能测试结果 |

---

> ✅ **本节重点：** 掌握使用 Nacos 抽取共享配置和实现热更新的方法；理解 Gateway 动态路由监听过程；区分限流、线程隔离、熔断与降级；能够接入 Sentinel Dashboard，并使用 JMeter 建立基础性能测试计划。
