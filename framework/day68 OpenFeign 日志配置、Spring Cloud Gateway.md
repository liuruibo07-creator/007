 # Day 68 学习笔记：OpenFeign 日志与微服务网关

> 本篇整理自 `day68.xmind`，主要涵盖 **OpenFeign 日志配置、Spring Cloud Gateway、路由断言、过滤器、用户登录、JWT 令牌与全局过滤器**。

---

## 📚 目录

- [一、OpenFeign 日志配置](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#一openfeign-日志配置)
- [二、Gateway 网关概述](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#二gateway-网关概述)
- [三、搭建网关服务](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#三搭建网关服务)
- [四、配置路由规则](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#四配置路由规则)
- [五、路由断言](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#五路由断言)
- [六、路由过滤器](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#六路由过滤器)
- [七、用户登录接口](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#七用户登录接口)
- [八、JWT 登录令牌](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#八jwt-登录令牌)
- [九、身份校验与全局过滤器](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#九身份校验与全局过滤器)
- [十、知识点速查](day68%20OpenFeign%20日志配置、Spring%20Cloud%20Gateway.md#十知识点速查)

---

## 一、OpenFeign 日志配置

开启 OpenFeign 日志后，可以查看远程调用时的请求与响应信息，便于排查接口地址、参数、状态码和响应内容等问题。

### 1.1 注册日志级别对象

将 OpenFeign 的日志级别设置为 `FULL`：

```java
@Bean
public Logger.Level feignLoggerLevel() {
    return Logger.Level.FULL;
}
```

`Logger.Level.FULL` 表示记录请求和响应的全部报文信息，包括请求方法、URL、请求头、请求体、响应状态和响应体。

### 1.2 设置 Feign 接口包的日志级别

在 `application.yml` 中，将 Feign 接口所在包设置为 `DEBUG`：

```yaml
logging:
  level:
    com.edu.apis.clients: DEBUG
```

只有同时配置 OpenFeign 的日志级别和对应包的 `DEBUG` 级别，相关日志才能正常输出。

### 1.3 与 Logback 的关系

实际项目通常会引入 `logback.xml`，其中主要配置：

- `appender`：日志的输出位置和输出方式，如控制台或日志文件。
- `logger`：为指定包或类设置日志级别。
- `root`：全局根日志记录器及其默认级别。

Spring Boot 的 `logging.level` 也可以自定义日志级别，其配置优先级高于 `logback.xml` 中对应的级别设置。

> ⚠️ `FULL` 会产生大量日志，并可能记录请求头或业务数据，适合开发和排错阶段；生产环境应谨慎启用。

---

## 二、Gateway 网关概述

Spring Cloud Gateway 是整个微服务集群的 **统一入口**。

### 2.1 核心作用

1. **路由转发：** 根据请求路径等条件，将请求转发到对应的微服务。
2. **身份校验：** 在请求进入内部服务之前统一检查登录状态或令牌。

```text
客户端请求
    ↓
Gateway 网关（统一入口）
    ├── /user/**  → user-service
    └── /order/** → order-service
```

网关可以把路由、认证等公共逻辑集中处理，避免每个微服务重复实现。

---

## 三、搭建网关服务

### 3.1 引入依赖

```xml
<!-- Gateway 网关 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

<!-- Nacos 服务注册与发现 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<!-- LoadBalancer 负载均衡 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

各依赖的作用：

| 依赖 | 作用 |
| --- | --- |
| `spring-cloud-starter-gateway` | 提供网关、路由、断言和过滤器能力 |
| `spring-cloud-starter-alibaba-nacos-discovery` | 让网关从 Nacos 获取服务实例 |
| `spring-cloud-starter-loadbalancer` | 在同一服务的多个实例之间进行负载均衡 |

### 3.2 基础配置

```yaml
server:
  port: 10086

spring:
  application:
    name: cloud-gateway
  cloud:
    nacos:
      server-addr: 192.168.150.102:8848
```

- 网关监听 `10086` 端口。
- 网关服务名为 `cloud-gateway`。
- 网关连接 `192.168.150.102:8848` 上的 Nacos。

---

## 四、配置路由规则

路由规则在 `application.yml` 中配置。每个中横线 `-` 表示一条路由，一条路由由唯一标识、目标地址和断言条件等部分组成。

### 4.1 完整示例

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/user/**

        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/order/**
            - Header=aaa,bbb
```

### 4.2 配置说明

| 配置项 | 说明 |
| --- | --- |
| `id` | 路由的唯一标识，通常使用服务名称 |
| `uri` | 请求要转发到的目标地址 |
| `lb://` | 使用负载均衡方式选择服务实例 |
| `user-service` | 在注册中心登记的目标服务名称 |
| `predicates` | 当前路由需要满足的断言条件 |

当请求路径匹配 `/user/**` 时，网关会从注册中心获取 `user-service` 的实例，再通过负载均衡选择一个实例并转发请求。

---

## 五、路由断言

`predicates` 表示路由条件。只有请求满足断言，网关才会使用对应路由。

### 5.1 Path 断言

```yaml
predicates:
  - Path=/user/**
```

表示请求路径匹配 `/user/**` 时，将请求转发到当前路由的目标服务。

### 5.2 Header 断言

```yaml
predicates:
  - Header=aaa,bbb
```

表示请求头需要包含名为 `aaa` 的字段，并且其值与 `bbb` 匹配。

### 5.3 多个断言之间的关系

一条路由配置多个断言时，各条件之间是 **并且（AND）** 的关系：

```yaml
predicates:
  - Path=/order/**
  - Header=aaa,bbb
```

请求必须同时满足路径条件和请求头条件，才能匹配该路由。

> 📝 课程导图中记录了 12 类断言。具体数量和可用类型可能随 Spring Cloud Gateway 版本变化，应以项目版本文档为准。

---

## 六、路由过滤器

`GatewayFilter` 可以在请求被转发前或响应返回后，对请求和响应进行处理。

常见用途包括：

- 添加、删除或修改请求头。
- 添加、删除或修改响应头。
- 修改请求路径。
- 记录请求日志。
- 限流、鉴权或处理跨域请求。

### 6.1 过滤器执行流程

```text
客户端
  ↓
前置过滤逻辑（pre）
  ↓
目标微服务
  ↓
后置过滤逻辑（post）
  ↓
客户端
```

课程导图记录了 33 个路由过滤器以及 1 个默认过滤器。实际可用的过滤器工厂数量同样可能随框架版本变化。

---

## 七、用户登录接口

用户登录功能的实现流程：

1. 修改用户表，增加密码字段。
2. 修改用户实体类，增加密码属性。
3. 定义 Controller 登录接口。
4. 定义 Service 登录业务。
5. 定义 Mapper 查询方法。

### 7.1 Controller 层

登录接口的四个要素：

- 请求方式：`POST`
- 请求路径：`/user/login`
- 请求参数：JSON 格式的 `username` 和 `password`，封装为 DTO
- 响应数据：`id`、`username`、`token`，封装为 VO

```java
@PostMapping("/login")
public UserVO login(@RequestBody UserLoginDTO userLoginDTO) {
    return userService.login(userLoginDTO);
}
```

### 7.2 Service 层

```java
public UserVO login(UserLoginDTO userLoginDTO) {
    // 1. 判断用户名是否存在
    User user = userMapper.selectByUsername(userLoginDTO.getUsername());
    if (user == null) {
        throw new RuntimeException("用户名不存在");
    }

    // 2. 判断密码是否正确
    if (!user.getPassword().equals(userLoginDTO.getPassword())) {
        throw new RuntimeException("密码错误");
    }

    // 3. 登录成功，生成 JWT
    Map<String, Object> claims = new HashMap<>();
    claims.put("userId", user.getId());
    String token = JwtUtil.createJWT("baby", 3600000, claims);

    // 4. 封装 VO 并返回
    return UserVO.builder()
            .id(user.getId())
            .username(user.getUsername())
            .token(token)
            .build();
}
```

### 7.3 Mapper 层

```java
@Select("select * from tb_user where username = #{username}")
User selectByUsername(String username);
```

> 🔐 上述代码来自课程示例。实际项目中不应以明文保存或直接比较密码，应使用 BCrypt 等安全算法保存密码摘要；JWT 密钥也不应硬编码在源代码中，应通过安全配置或密钥管理服务提供。

---

## 八、JWT 登录令牌

登录验证成功后，服务生成 JWT 并将其放入 `UserVO` 返回给客户端。

### 8.1 JWT 的组成

JWT 本质上是一个字符串，通常由英文句点分隔为三部分：

```text
Header.Payload.Signature
```

| 部分 | 作用 |
| --- | --- |
| Header | 描述令牌类型、签名算法等元数据 |
| Payload | 保存声明信息，如用户 ID |
| Signature | 使用密钥生成签名，用于校验令牌是否被篡改 |

示例中将用户 ID 放入 Payload：

```java
Map<String, Object> claims = new HashMap<>();
claims.put("userId", user.getId());
```

生成有效期为 3,600,000 毫秒（1 小时）的令牌：

```java
String token = JwtUtil.createJWT("baby", 3600000, claims);
```

---

## 九、身份校验与全局过滤器

网关可以通过自定义全局过滤器统一处理身份校验，使认证逻辑在请求进入各个微服务之前执行。

### 9.1 创建全局过滤器

自定义过滤器需要：

1. 使用 `@Component` 注册为 Spring Bean。
2. 实现 `GlobalFilter` 接口。
3. 实现 `filter()` 方法，编写请求放行前后的逻辑。

课程配图中的过滤器骨架：

```java
@Component
@Slf4j
public class UserLoginGlobalFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange,
                             GatewayFilterChain chain) {
        log.info("UserLoginGlobalFilter.pre 放行前代码...");

        return chain.filter(exchange)
                .doFinally(signalType -> {
                    HttpStatus status = exchange.getResponse().getStatusCode();

                    if (signalType == SignalType.ON_COMPLETE
                            && status != null
                            && status.is2xxSuccessful()) {
                        log.info("UserLoginGlobalFilter.post 放行后代码，业务请求成功");
                    } else {
                        log.info(
                            "UserLoginGlobalFilter.post，业务请求失败，signalType={}, status={}",
                            signalType,
                            status
                        );
                    }
                });
    }
}
```

代码要点：

- `chain.filter(exchange)`：将请求交给过滤器链继续处理。
- `doFinally(...)`：请求完成、异常或被取消后执行收尾逻辑。
- `exchange.getResponse().getStatusCode()`：获取 HTTP 响应状态码。
- `status.is2xxSuccessful()`：判断响应是否为成功的 `2xx` 状态。

> 📝 该配图代码主要演示全局过滤器的前置、放行和后置处理流程，尚未包含 JWT 提取与校验逻辑。

### 9.2 设置过滤器顺序

过滤器可以实现 `Ordered` 接口，并通过 `getOrder()` 设置执行顺序：

```java
@Override
public int getOrder() {
    return 0;
}
```

排序规则：**数值越小，执行顺序越靠前。**

### 9.3 身份校验的典型流程

```text
请求到达网关
    ↓
判断是否为无需登录的放行路径
    ↓ 否
从请求头中获取 JWT
    ↓
校验令牌是否存在、有效、未过期
    ├── 校验失败 → 返回未授权响应
    └── 校验成功 → 放行到目标微服务
```

---

## 十、知识点速查

| 知识点 | 核心内容 |
| --- | --- |
| `Logger.Level.FULL` | 输出完整的 Feign 请求和响应报文 |
| `logging.level` | 为 Feign 接口包开启 `DEBUG` 日志 |
| Gateway | 微服务集群的统一入口 |
| 路由 | 将符合条件的请求转发到指定服务 |
| `lb://服务名` | 通过负载均衡访问注册中心中的服务 |
| Predicate | 判断请求是否匹配某条路由 |
| 多个 Predicate | 条件之间为 AND 关系 |
| GatewayFilter | 在请求前后处理请求或响应 |
| JWT | 由 Header、Payload、Signature 三部分组成 |
| `GlobalFilter` | 对所有匹配请求执行全局过滤逻辑 |
| `Ordered` | 控制过滤器执行顺序，数值越小越靠前 |
| `chain.filter(exchange)` | 将请求放行到后续过滤器或目标服务 |

---

> ✅ **本节重点：** 能够开启 OpenFeign 完整日志；理解 Gateway 的统一入口、路由转发和身份校验职责；掌握 `id`、`uri`、`predicates` 等路由配置；理解登录接口生成 JWT 的过程，并能通过全局过滤器统一处理请求的前置与后置逻辑。
