# Spring MVC 请求响应与 SSM 整合总结（Day 53）

> 本节核心内容：**Spring MVC 参数绑定、日期时间转换、响应方式、RESTful API 设计，以及 Spring + Spring MVC + MyBatis 整合**。

## 目录

- [一、Param 参数接收](#一param-参数接收)
- [二、JSON 参数接收](#二json-参数接收)
- [三、路径参数](#三路径参数)
- [四、日期时间参数](#四日期时间参数)
- [五、Spring MVC 响应](#五spring-mvc-响应)
- [六、RESTful 风格](#六restful-风格)
- [七、SSM 整体架构](#七ssm-整体架构)
- [八、SSM 配置类](#八ssm-配置类)
- [九、Web 应用初始化与编码过滤](#九web-应用初始化与编码过滤)
- [十、单元测试环境问题](#十单元测试环境问题)
- [十一、重点速记](#十一重点速记)

---

## 一、Param 参数接收

Param 参数通常采用以下格式：

```text
key=value&key=value
```

它可以出现在 URL 查询字符串或 `application/x-www-form-urlencoded` 请求体中。

### 1. 零散形参接收

请求参数名与 Controller 形参名一致时，可以直接接收：

```java
@GetMapping("/users/search")
public String search(String name, Integer age) {
    return name + ":" + age;
}
```

请求示例：

```text
GET /users/search?name=Tom&age=18
```

为了在运行时保留 Java 方法参数名，可配置 Maven 编译插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>17</source>
                <target>17</target>
                <parameters>true</parameters>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 2. 参数名不一致

使用 `@RequestParam` 指定请求参数名：

```java
@GetMapping("/users/search")
public String search(@RequestParam("username") String name) {
    return name;
}
```

```text
请求参数 username → 方法形参 name
```

### 3. required 与 defaultValue

```java
@GetMapping("/users")
public String list(
        @RequestParam(required = false) String keyword,
        @RequestParam(defaultValue = "1") Integer page
) {
    return keyword + ":" + page;
}
```

| 属性 | 含义 |
|---|---|
| `value` / `name` | 请求参数名 |
| `required` | 是否必填，默认 `true` |
| `defaultValue` | 参数未提供或为空时使用的默认值，并隐含非必填语义 |

> 使用 `defaultValue` 时通常不必再显式写 `required = false`。

### 4. 同名参数的多个值

客户端可以重复传递同名参数：

```text
hobby=java&hobby=music
```

也可能按接口约定使用逗号分隔：

```text
hobby=java,music
```

#### 数组接收

```java
@GetMapping("/hobbies")
public String hobbies(String[] hobby) {
    return Arrays.toString(hobby);
}
```

#### 集合接收

集合参数建议明确添加 `@RequestParam`：

```java
@GetMapping("/hobbies")
public String hobbies(
        @RequestParam("hobby") List<String> hobbies
) {
    return hobbies.toString();
}
```

### 5. 对象形参接收

请求参数较多时，可以封装为对象。请求参数名应与对象属性名一致：

```java
public class UserQuery {
    private String name;
    private Integer age;
    // Getter、Setter
}
```

```java
@GetMapping("/users/search")
public UserQuery search(UserQuery query) {
    return query;
}
```

请求示例：

```text
/users/search?name=Tom&age=18
```

---

## 二、JSON 参数接收

### 1. 准备环境

Spring MVC 处理 JSON 通常需要：

- 引入与当前 Spring 版本兼容的 `jackson-databind`。
- 在 Spring MVC 配置类中启用 `@EnableWebMvc`。
- 客户端设置 `Content-Type: application/json`。

```java
@Configuration
@ComponentScan("com.edu.controller")
@EnableWebMvc
public class SpringMvcConfig {
}
```

### 2. 接收 JSON 对象

请求体：

```json
{
  "name": "张三",
  "age": 18
}
```

Controller：

```java
@PostMapping("/users")
public String save(@RequestBody User user) {
    return "success";
}
```

- 参数必须添加 `@RequestBody`。
- JSON 的键需要与 Java 对象属性正确对应。
- Jackson 将 JSON 反序列化为 Java 对象。

### 3. 接收 JSON 数组

请求体：

```json
[
  {"name": "张三", "age": 18},
  {"name": "李四", "age": 20}
]
```

Controller：

```java
@PostMapping("/users/batch")
public List<User> saveBatch(
        @RequestBody List<User> users
) {
    return users;
}
```

集合泛型类型要与 JSON 数组中的元素结构对应。

### 4. Param 与 JSON 对比

| 对比项 | Param 参数 | JSON 参数 |
|---|---|---|
| 格式 | `key=value&key=value` | 对象 `{}` 或数组 `[]` |
| 常见位置 | URL、表单请求体 | 请求体 |
| 常用注解 | `@RequestParam` | `@RequestBody` |
| 对象绑定 | 属性名对应参数名 | 属性名对应 JSON 键 |
| Content-Type | 表单或查询字符串 | `application/json` |

---

## 三、路径参数

在请求映射路径中使用 `{变量名}` 定义路径变量

==@pathVarible

```java
@GetMapping("/users/{id}")
public User selectById(@PathVariable Integer id) {
    return userService.selectById(id);
}
```

请求示例：

```text
GET /users/1
```

### 参数名不一致

```java
@GetMapping("/users/{id}")
public User selectById(
        @PathVariable("id") Integer userId
) {
    return userService.selectById(userId);
}
```

> 正确注解名是 `@PathVariable`，可通过 `value` 或 `name` 指定路径中的变量名。

---

## 四、日期时间参数

日期时间来自 Param 参数和 JSON 请求体时，转换机制不同。

### 1. Param 中的日期时间

使用 `@DateTimeFormat` 指定格式。

#### 添加在方法形参上

```java
@GetMapping("/events")
public String event(
        @RequestParam
        @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
        LocalDateTime time
) {
    return time.toString();
}
```

#### 添加在对象属性上

```java
public class EventQuery {

    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private LocalDate date;
}
```

### 2. JSON 中的日期时间

Java 8 日期时间类型通常需要 Jackson 的 JSR-310 模块：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>与 Jackson 主版本一致</version>
</dependency>
```

使用 `@JsonFormat`：

```java
public class Event {

    @JsonFormat(
        pattern = "yyyy-MM-dd HH:mm:ss",
        timezone = "GMT+8"
    )
    private LocalDateTime time;
}
```

| 数据来源 | 常用注解 | 转换组件 |
|---|---|---|
| Param / 表单 / 路径 | `@DateTimeFormat` | Spring 类型转换体系 |
| JSON 请求体或响应体 | `@JsonFormat` | Jackson |

---

## 五、Spring MVC 响应

### 1. 响应页面

方法不添加 `@ResponseBody`，返回值通常作为逻辑视图名称：

```java
@GetMapping("/home")
public String home(Model model) {
    model.addAttribute("message", "欢迎访问");
    return "home";
}
```

视图解析器的作用：

1. 根据逻辑视图名称找到物理视图。
2. 将数据模型与视图模板渲染成 HTML 页面。

例如逻辑视图 `home` 可以被解析成：

```text
/WEB-INF/views/home.jsp
```

### 2. 响应字符串

添加 `@ResponseBody` 后，返回值写入响应体：

```java
@GetMapping("/hello")
@ResponseBody
public String hello() {
    return "Hello Spring MVC";
}
```

### 3. 响应 JSON

返回 Java 对象或集合，消息转换器会使用 Jackson 序列化为 JSON：

```java
@GetMapping("/users/{id}")
@ResponseBody
public User selectById(@PathVariable Integer id) {
    return userService.selectById(id);
}
```

```java
@GetMapping("/users")
@ResponseBody
public List<User> selectAll() {
    return userService.selectAll();
}
```

| Controller 返回方式 | 处理结果 |
|---|---|
| 无 `@ResponseBody` + 返回字符串 | 字符串作为逻辑视图名 |
| `@ResponseBody` + 返回字符串 | 返回普通文本 |
| `@ResponseBody` + 返回对象/集合 | 转换成 JSON 响应 |

---

## 六、RESTful 风格

RESTful 是一种用于设计 Web API 的风格，不是强制协议。

### 1. 设计特点

- URL 表示资源，通常使用名词。
- 资源集合建议使用复数形式。
- HTTP 方法表示对资源执行的操作。
- 前后端交互通常使用 JSON。

| 操作 | HTTP 方法 | 示例 |
|---|---|---|
| 查询全部用户 | `GET` | `/users` |
| 查询单个用户 | `GET` | `/users/1` |
| 新增用户 | `POST` | `/users` |
| 修改用户 | `PUT` | `/users/1` |
| 删除用户 | `DELETE` | `/users/1` |

### 2. REST Controller

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public User selectById(@PathVariable Integer id) {
        return userService.selectById(id);
    }

    @PostMapping
    public User save(@RequestBody User user) {
        userService.save(user);
        return user;
    }

    @PutMapping("/{id}")
    public void update(
            @PathVariable Integer id,
            @RequestBody User user
    ) {
        user.setId(id);
        userService.update(user);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Integer id) {
        userService.deleteById(id);
    }
}
```

`@RestController` 等价于：

```text
@Controller + @ResponseBody
```

常用映射注解：

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`

---

## 七、SSM 整体架构

SSM 通常指 Spring、Spring MVC 和 MyBatis 的组合。

```text
客户端
   ↓ HTTP
Controller（Spring MVC）
   ↓
Service（Spring + 事务）
   ↓
Mapper（MyBatis）
   ↓
数据库
```

三层代码结构：

```text
com.edu
├─ controller        # 表现层
├─ service           # 业务层
├─ mapper            # 持久层接口
├─ entity            # 实体类
├─ config            # 配置类
└─ exception         # 异常相关（可选）

resources
├─ jdbc.properties
└─ com/edu/mapper    # Mapper XML（如使用 XML）
```

Mapper SQL 可以使用 XML 或注解配置。

---

## 八、SSM 配置类

建议在 `config` 包中按职责拆分配置类。

### 1. JdbcConfig

负责加载数据库配置、创建数据源和事务管理器：

```java
@Configuration
@PropertySource("classpath:jdbc.properties")
public class JdbcConfig {

    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        PooledDataSource source = new PooledDataSource();
        source.setDriver(driver);
        source.setUrl(url);
        source.setUsername(username);
        source.setPassword(password);
        return source;
    }

    @Bean
    public PlatformTransactionManager transactionManager(
            DataSource dataSource
    ) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

> `classpath:` 后不要误加空格，应写成 `classpath:jdbc.properties`。

### 2. MyBatisConfig

```java
@Configuration
public class MyBatisConfig {

    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(
            DataSource dataSource
    ) {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        factory.setTypeAliasesPackage("com.edu.entity");
        return factory;
    }

    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer scanner = new MapperScannerConfigurer();
        scanner.setBasePackage("com.edu.mapper");
        return scanner;
    }
}
```

### 3. SpringConfig

根容器只扫描业务层等非 Controller 组件：

```java
@Configuration
@ComponentScan("com.edu.service")
@EnableTransactionManagement
@Import({JdbcConfig.class, MyBatisConfig.class})
public class SpringConfig {
}
```

### 4. SpringMvcConfig

```java
@Configuration
@ComponentScan("com.edu.controller")
@EnableWebMvc
public class SpringMvcConfig {
}
```

### 5. 配置职责对比

| 配置类 | 主要职责 |
|---|---|
| `JdbcConfig` | 数据源、事务管理器、数据库属性 |
| `MyBatisConfig` | `SqlSessionFactory`、Mapper 扫描 |
| `SpringConfig` | Service 扫描、事务开启、导入持久层配置 |
| `SpringMvcConfig` | Controller 扫描、开启 MVC |
| `WebAppInitConfig` | 创建父子容器、注册 DispatcherServlet 与 Filter |

---

## 九、Web 应用初始化与编码过滤

### 1. WebAppInitConfig

```java
public class WebAppInitConfig
        extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext
    createServletApplicationContext() {
        AnnotationConfigWebApplicationContext context =
                new AnnotationConfigWebApplicationContext();
        context.register(SpringMvcConfig.class);
        return context;
    }

    @Override
    protected WebApplicationContext
    createRootApplicationContext() {
        AnnotationConfigWebApplicationContext context =
                new AnnotationConfigWebApplicationContext();
        context.register(SpringConfig.class);
        return context;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter filter =
                new CharacterEncodingFilter();
        filter.setEncoding("UTF-8");
        filter.setForceRequestEncoding(true);
        filter.setForceResponseEncoding(true);
        return new Filter[]{filter};
    }
}
```

| 方法 | 作用 |
|---|---|
| `createServletApplicationContext()` | 初始化 Spring MVC 子容器 |
| `createRootApplicationContext()` | 初始化 Spring 根容器 |
| `getServletMappings()` | 设置 `DispatcherServlet` 映射路径 |
| `getServletFilters()` | 注册 Servlet Filter |

### 2. CharacterEncodingFilter

它会在请求到达 Controller 前设置请求编码，并可设置响应编码：

```text
request.setCharacterEncoding("UTF-8")
response.setCharacterEncoding("UTF-8")
```

主要用于：

- 解决 POST 表单请求中文乱码。
- 统一响应字符编码。

---

## 十、单元测试环境问题

### 1. 问题原因

如果 Spring 根容器的组件扫描范围过大，把 `SpringMvcConfig` 也扫描进来，则 `@EnableWebMvc` 会在普通非 Web 测试上下文中加载依赖 Web 环境的组件，可能导致测试启动失败。

### 2. 推荐解决方案

明确隔离扫描范围：

```java
// 根容器只扫描业务层
@ComponentScan("com.edu.service")
```

```java
// MVC 子容器只扫描控制器
@ComponentScan("com.edu.controller")
```

这是结构更清晰的解决方式。

### 3. 测试确实需要 Web 环境

可添加 `@WebAppConfiguration`：

```java
@SpringJUnitConfig(SpringConfig.class)
@WebAppConfiguration
class UserControllerTest {
}
```

| 场景 | 处理方式 |
|---|---|
| Service / Mapper 单元或集成测试 | 不加载 Spring MVC 配置 |
| Controller Web 测试 | 加载 MVC 配置并使用 `@WebAppConfiguration` |

---

## 十一、重点速记

> [!TIP]
> **Param 用 `@RequestParam`，JSON 用 `@RequestBody`，路径变量用 `@PathVariable`；REST Controller 默认将返回值写入响应体。**

- 请求参数名与形参名一致时可直接绑定。
- 名称不一致时使用 `@RequestParam("请求参数名")`。
- `required` 默认是 `true`；`defaultValue` 可设置默认值。
- 同名多值可用数组或带 `@RequestParam` 的集合接收。
- 对象绑定要求请求参数名与对象属性名对应。
- JSON 对象和数组使用 `@RequestBody` 接收。
- `@PathVariable` 接收 URL 中的 `{变量}`。
- Param 日期用 `@DateTimeFormat`，JSON 日期用 `@JsonFormat`。
- 不加 `@ResponseBody` 时，字符串返回值通常表示逻辑视图名。
- 加 `@ResponseBody` 后，对象和集合通过 Jackson 转换为 JSON。
- `@RestController = @Controller + @ResponseBody`。
- RESTful URL 表示资源，HTTP 方法表示操作。
- `GET` 查询、`POST` 新增、`PUT` 修改、`DELETE` 删除。
- SSM 分层：Controller → Service → Mapper → 数据库。
- `JdbcConfig` 管理数据源和事务管理器。
- `MyBatisConfig` 管理 `SqlSessionFactory` 和 Mapper 扫描。
- `SpringConfig` 管理业务层，`SpringMvcConfig` 管理控制器。
- `CharacterEncodingFilter` 统一处理请求和响应编码。
- 普通 Service 测试不要意外加载 `@EnableWebMvc` 配置。

---

> 本文根据思维导图 **《day53》** 整理，对 Spring MVC 参数绑定、响应、RESTful API 和 SSM 整合流程进行了结构化归纳与方法名称规范化。
