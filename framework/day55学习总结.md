# Spring Boot 配置、整合与自动配置原理（Day 55）

> 核心内容：**YAML 集合配置、Profiles 多环境、MyBatis/Druid/JUnit 整合、日志、Bean 注册方式，以及 Spring Boot 自动配置原理**。

## 目录

- [一、YAML 多值配置](#一yaml-多值配置)
- [二、Profiles 多环境配置](#二profiles-多环境配置)
- [三、配置覆盖优先级](#三配置覆盖优先级)
- [四、Spring Boot 整合 MyBatis](#四spring-boot-整合-mybatis)
- [五、Spring Boot 整合 Druid](#五spring-boot-整合-druid)
- [六、Spring Boot 整合 JUnit 5](#六spring-boot-整合-junit-5)
- [七、Spring Boot 日志](#七spring-boot-日志)
- [八、自定义与第三方 Bean 注册](#八自定义与第三方-bean-注册)
- [九、ImportSelector 与 Enable 模式](#九importselector-与-enable-模式)
- [十、Spring Boot 自动配置原理](#十spring-boot-自动配置原理)
- [十一、条件装配注解](#十一条件装配注解)
- [十二、重点速记](#十二重点速记)

---

## 一、YAML 多值配置

YAML 使用 `-` 定义列表，可绑定到数组、`List` 或 `Set` 等集合类型。

### 1. YAML 中定义列表

```yaml
app:
  servers:
    - server-a
    - server-b
    - server-c
  ports:
    - 8080
    - 8081
```

对象列表：

```yaml
app:
  users:
    - name: zhangsan
      age: 18
    - name: lisi
      age: 20
```

### 2. 使用配置类读取

```java
@Data
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private List<String> servers;
    private int[] ports;
    private List<UserProperties> users;
}
```

```java
@Data
public class UserProperties {
    private String name;
    private Integer age;
}
```

> 多个相关配置优先使用 `@ConfigurationProperties` 批量绑定，比在各字段上重复使用 `@Value` 更清晰。

---

## 二、Profiles 多环境配置

不同运行环境通常拥有不同的数据库地址、端口和日志级别。Spring Boot Profiles 可拆分共性配置和环境专属配置。

### 1. 配置文件划分

```text
application.yml        # 公共配置和环境激活
application-dev.yml    # 开发环境
application-test.yml   # 测试环境
application-prod.yml   # 生产环境
```

公共配置：

```yaml
spring:
  profiles:
    active: dev

spring:
  application:
    name: user-service
```

> 实际 YAML 中不能重复定义同一顶层键，否则后一个键可能覆盖前一个。应合并书写：

```yaml
spring:
  application:
    name: user-service
  profiles:
    active: dev
```

开发环境 `application-dev.yml`：

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/demo_dev
```

生产环境 `application-prod.yml`：

```yaml
server:
  port: 80

spring:
  datasource:
    url: jdbc:mysql://prod-db:3306/demo
```

### 2. 激活环境

配置文件激活：

```yaml
spring:
  profiles:
    active: prod
```

命令行激活：

```bash
java -jar app.jar --spring.profiles.active=prod
```

环境变量也可用于部署平台：

```text
SPRING_PROFILES_ACTIVE=prod
```

> 生产环境通常由部署命令或环境变量激活，不建议把 `prod` 固定写入项目默认配置。

---

## 三、配置覆盖优先级

同一个属性来自多个位置时，高优先级来源会覆盖低优先级来源。课程中的常见简化顺序如下：

```text
命令行参数
    ↓
JVM 系统属性
    ↓
JAR 外 application-{profile}.yml
    ↓
JAR 外 application.yml
    ↓
JAR 内 application-{profile}.yml
    ↓
JAR 内 application.yml
```

示例：

```bash
java -Dserver.port=8888 -jar app.jar \
  --spring.profiles.active=prod \
  --server.port=9999
```

此时命令行中的 `--server.port=9999` 优先级高于 JVM 系统属性中的 `-Dserver.port=8888`。

> Spring Boot 的完整外部配置顺序还包含环境变量、Java 系统属性、JSON、测试属性等。实际排查覆盖问题时，可结合启动日志和 Actuator 的环境端点确认最终值。

---

## 四、Spring Boot 整合 MyBatis

### 1. 引入依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>与 Spring Boot 版本兼容的版本</version>
</dependency>

<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 2. 配置数据源和 MyBatis

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springboot
    username: root
    password: 1234

mybatis:
  type-aliases-package: com.edu.entity
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
```

### 3. 扫描 Mapper

```java
@SpringBootApplication
@MapperScan("com.edu.mapper")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

也可以在每个接口上添加 `@Mapper`，但集中使用 `@MapperScan` 更方便。

### 4. MyBatis 日志

未显式配置 `log-impl` 时，MyBatis 会按可用日志实现进行探测；Spring Boot 项目通常通过 SLF4J 进入统一日志体系。

```yaml
logging:
  level:
    com.edu.mapper: debug
```

若配置：

```yaml
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

MyBatis SQL 日志会直接输出到标准输出，不再按 SLF4J 的级别、格式和输出目标统一管理。因此正式项目通常优先使用 SLF4J 日志体系。

---

## 五、Spring Boot 整合 Druid

### 1. 默认数据源

Spring Boot JDBC Starter 默认常使用 HikariCP，对应实现为 `HikariDataSource`。

### 2. 引入 Druid Starter

Spring Boot 3 项目可使用对应的 Druid Starter。导图中的版本示例：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-3-starter</artifactId>
    <version>1.2.21</version>
</dependency>
```

> 具体版本需要与所用 Spring Boot、JDK 版本兼容。

### 3. 配置 Druid

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springboot
    username: root
    password: 1234
    druid:
      initial-size: 10
      max-active: 20
      min-idle: 5
      max-wait: 60000
```

公共数据源属性写在 `spring.datasource` 下，Druid 特有属性写在 `spring.datasource.druid` 下。

---

## 六、Spring Boot 整合 JUnit 5

### 1. 测试依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

该 Starter 默认集成 JUnit Jupiter 等常用测试工具。

### 2. 测试类

```java
@SpringBootTest
class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    void selectAll() {
        assertNotNull(userService.selectAll());
    }
}
```

当测试类位于启动类所在包或其子包时，通常能自动找到 `@SpringBootConfiguration`，无需指定 `classes`。

若不在扫描结构中：

```java
@SpringBootTest(classes = Application.class)
```

> 正确注解名是 `@SpringBootTest`。

---

## 七、Spring Boot 日志

Spring Boot 默认使用 SLF4J 门面，常见默认实现是 Logback。

### 1. 使用日志

```java
@Slf4j
@Service
public class UserService {

    public void save(User user) {
        log.info("保存用户，id={}", user.getId());
    }
}
```

使用 `{}` 参数化日志可避免不必要的字符串拼接。

### 2. application.yml 配置

```yaml
logging:
  level:
    root: info
    com.edu: debug
  file:
    name: logs/application.log
```

### 3. 自定义 Logback

简单规则优先使用 `application.yml`。需要复杂滚动、格式或多输出目标时，可在 `resources` 中提供：

```text
logback-spring.xml（Spring Boot 推荐）
logback.xml
```

`logback-spring.xml` 支持部分 Spring Boot 扩展能力。

---

## 八、自定义与第三方 Bean 注册

### 1. 自定义 Bean

```java
@Controller
@Service
@Repository
@Component
```

`@SpringBootApplication` 默认扫描启动类所在包及其子包，所以启动类应放在项目根包。

### 2. 第三方类没有配置类

自己编写配置类并使用 `@Bean`：

```java
@Configuration
public class ThirdPartyConfig {

    @Bean
    public SomeClient someClient() {
        return new SomeClient();
    }
}
```

### 3. 第三方 JAR 已提供配置类

推荐使用 `@Import` 精确导入：

```java
@SpringBootApplication
@Import(ThirdPartyConfig.class)
public class Application {
}
```

也可以扩展 `@ComponentScan`，但若重新声明扫描包，必须保留应用自身根包：

```java
@SpringBootApplication
@ComponentScan({"com.edu", "com.thirdparty"})
```

否则可能让应用自己的默认扫描范围丢失或发生变化。对于第三方配置，`@Import` 通常更明确。

---

## 九、ImportSelector 与 Enable 模式

### 1. ImportSelector

第三方库可以提供 `ImportSelector`，动态返回需要导入的配置类全限定名：

```java
public class MyImportSelector implements ImportSelector {

    @Override
    public String[] selectImports(
            AnnotationMetadata importingClassMetadata
    ) {
        return new String[]{
                "com.example.config.ClientConfig",
                "com.example.config.CacheConfig"
        };
    }
}
```

使用：

```java
@Import(MyImportSelector.class)
```

### 2. 自定义 `@EnableXxx`

第三方库通常进一步封装为启用注解：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(MyImportSelector.class)
public @interface EnableMyFeature {
}
```

应用只需：

```java
@SpringBootApplication
@EnableMyFeature
public class Application {
}
```

```text
@EnableMyFeature
      ↓ @Import
ImportSelector
      ↓ selectImports()
配置类
      ↓
Bean 注册到容器
```

---

## 十、Spring Boot 自动配置原理

### 1. 注解链路

```text
@SpringBootApplication
├─ @SpringBootConfiguration
├─ @ComponentScan
└─ @EnableAutoConfiguration
       ↓
   @Import(AutoConfigurationImportSelector.class)
       ↓
   发现候选自动配置类
       ↓
   条件匹配
       ↓
   注册满足条件的 Bean
```

### 2. 候选配置来源

Spring Boot 2 时代常通过以下文件声明自动配置候选项：

```text
META-INF/spring.factories
```

Spring Boot 3 的自动配置导入主要使用：

```text
META-INF/spring/
org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

`AutoConfigurationImportSelector` 负责读取和筛选候选自动配置。并非读取到的配置都会生效，还要通过条件注解判断。

### 3. 核心理解

```text
Starter：把需要的依赖放到类路径
自动配置：发现候选配置并按条件注册 Bean
用户配置：可通过自定义 Bean 或属性覆盖默认行为
```

---

## 十一、条件装配注解

### 1. `@ConditionalOnClass`

```java
@ConditionalOnClass(SomeClient.class)
```

只有类路径存在指定类时配置才生效，通常意味着应用引入了相关依赖。

### 2. `@ConditionalOnProperty`

```java
@ConditionalOnProperty(
        name = "feature.age",
        havingValue = "18"
)
```

只有配置属性存在且值符合要求时才生效。可使用 `matchIfMissing` 控制属性缺失时是否匹配。

### 3. `@ConditionalOnMissingBean`

```java
@ConditionalOnMissingBean(SomeClient.class)
```

只有容器中不存在指定 Bean 时才注册默认 Bean。这体现了 Spring Boot 常见的“用户配置优先”退让机制。

### 4. 自动配置示例

```java
@AutoConfiguration
@ConditionalOnClass(SomeClient.class)
@ConditionalOnProperty(
        prefix = "some.client",
        name = "enabled",
        havingValue = "true",
        matchIfMissing = true
)
public class SomeClientAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public SomeClient someClient() {
        return new SomeClient();
    }
}
```

| 条件注解 | 判断依据 |
|---|---|
| `@ConditionalOnClass` | 类路径是否存在指定类 |
| `@ConditionalOnProperty` | 配置属性是否满足要求 |
| `@ConditionalOnMissingBean` | 容器中是否缺少指定 Bean |

---

## 十二、重点速记

> [!TIP]
> **Profile 负责环境切换，Starter 提供依赖，自动配置按条件注册 Bean，用户自定义 Bean 可替代默认配置。**

- YAML 列表使用 `-` 定义，可绑定数组或集合。
- `application.yml` 放共性配置，`application-{profile}.yml` 放环境专属配置。
- 使用 `spring.profiles.active` 激活环境。
- 命令行参数通常可覆盖 JVM 属性和配置文件。
- MyBatis Starter 简化 `SqlSessionFactory` 和 Mapper 代理配置。
- `@MapperScan` 指定 Mapper 接口包。
- MyBatis 使用 `StdOutImpl` 后，SQL 日志不再走统一 SLF4J 管理。
- Spring Boot 默认常使用 HikariCP，也可通过 Starter 切换到 Druid。
- `spring-boot-starter-test` 默认支持 JUnit 5。
- `@SpringBootTest` 加载 Spring Boot 测试上下文。
- Spring Boot 默认使用 SLF4J + Logback。
- 启动类应放在根包，以覆盖业务子包扫描。
- 第三方类没有配置时，使用 `@Bean` 注册。
- 已有第三方配置类时，优先使用 `@Import` 精确导入。
- `ImportSelector.selectImports()` 返回待导入配置类的全限定名数组。
- `@EnableXxx` 通常封装了 `@Import`。
- `@SpringBootApplication` 包含 `@EnableAutoConfiguration`。
- Spring Boot 3 主要从 `AutoConfiguration.imports` 发现自动配置候选项。
- `@ConditionalOnClass` 判断依赖，`@ConditionalOnProperty` 判断属性，`@ConditionalOnMissingBean` 允许用户配置优先。

---

> 本文根据思维导图 **《day55》** 整理，对 Spring Boot 多环境、框架整合、Bean 导入机制与自动配置原理进行了结构化归纳和名称规范化。
