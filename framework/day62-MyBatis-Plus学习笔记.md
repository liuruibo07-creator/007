# Day 62｜MyBatis-Plus 学习笔记

> **主题：** MyBatis-Plus 入门、CRUD、分页、条件构造器、实体类注解与逻辑删除  
> **核心定位：** MyBatis-Plus（简称 MP）是一款持久层增强框架，用于简化 MyBatis 开发，尤其适合快速完成单表 CRUD 操作。

---

## 目录

- [一、MyBatis-Plus 概述](#一mybatis-plus-概述)
- [二、入门案例](#二入门案例)
- [三、常用 CRUD 操作](#三常用-crud-操作)
- [四、分页查询](#四分页查询)
- [五、条件构造器 Wrapper](#五条件构造器-wrapper)
- [六、实体类常用注解](#六实体类常用注解)
- [七、逻辑删除](#七逻辑删除)
- [八、知识点速记](#八知识点速记)

---

## 一、MyBatis-Plus 概述

MyBatis-Plus 是一个持久层框架，在 MyBatis 的基础上提供增强能力，主要作用包括：

- 减少重复的 SQL 与 Mapper 编写工作；
- 简化单表的新增、删除、修改和查询；
- 提供分页、条件构造器、逻辑删除等常用功能。

---

## 二、入门案例

### 2.1 创建 Spring Boot 工程

可以使用 Spring Initializr 脚手架创建工程：<https://start.springboot.io>

### 2.2 引入 MyBatis-Plus 依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.17</version>
</dependency>
```

### 2.3 修改 YAML 配置

配置文件中主要完成两项工作：

1. 配置数据库连接四要素；
2. 配置日志实现，使控制台能够打印执行的 SQL 语句。

### 2.4 定义实体类

实体类可以使用 Lombok 提供的 `@Builder` 注解，以构建器模式创建对象，并支持链式设置属性：

```java
User user = User.builder()
        .id(1L)
        .age(19)
        .password("123")
        .build();
```

`@Builder` 的基本原理：

- 外部类中生成一个 `builder()` 方法，用于返回静态内部类 `XxxBuilder` 的对象；
- `XxxBuilder` 中包含外部类的各个属性；
- 内部类中生成与属性同名的方法，用于设置值并返回当前 Builder，因此能够链式调用；
- 调用 `build()` 后返回构建完成的外部类对象。

### 2.5 定义 Mapper 接口

Mapper 接口继承 `BaseMapper<实体类型>`，即可获得常用的单表 CRUD 能力：

```java
public interface UserMapper extends BaseMapper<User> {
}
```

---

## 三、常用 CRUD 操作

| 功能 | `BaseMapper` 常用接口 | 说明 |
| --- | --- | --- |
| 新增 | `int insert(T entity)` | 插入一条记录 |
| 根据 ID 删除 | `int deleteById(Serializable id)` | 按主键删除 |
| 根据 ID 修改 | `int updateById(T entity)` | 按主键更新非空字段 |
| 根据 ID 查询 | `T selectById(Serializable id)` | 按主键查询 |
| 查询全部 | `List<T> selectList(null)` | 不设置条件时查询全部 |
| 分页查询 | `IPage<T> selectPage(IPage<T> page, Wrapper<T> wrapper)` | 查询指定页数据 |

批量删除可以使用：

```java
userMapper.deleteByIds(idCollection);
```

其中，`idCollection` 是只包含主键值的单列集合 `Collection`。

---

## 四、分页查询

### 4.1 引入分页插件依赖

从 MyBatis-Plus `3.5.9` 开始，分页插件被单独拆分，需要额外引入 `mybatis-plus-jsqlparser`。其版本应与 MyBatis-Plus 起步依赖保持一致：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-jsqlparser</artifactId>
    <version>3.5.17</version>
</dependency>
```

### 4.2 注册分页拦截器

- `MybatisPlusInterceptor`：拦截器容器，本身不承担具体功能；
- `PaginationInnerInterceptor`：内部拦截器，负责分页功能。

```java
@Configuration
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }
}
```

### 4.3 ==执行分页查询==

```java
Page<User> page = new Page<>(pageNumber, pageSize);
Page<User> result = userMapper.selectPage(page, null);

long total = result.getTotal();
List<User> records = result.getRecords();
```

- `new Page<>(页码, 每页记录数)`：创建分页对象；
- `getTotal()`：获取符合条件的总记录数；
- `getRecords()`：获取当前页的数据列表。

---

## 五、条件构造器 Wrapper

Wrapper 用于以链式 API 构造 SQL 条件。

| 类型 | 作用 |
| --- | --- |
| `QueryWrapper` | 生成查询或删除操作所需的 `WHERE` 条件 |
| `UpdateWrapper` | 生成更新操作所需的 `WHERE` 条件和 `SET` 语句 |

### 5.1 条件更新

方式一：在 `UpdateWrapper` 中同时设置更新条件与新值。

```java
UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
updateWrapper.eq("id", 2);
updateWrapper.set("age", 18);
userMapper.update(updateWrapper);
```

方式二：实体对象提供更新值，`UpdateWrapper` 提供更新条件。

```java
UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
updateWrapper.eq("id", 2);

User user = User.builder()
        .age(19)
        .password("123")
        .build();

userMapper.update(user, updateWrapper);
```

### 5.2 条件删除

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("id", 2);
userMapper.delete(queryWrapper);
```

### 5.3 多条件组合

默认情况下，连续添加的条件使用 `AND` 连接：

```java
queryWrapper.eq("id", 2).eq("age", 18);
```

使用 `or()` 构造“或”关系：

```java
queryWrapper.eq("id", 2).or().eq("age", 18);
```

使用嵌套的 `and(...)` 为一组条件添加括号、提升其运算优先级：

```java
queryWrapper
        .and(w -> w.eq("id", 2).or().eq("age", 18))
        .eq("password", "123");
```

对应的条件逻辑为：

```sql
(id = 2 OR age = 18) AND password = '123'
```

### 5.4 动态条件与空值判断

设置条件时，可以使用带 `condition` 参数的重载方法。只有条件为 `true` 时，MyBatis-Plus 才会拼接对应 SQL：

```java
queryWrapper.eq(
        StrUtil.isNotBlank(user.getPassword()),
        "password",
        user.getPassword()
);
```

这里使用 Hutool 的 `StrUtil.isNotBlank(...)` 判断密码是否为非空白字符串。

### 5.5 查询投影

`select(...)` 用于指定需要查询的字段；不调用时，默认查询全部字段。

```java
queryWrapper.select("id", "username", "age");
```

### 5.6 常用条件运算符

```java
queryWrapper.orderByDesc("age").orderByAsc("id");
queryWrapper.likeLeft("username", "li");  // username LIKE '%li'
queryWrapper.notLike("username", "li");   // username NOT LIKE '%li%'
queryWrapper.between("age", 18, 20);       // age BETWEEN 18 AND 20
queryWrapper.in("id", 1, 2, 3);            // id IN (1, 2, 3)
```

更多 Wrapper 用法：<https://baomidou.com/guides/wrapper/>

---

## 六、实体类常用注解

### 6.1 `@TableName`

用于指定实体类对应的数据库表名：

```java
@TableName("user")
public class User {
}
```

### 6.2 `@TableField`

用于配置实体属性与数据库字段的映射关系。

| 配置 | 作用 |
| --- | --- |
| `value = "字段名"` | 指定属性对应的数据库字段名 |
| `exist = false` | 表示该属性在数据库表中没有对应字段 |
| `select = false` | 表示查询时不返回该字段 |

> 原导图中的 `@TableFiled` 应为 `@TableField`。

### 6.3 `@TableId`

用于标记主键字段，`type` 属性用于设置主键生成策略。

| `IdType` 策略 | 说明 |
| --- | --- |
| `AUTO` | 使用数据库自增策略 |
| `ASSIGN_ID` | 默认使用雪花算法生成全局唯一 ID |

`ASSIGN_ID` 生成的 ID 使用 64 位二进制描述，转为十进制后通常是 19 位 `long` 类型数值，因此数据库字段一般使用 `BIGINT`。

---

## 七、逻辑删除

逻辑删除不是物理移除记录，而是通过字段标识记录的删除状态。

### 实现步骤

1. 在数据库表中添加逻辑删除标识字段，例如 `deleted`；
2. 推荐使用 `0` 表示未删除、`1` 表示已删除，并将默认值设为 `0`；
3. 在实体类中添加对应属性，并使用 `@TableLogic` 注解；
4. 执行删除操作时，MyBatis-Plus 自动将其转换为更新标识状态的 `UPDATE`；
5. 执行查询操作时，MyBatis-Plus 自动过滤已逻辑删除的数据。

```java
@TableLogic
private Integer deleted;
```

> **理解重点：** 逻辑删除后的数据仍然存在于数据库中，只是在常规查询中被自动排除。

---

## 八、知识点速记

> [!TIP]
> - `BaseMapper<T>`：提供通用单表 CRUD。
> - `QueryWrapper`：构造查询、删除等操作的 `WHERE` 条件。
> - `UpdateWrapper`：构造更新操作的 `WHERE` 与 `SET`。
> - `Page`：封装分页参数和分页结果。
> - `MybatisPlusInterceptor`：内部拦截器的容器。
> - `PaginationInnerInterceptor`：真正执行分页处理。
> - `@TableName`：表名映射。
> - `@TableField`：字段映射与查询控制。
> - `@TableId`：主键及其生成策略。
> - `@TableLogic`：启用逻辑删除。

### 一句话串联

**实体类负责数据映射，`BaseMapper` 提供基础 CRUD，Wrapper 负责拼接条件，分页拦截器负责分页，`@TableLogic` 负责逻辑删除。**
