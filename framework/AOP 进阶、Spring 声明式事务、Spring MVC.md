# Spring AOP、事务与 Spring MVC 学习总结（Day 52）

> 本节核心内容：**AOP 通知中获取参数、返回值与异常，Spring 声明式事务，以及 Spring MVC 的环境搭建、容器关系和请求参数接收**。

## 目录

- [第一部分：AOP 进阶](#第一部分aop-进阶)
- [一、通知中获取方法参数](#一通知中获取方法参数)
- [二、获取返回值与异常](#二获取返回值与异常)
- [三、Spring AOP 代理机制](#三spring-aop-代理机制)
- [第二部分：Spring 声明式事务](#第二部分spring-声明式事务)
- [四、声明式事务配置](#四声明式事务配置)
- [五、事务回滚与传播行为](#五事务回滚与传播行为)
- [第三部分：Spring MVC](#第三部分spring-mvc)
- [六、Spring MVC 概述](#六spring-mvc-概述)
- [七、Spring MVC 入门程序](#七spring-mvc-入门程序)
- [八、Spring 与 Spring MVC 父子容器](#八spring-与-spring-mvc-父子容器)
- [九、请求映射](#九请求映射)
- [十、请求参数分类与接收](#十请求参数分类与接收)
- [十一、中文乱码处理](#十一中文乱码处理)
- [十二、重点速记](#十二重点速记)

---

# 第一部分：AOP 进阶

## 一、通知中获取方法参数

Spring AOP 的五种通知都可以获取目标方法的实参。

### 1. 非环绕通知

前置、后置、返回和异常通知使用 `JoinPoint`：

```java
@Before("pt()")
public void before(JoinPoint joinPoint) {
    Object[] args = joinPoint.getArgs();
    System.out.println(Arrays.toString(args));
}
```

`getArgs()` 返回 `Object[]`，数组内容就是本次目标方法调用时传入的参数。

还可以获取方法签名和目标对象：

```java
String methodName = joinPoint.getSignature().getName();
Object target = joinPoint.getTarget();
```

### 2. 环绕通知

环绕通知使用 `ProceedingJoinPoint`，它是 `JoinPoint` 的子接口：

```java
@Around("pt()")
public Object around(ProceedingJoinPoint pjp) throws Throwable {
    Object[] args = pjp.getArgs();
    System.out.println("参数：" + Arrays.toString(args));
    return pjp.proceed();
}
```

`proceed()` 用于执行目标方法。也可以传入新的参数数组：

```java
return pjp.proceed(newArgs);
```

| 通知类型 | 连接点参数 |
|---|---|
| `@Before`、`@After`、`@AfterReturning`、`@AfterThrowing` | `JoinPoint` |
| `@Around` | `ProceedingJoinPoint` |

---

## 二、获取返回值与异常

### 1. 获取目标方法返回值

只有环绕通知和返回通知能获得正常返回值。

#### 环绕通知

`proceed()` 的返回结果就是目标方法返回值：

```java
@Around("pt()")
public Object around(ProceedingJoinPoint pjp) throws Throwable {
    Object result = pjp.proceed();
    System.out.println("返回值：" + result);
    return result;
}
```

> 环绕通知应返回目标结果，否则调用方可能得到 `null` 或错误结果。

#### 返回通知

`returning` ==属性值==必须与通知方法中的==形参名==一致：

```java
@AfterReturning(value = "pt()",
        returning = "result"
)
public void afterReturning(JoinPoint joinPoint, Object result) {
    System.out.println("返回值：" + result);
}
```

### 2. 获取目标方法异常

只有环绕通知和异常通知能获得目标方法抛出的异常。

#### 环绕通知

```java
@Around("servicePointcut()")
public Object around(ProceedingJoinPoint pjp) throws Throwable {
    try {
        return pjp.proceed();
    } catch (Throwable throwable) {
        System.out.println("异常：" + throwable.getMessage());
    }
}
```

捕获异常后通常要继续抛出，避免悄悄吞掉异常而改变业务语义。

#### 异常通知

`throwing` 属性值必须与通知方法中的形参名一致：

```java
@AfterThrowing(value = "pt()",
        throwing = "exception"
)
public void afterThrowing(
        JoinPoint joinPoint,
        Throwable exception
) {
    System.out.println("异常：" + exception.getMessage());
}
```

### 3. 能力对比

| 通知 | 获取实参 | 获取返回值 | 获取异常 | 控制目标方法执行 |
|---|:---:|:---:|:---:|:---:|
| `@Before` | ✅ | ❌ | ❌ | ❌ |
| `@After` | ✅ | ❌ | ❌ | ❌ |
| `@AfterReturning` | ✅ | ✅ | ❌ | ❌ |
| `@AfterThrowing` | ✅ | ❌ | ✅ | ❌ |
| `@Around` | ✅ | ✅ | ✅ | ✅ |

---

## 三、Spring AOP 代理机制

Spring AOP 底层主要使用 JDK 动态代理和 CGLIB 代理。

### 1. JDK 动态代理

- 基于接口创建代理。
- 代理对象与原始对象实现相同接口。
- 原始对象和代理对象可理解为同一接口下的“兄弟关系”。

```text
        UserService 接口
          /        \
原始实现对象      JDK 代理对象
```

### 2. CGLIB 代理

- 通过生成目标类的子类创建代理。
- 原始类与代理类是父子关系。
- `final` 类或 `final` 方法不能被这种继承式代理有效增强。

```text
原始目标类（父类）
       ↑
CGLIB 代理类（子类）
```

### 3. 默认选择

一般情况下：

```text
目标类实现接口 → 优先使用 JDK 动态代理
目标类没有接口 → 使用 CGLIB 代理
```

可通过以下配置倾向于使用类代理：

```java
@EnableAspectJAutoProxy(proxyTargetClass = true)
```

| 对比项 | JDK 动态代理 | CGLIB |
|---|---|---|
| 基础 | 接口 | 继承目标类 |
| 代理关系 | 实现相同接口 | 代理类继承目标类 |
| 无接口类 | 不适用 | 适用 |
| 主要限制 | 通过接口暴露方法 | `final` 类/方法受限 |

---

# 第二部分：Spring 声明式事务

## 四、声明式事务配置

### 1. 引入依赖

主要使用 `spring-jdbc` 和 `spring-tx`。通常引入 `spring-jdbc` 时会传递引入 `spring-tx`，但版本应与其他 Spring 模块保持一致。

### 2. 注册事务管理器

```java
@Configuration
public class TransactionConfig {

    @Bean
    public PlatformTransactionManager transactionManager(
            DataSource dataSource
    ) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

`PlatformTransactionManager` 是 Spring 事务管理器的统一接口；使用 JDBC/MyBatis 时常用 `DataSourceTransactionManager`。

### 3. 开启声明式事务

```java
@Configuration
@EnableTransactionManagement
public class SpringConfig {
}
```

### 4. 添加 `@Transactional`

事务通常加在业务层方法上：

```java
@Service
public class AccountServiceImpl implements AccountService {

    @Transactional
    @Override
    public void transfer(
            Integer sourceId,
            Integer targetId,
            BigDecimal amount
    ) {
        accountMapper.decrease(sourceId, amount);
        accountMapper.increase(targetId, amount);
    }
}
```

执行过程：

```text
进入事务代理 → 开启事务 → 执行业务方法
                         ├─ 正常：提交事务
                         └─ 异常：按回滚规则处理
```

### 5. 完整步骤

1. 引入 `spring-jdbc` 等依赖。
2. 将 `PlatformTransactionManager` 注册到 IoC 容器。
3. 在业务方法或业务类上添加 `@Transactional`。
4. 使用 `@EnableTransactionManagement` 开启声明式事务。

---

## 五、事务回滚与传播行为

### 1. rollbackFor

默认情况下，Spring 通常对 `RuntimeException` 和 `Error` 回滚，对普通受检异常默认不回滚。

若希望所有 `Exception` 都触发回滚：

```java
@Transactional(rollbackFor = Exception.class)
public void save() throws Exception {
    // 业务逻辑
}
```

### 2. propagation

事务传播行为决定一个带事务的方法调用另一个事务方法时，两个方法如何使用事务。

#### REQUIRED（默认）

```java
@Transactional(propagation = Propagation.REQUIRED)
```

- 外层已有事务：加入当前事务。
- 外层没有事务：创建新事务。
- 两个方法通常共同提交或共同回滚。

#### REQUIRES_NEW

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
```

- 无论外层是否存在事务，都创建新事务。
- 若外层已有事务，会先挂起外层事务。
- 新事务可以独立提交或回滚。

| 传播行为 | 外层有事务 | 外层无事务 |
|---|---|---|
| `REQUIRED` | 加入外层事务 | 创建新事务 |
| `REQUIRES_NEW` | 挂起外层并创建新事务 | 创建新事务 |

> `REQUIRES_NEW` 常用于必须独立保存的操作，如审计记录，但仍需结合异常是否传播来判断外层事务最终结果。

### 3. 事务常见注意事项

- `@Transactional` 依赖 Spring 代理，通常应通过代理对象调用。
- 同一类内部直接调用另一个事务方法，可能绕过代理导致事务配置不生效。
- 事务方法一般应是可被代理的公共业务方法。
- 不要捕获异常后完全吞掉，否则事务拦截器可能认为方法正常结束并提交。

---

# 第三部分：Spring MVC

## 六、Spring MVC 概述

Spring MVC 是基于 Servlet 实现的表现层框架，用于简化 Java Web 开发。

主要能力：

- 简化请求路径映射。
- 简化请求参数接收和类型转换。
- 简化响应数据输出。
- 统一异常、拦截器、文件上传等 Web 功能。

核心入口是 `DispatcherServlet`：

```text
浏览器请求
    ↓
DispatcherServlet
    ↓
匹配 Controller 方法
    ↓
调用业务层
    ↓
生成响应
```

---

## 七、Spring MVC 入门程序

### 1. 创建 Maven Web 项目

项目打包方式通常为：

```xml
<packaging>war</packaging>
```

集成本地 Tomcat 时，可按需要将应用上下文路径设置为 `/`。

### 2. 引入依赖

主要依赖：

- `spring-webmvc`
- `jakarta.servlet-api`

Servlet API 由 Tomcat 提供，应使用 `provided`：

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>项目指定版本</version>
    <scope>provided</scope>
</dependency>
```

### 3. 定义 Controller

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/hello")
    @ResponseBody
    public String hello() {
        return "Hello Spring MVC";
    }
}
```

- `@Controller`：将控制器注册为 Bean。
- `@RequestMapping`：建立请求路径与方法的映射。
- `@ResponseBody`：将方法返回值写入 HTTP 响应体，由消息转换器转换成相应格式。

### 4. Spring MVC 配置类

```java
@Configuration
@ComponentScan("com.edu.controller")
@EnableWebMvc
public class SpringMvcConfig {
}
```

### 5. Web 应用初始化类

使用 Java 配置替代 `web.xml`：

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
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    @Override
    protected WebApplicationContext
    createRootApplicationContext() {
        AnnotationConfigWebApplicationContext context =
                new AnnotationConfigWebApplicationContext();
        context.register(SpringConfig.class);
        return context;
    }
}
```

| 方法 | 作用 |
|---|---|
| `createServletApplicationContext()` | 初始化 Spring MVC 子容器 |
| `getServletMappings()` | 配置 `DispatcherServlet` 映射路径；`/` 表示处理默认 Web 请求 |
| `createRootApplicationContext()` | 初始化 Spring 根容器 |

---

## 八、Spring 与 Spring MVC 父子容器

### 1. Bean 管理范围

Spring MVC 子容器只扫描控制器：

```java
@ComponentScan("com.edu.controller")
```

Spring 根容器管理 Service、DAO 等非控制器 Bean：

```java
@ComponentScan(
    value = "com.edu",
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = Controller.class
    )
)
```

### 2. 父子关系

```text
Spring 根容器（父）
├─ Service
├─ Mapper / DAO
├─ DataSource
└─ TransactionManager
        ↑ 子容器可访问
Spring MVC 容器（子）
└─ Controller
```

- 父容器通常先初始化。
- 子容器可以访问父容器中的 Bean，因此 Controller 可以注入 Service。
- 父容器不能反向访问子容器独有的 Bean。

这样可以避免 Controller 被两个容器重复扫描和注册。

---

## 九、请求映射

`@RequestMapping` 可以加在类和方法上。

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/list")
    @ResponseBody
    public String list() {
        return "list";
    }
}
```

最终路径：

```text
/user/list
```

- 类上的路径是该类所有方法的统一前缀。
- 方法上的路径定位具体处理方法。
- 路径开头的 `/` 通常可以省略，但建议团队统一风格。

实际开发还常使用限定 HTTP 方法的组合注解：

```java
@GetMapping("/{id}")
@PostMapping
@PutMapping("/{id}")
@DeleteMapping("/{id}")
```

---

## 十、请求参数分类与接收

### 1. Param 参数

格式：

```text
key=value&key=value
```

常见传递位置：

- GET：URL 查询字符串，如 `/user?name=Tom&age=18`。
- POST：请求体使用 `application/x-www-form-urlencoded`。
- PUT、DELETE 也可通过查询字符串传递。

Apifox 中：

- GET 使用 `Params`。
- POST 表单使用 `Body → x-www-form-urlencoded`。

#### 零散形参接收

请求参数名和方法形参名一致时：

```java
@GetMapping("/search")
@ResponseBody
public String search(String name, Integer age) {
    return name + ":" + age;
}
```

为了让运行时保留 Java 方法参数名，可在 Maven 编译插件中设置：

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

更明确的写法是使用 `@RequestParam`：

```java
@GetMapping("/search")
@ResponseBody
public String search(
        @RequestParam("name") String username,
        @RequestParam(defaultValue = "1") Integer page
) {
    return username + ":" + page;
}
```

### 2. JSON 参数

格式示例：

```json
{
  "name": "张三",
  "age": 18
}
```

JSON 通常放在 POST 或 PUT 请求体中，并设置：

```http
Content-Type: application/json
```

接收时使用 `@RequestBody`：

```java
@PostMapping("/user")
@ResponseBody
public User save(@RequestBody User user) {
    return user;
}
```

Apifox 中选择 `Body → JSON`。

> HTTP 规范并非绝对禁止其他方法携带请求体，但接口设计和工具使用通常按上述约定进行。

### 3. 路径参数

URL 示例：

```text
http://localhost/user/1
```

使用 `{变量名}` 定义，并通过 `@PathVariable` 接收：

```java
@GetMapping("/user/{id}")
@ResponseBody
public User selectById(@PathVariable Integer id) {
    return userService.selectById(id);
}
```

路径变量可配合 GET、POST、PUT、DELETE 等请求方式使用。Apifox 中直接在 URL 中书写实际值。

### 4. 三类参数对比

| 参数类型 | 示例 | 常用接收注解 | Apifox 位置 |
|---|---|---|---|
| Param | `?name=Tom&age=18` | `@RequestParam` 或同名形参 | Params / x-www-form-urlencoded |
| JSON | `{"name":"Tom"}` | `@RequestBody` | Body → JSON |
| 路径参数 | `/user/1` | `@PathVariable` | URL |

---

## 十一、中文乱码处理

### 1. GET 参数

Tomcat 10 通常已经使用 UTF-8 正确处理 URL 查询参数，一般无需额外代码。

### 2. POST 表单参数

可注册 Spring 提供的 `CharacterEncodingFilter`：

```java
@Override
protected Filter[] getServletFilters() {
    CharacterEncodingFilter filter =
            new CharacterEncodingFilter();
    filter.setEncoding("UTF-8");
    filter.setForceEncoding(true);
    return new Filter[]{filter};
}
```

将该方法添加到 `WebAppInitConfig` 中，使请求进入 `DispatcherServlet` 前统一设置编码。

### 3. JSON 编码

客户端应正确设置：

```http
Content-Type: application/json;charset=UTF-8
```

Spring MVC 的消息转换器会按媒体类型和编码读取请求体。

---

## 十二、重点速记

> [!TIP]
> **AOP 负责横切增强，事务保证一组数据库操作的一致性，Spring MVC 负责接收请求和返回响应。**

- 五种 AOP 通知都可通过连接点对象获取方法实参。
- 环绕通知使用 `ProceedingJoinPoint`，其他通知使用 `JoinPoint`。
- 环绕通知通过 `proceed()` 获取返回值并执行目标方法。
- 返回通知使用 `returning` 绑定返回值参数。
- 异常通知使用 `throwing` 绑定异常参数。
- 有接口通常使用 JDK 动态代理，无接口通常使用 CGLIB。
- `PlatformTransactionManager` 是 Spring 事务管理器接口。
- `@EnableTransactionManagement` 开启声明式事务。
- `@Transactional` 通常加在业务层公共方法上。
- 默认主要对运行时异常和错误回滚；`rollbackFor` 可扩展规则。
- `REQUIRED`：有事务就加入，无事务就创建。
- `REQUIRES_NEW`：始终创建新事务，必要时挂起外层事务。
- Spring MVC 基于 Servlet，核心入口是 `DispatcherServlet`。
- `@Controller` 注册控制器，`@RequestMapping` 映射路径。
- `@ResponseBody` 将返回值通过消息转换器写入响应体。
- Spring MVC 子容器管理 Controller；Spring 父容器管理 Service、DAO 等。
- 子容器可以访问父容器，父容器不能访问子容器独有 Bean。
- Param 参数使用 `@RequestParam`，JSON 使用 `@RequestBody`，路径参数使用 `@PathVariable`。
- POST 表单中文可通过 `CharacterEncodingFilter` 统一处理。

---

> 本文根据思维导图 **《day52》** 整理，对 AOP 进阶、声明式事务和 Spring MVC 入门知识进行了结构化归纳与规范化补充。
