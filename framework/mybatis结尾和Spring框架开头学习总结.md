# MyBatis 收尾与 Spring 入门总结（Day 50）

> 本节分为两部分：MyBatis 的**对一关联查询与 PageHelper 分页**，以及 Spring Framework 的 **IoC、Bean 注册、容器与依赖注入**。

## 目录

- [第一部分：MyBatis 收尾](#第一部分mybatis-收尾)
- [一、对一关联查询](#一对一关联查询)
- [二、PageHelper 分页插件](#二pagehelper-分页插件)
- [第二部分：Spring 入门](#第二部分spring-入门)
- [三、Spring 概述](#三spring-概述)
- [四、IoC、Bean 与 DI](#四iocbean-与-di)
- [五、Bean 注册](#五bean-注册)
- [六、IoC 容器](#六ioc-容器)
- [七、获取 Bean](#七获取-bean)
- [八、依赖注入](#八依赖注入)
- [九、外部属性注入](#九外部属性注入)
- [十、重点速记](#十重点速记)

---

# 第一部分：MyBatis 收尾

## 一、对一关联查询

### 1. 对一关系

对一关系是指当前对象关联一个其他对象。例如：每个订单项只属于一个订单。

```text
OrderItemVo（订单项） ─────> Order（订单）
               多         一
```

VO 示例：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class OrderItemVo {
    private Integer id;
    private String goodsName;
    private BigDecimal goodsPrice;
    private Integer count;
    private Integer orderId;

    // 对一关系
    private Order order;
}
```

MyBatis 使用 `<association>` 封装对一关系，常见实现方式有两种。

### 2. 方案一：一条 SQL 联表查询

通过联表查询一次获得订单项及所属订单，再使用 `<association>` 映射关联对象。

```xml
<resultMap id="orderItemVoMap" type="OrderItemVo">
    <id column="item_id" property="id"/>
    <result column="goods_name" property="goodsName"/>
    <result column="goods_price" property="goodsPrice"/>
    <result column="item_count" property="count"/>
    <result column="item_order_id" property="orderId"/>

    <association property="order" javaType="Order">
        <id column="order_id" property="id"/>
        <result column="payment" property="payment"/>
        <result column="payment_type" property="paymentType"/>
        <result column="order_status" property="status"/>
    </association>
</resultMap>

<select id="selectWithOrderById" resultMap="orderItemVoMap">
    select i.id         as item_id,
           i.goods_name,
           i.goods_price,
           i.count      as item_count,
           i.order_id   as item_order_id,
           o.id         as order_id,
           o.payment,
           o.payment_type,
           o.status     as order_status
    from tb_order_item i
    inner join tb_order o on i.order_id = o.id
    where i.id = #{id}
</select>
```

`<association>` 的关键属性：

| 属性 | 含义 |
|---|---|
| `property` | VO 中的关联对象属性名 |
| `javaType` | 关联对象的 Java 类型 |

### 3. 方案二：两条 SQL 嵌套查询

先查询订单项，再使用订单外键调用另一条 SQL 查询订单。

```xml
<resultMap id="orderItemVoMap2" type="OrderItemVo">
    <id column="id" property="id"/>
    <result column="goods_name" property="goodsName"/>
    <result column="goods_price" property="goodsPrice"/>
    <result column="count" property="count"/>
    <result column="order_id" property="orderId"/>

    <association
        property="order"
        select="com.edu.mapper.OrderMapper.selectById"
        column="order_id"/>
</resultMap>

<select id="selectWithOrderById2" resultMap="orderItemVoMap2">
    select id, goods_name, goods_price, count, order_id
    from tb_order_item
    where id = #{id}
</select>
```

订单查询：

```xml
<select id="selectById" resultType="Order">
    select id, payment, payment_type, status
    from tb_order
    where id = #{id}
</select>
```

| 属性 | 作用 |
|---|---|
| `select` | 指定用于查询关联对象的 Mapper SQL |
| `column` | 将当前结果中的字段作为子查询参数 |

### 4. 两种方案对比

| 对比项 | 联表查询 | 嵌套查询 |
|---|---|---|
| SQL 数量 | 通常 1 条 | 主查询 + 关联查询 |
| 映射配置 | `association + javaType` | `association + select + column` |
| 懒加载 | 不属于嵌套查询式懒加载 | 可以配置懒加载 |
| N+1 风险 | 通常没有 | 批量查询时可能出现 |
| 适用场景 | 一次需要完整数据 | 关联数据不一定使用 |

> 查询多个订单项时，嵌套查询可能产生一次主查询加 N 次订单查询，即 N+1 问题。

---

## 二、PageHelper 分页插件

### 1. PageHelper 的作用

PageHelper 是 MyBatis 分页插件，可拦截查询并根据数据库方言生成分页 SQL，从而屏蔽部分数据库分页语法差异。

```text
普通查询 SQL
    ↓ PageHelper 拦截
生成对应数据库的分页 SQL
    ↓
返回当前页数据和分页信息
```

### 2. 引入依赖

导图中的版本示例：

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.2.1</version>
</dependency>
```

### 3. 注册插件

在 `mybatis-config.xml` 中添加：

```xml
<plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <property name="helperDialect" value="mysql"/>
    </plugin>
</plugins>
```

> `<plugins>` 在 MyBatis 核心配置中有固定位置，应按照配置文件规定的标签顺序放置。

### 4. Mapper 方法

Mapper 方法本身通常不需要添加分页参数：

```java
List<Brand> selectAll();
```

```xml
<select id="selectAll" resultType="Brand">
    select * from tb_brand order by id
</select>
```

### 5. 使用 PageHelper

必须紧接着执行需要分页的查询：

```java
PageHelper.startPage(pageNum, pageSize);
List<Brand> brands = brandMapper.selectAll();
PageInfo<Brand> pageInfo = new PageInfo<>(brands);
```

完整结构：

```java
public PageInfo<Brand> selectPage(int pageNum, int pageSize) {
    PageHelper.startPage(pageNum, pageSize);
    List<Brand> brands = brandMapper.selectAll();
    return new PageInfo<>(brands);
}
```

### 6. Page 与 PageInfo

`Page` 是 `ArrayList` 的子类，既保存当前页列表，也包含分页信息。

如果 Mapper 返回 `List`，常使用 `PageInfo` 包装：

```java
List<Brand> list = brandMapper.selectAll();
PageInfo<Brand> pageInfo = new PageInfo<>(list);
```

如果实际结果对象是 `Page`，也可以直接获取：

```java
Page<Brand> page = (Page<Brand>) brandMapper.selectAll();
long total = page.getTotal();
List<Brand> result = page.getResult();
```

常用分页信息：

| 信息 | 含义 |
|---|---|
| `pageNum` | 当前页码 |
| `pageSize` | 每页条数 |
| `total` | 总记录数 |
| `pages` | 总页数 |
| `list` / `result` | 当前页数据 |

> `PageHelper.startPage()` 只影响其后紧邻的第一条符合条件的查询，不要在两者之间执行其他查询。

---

# 第二部分：Spring 入门

## 三、Spring 概述

### 1. 广义 Spring

广义上的 Spring 指 Spring 技术生态，也常称为 Spring 全家桶，包含多个项目，例如 Spring Framework、Spring Boot、Spring Data 等。

### 2. 狭义 Spring

狭义上的 Spring 指 **Spring Framework**，它是 Spring 生态中最基础的项目，为其他项目提供 IoC、AOP、事务管理等核心能力。

---

## 四、IoC、Bean 与 DI

### 1. IoC：控制反转

IoC（Inversion of Control）是将对象创建及对象依赖关系的管理权，从业务代码交给 Spring 容器。

传统方式：

```java
BookDao bookDao = new BookDaoImpl();
BookService bookService = new BookServiceImpl(bookDao);
```

IoC 方式：

```text
Spring 容器创建 BookDao
        ↓
Spring 容器创建 BookService
        ↓
Spring 将 BookDao 注入 BookService
```

### 2. Bean

由 Spring IoC 容器创建、配置和管理的对象称为 Bean。

### 3. DI：依赖注入

DI（Dependency Injection）是实现 IoC 思想的重要技术手段。容器在创建对象后，把该对象需要的依赖注入进去。

三者关系：

```text
IoC：设计思想——控制权交给容器
Bean：容器管理的对象
DI：容器建立 Bean 依赖关系的手段
```

---

## 五、Bean 注册

### 1. 组件注解

Spring 提供以下注解将类注册为 Bean：

| 注解 | 常用位置 |
|---|---|
| `@Component` | 通用组件或三层之外的类 |
| `@Controller` | 表现层 |
| `@Service` | 业务层 |
| `@Repository` | 持久层 |

这四个注解在注册 Bean 的能力上相同，后三个主要用于表达更明确的分层语义。

```java
@Service
public class BookServiceImpl implements BookService {
}
```

默认 Bean 名称通常是类名首字母小写：

```text
BookServiceImpl → bookServiceImpl
```

也可以显式指定名称：

```java
@Service("bookService")
public class BookServiceImpl implements BookService {
}
```

### 2. 组件扫描

配置类需要指定扫描范围：

```java
@Configuration
@ComponentScan(basePackages = "com.edu")
public class SpringConfig {
}
```

该配置会扫描 `com.edu` 包及其所有子包中的组件注解。

> 只有处于组件扫描范围内的类，才能通过上述注解被自动注册。

---

## 六、IoC 容器

### 1. ApplicationContext

`ApplicationContext` 是 Spring IoC 容器的核心接口。

### 2. XML 配置容器

`ClassPathXmlApplicationContext` 根据类路径中的 XML 配置创建容器：

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");
```

### 3. 注解配置容器

`AnnotationConfigApplicationContext` 根据 Java 配置类创建容器：

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(SpringConfig.class);
```

| 实现类 | 配置来源 |
|---|---|
| `ClassPathXmlApplicationContext` | 类路径下的 Spring XML 文件 |
| `AnnotationConfigApplicationContext` | Java 注解配置类 |

---

## 七、获取 Bean

### 1. 根据类型获取

```java
BookService bookService = context.getBean(BookService.class);
```

- 不需要类型强制转换。
- 要求该类型通常只能匹配到一个候选 Bean，否则会产生冲突。

### 2. 根据名称获取

```java
BookService bookService =
        (BookService) context.getBean("bookServiceImpl");
```

- 使用 Bean 名称定位。
- 返回类型为 `Object`，需要强制转换。

### 3. 根据名称和类型获取

```java
BookService bookService = context.getBean(
        "bookServiceImpl",
        BookService.class
);
```

- 不需要强制转换。
- 同类型存在多个 Bean 时，可通过名称精确选择。

### 4. 三种方式对比

| 获取方式 | 是否强转 | 特点 |
|---|:---:|---|
| `getBean(Type.class)` | 否 | 简洁，但同类型多 Bean 时冲突 |
| `getBean("name")` | 是 | 按名称精确获取 |
| `getBean("name", Type.class)` | 否 | 同时校验名称和类型，较明确 |

---

## 八、依赖注入

### 1. `@Autowired`

`@Autowired` 是 Spring 提供的依赖注入注解，默认以类型为主要匹配依据。

```java
@Service
public class BookServiceImpl implements BookService {

    @Autowired
    private BookDao bookDao;
}
```

常见匹配过程：

1. 按类型查找候选 Bean。
2. 若只有一个候选项，直接注入。
3. 若存在多个候选项，需要通过名称、`@Qualifier`、`@Primary` 等进一步确定。

结合 `@Qualifier` 指定 Bean 名称：

```java
@Autowired
@Qualifier("bookDaoMysql")
private BookDao bookDao;
```

更推荐构造器注入：

```java
@Service
public class BookServiceImpl implements BookService {

    private final BookDao bookDao;

    public BookServiceImpl(
            @Qualifier("bookDaoMysql") BookDao bookDao
    ) {
        this.bookDao = bookDao;
    }
}
```

构造器注入便于创建不可变依赖，也更方便测试。

### 2. `@Resource`

`@Resource` 属于 Jakarta 注解规范，通常优先按名称匹配，也可以通过 `name` 明确指定：

```java
@Resource(name = "bookDaoMysql")
private BookDao bookDao;
```

在使用 Jakarta 命名空间的项目中，依赖示例：

```xml
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>
```

> 从较新的 JDK 版本开始，相关注解不再随 JDK 默认提供，需要由项目依赖显式引入。旧版项目也可能使用 `javax.annotation.Resource`。

### 3. `@Autowired` 与 `@Resource` 对比

| 对比项 | `@Autowired` | `@Resource` |
|---|---|---|
| 来源 | Spring | Jakarta 注解规范 |
| 主要匹配方式 | 按类型 | 通常按名称 |
| 指定名称 | 配合 `@Qualifier` | 使用 `name` 属性 |
| 示例 | `@Autowired @Qualifier("x")` | `@Resource(name = "x")` |

---

## 九、外部属性注入

### 1. 加载属性文件

创建 `jdbc.properties`：

```properties
jdbc.username=root
jdbc.age=18
```

在配置类中加载：

```java
@Configuration
@ComponentScan("com.edu")
@PropertySource("classpath:jdbc.properties")
public class SpringConfig {
}
```

### 2. 使用 `@Value` 注入

`@Value` 可注入字符串、基本类型及其包装类型等简单值：

```java
@Value("${jdbc.username}")
private String username;

@Value("${jdbc.age}")
private int age;
```

### 3. 设置默认值

占位符可以使用冒号设置默认值：

```java
@Value("${jdbc.username2:lisi}")
private String username;
```

含义：若配置中存在 `jdbc.username2`，则使用配置值；否则使用 `lisi`。

| 注解 | 作用 |
|---|---|
| `@PropertySource` | 加载外部属性文件 |
| `@Value("${key}")` | 读取指定属性 |
| `@Value("${key:默认值}")` | 属性不存在时使用默认值 |

---

## 十、重点速记

> [!TIP]
> **MyBatis 的 `association` 负责对一映射；Spring 的 IoC 负责创建 Bean，DI 负责建立 Bean 之间的依赖。**

- 对一关系使用 `<association>`。
- 联表查询：`association + javaType`，通常只执行一条 SQL。
- 嵌套查询：`association + select + column`，支持配置懒加载。
- 嵌套查询批量加载时要警惕 N+1 问题。
- PageHelper 在查询前调用 `startPage(pageNum, pageSize)`。
- PageHelper 只影响后面紧邻的第一条查询。
- `Page` 是 `ArrayList` 的子类，可保存分页信息。
- Mapper 返回 `List` 时，可使用 `PageInfo` 包装分页结果。
- 广义 Spring 是技术生态，狭义 Spring 指 Spring Framework。
- IoC：将对象创建和依赖管理交给容器。
- Bean：由 Spring 容器管理的对象。
- DI：容器向 Bean 注入其所需依赖。
- `@Component`、`@Controller`、`@Service`、`@Repository` 均可注册 Bean。
- `@ComponentScan` 指定组件扫描范围。
- `ApplicationContext` 是 Spring IoC 容器核心接口。
- `@Autowired` 主要按类型注入，多实现时可配合 `@Qualifier`。
- `@Resource` 通常按名称注入，可通过 `name` 指定 Bean。
- `@PropertySource` 加载属性文件，`@Value` 注入简单属性值。
- `${key:默认值}` 可在配置缺失时使用默认值。

---

> 本文根据思维导图 **《mybatis结尾和spring框架开头》** 整理，对 MyBatis 对一映射、分页和 Spring IoC / DI 入门知识进行了结构化归纳。
