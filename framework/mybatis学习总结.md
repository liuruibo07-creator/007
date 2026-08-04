# Java Web 与 MyBatis 学习总结（Day 48）

> 本节核心内容：**Filter 过滤器、ServletContextListener 监听器、MyBatis 入门与代理开发、参数占位、结果映射、日志和 Lombok**。

## 目录

- [一、Filter 过滤器](#一filter-过滤器)
- [二、Listener 监听器](#二listener-监听器)
- [三、MyBatis 概述](#三mybatis-概述)
- [四、MyBatis 入门案例](#四mybatis-入门案例)
- [五、Mapper 代理开发](#五mapper-代理开发)
- [六、MyBatis 核心配置](#六mybatis-核心配置)
- [七、参数占位符](#七参数占位符)
- [八、字段名与属性名映射](#八字段名与属性名映射)
- [九、MyBatis 日志](#九mybatis-日志)
- [十、Lombok](#十lombok)
- [十一、重点速记](#十一重点速记)

---

## 一、Filter 过滤器

### 1. Filter 的作用

Filter 可以拦截进入 Web 应用的请求和返回客户端的响应，适合抽取多个资源共同需要的功能。

常见应用场景：

- 登录与权限校验。
- 请求字符编码处理。
- 日志记录。
- 敏感词过滤。
- 响应数据统一处理。

```text
客户端
  ↓ 请求
Filter
  ↓
Servlet / 目标资源
  ↓ 响应
Filter
  ↓
客户端
```

### 2. 定义 Filter

1. 定义类并实现 `Filter` 接口。
2. 重写 `doFilter()` 方法。
3. 使用 `@WebFilter` 配置拦截路径。
4. 调用 `chain.doFilter()` 放行请求。

```java
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;

import java.io.IOException;
//*为拦截所有url
@WebFilter("/*")
public class EncodingFilter implements Filter {

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {

        // 放行前：处理请求
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");

        // 放行，进入下一个过滤器或目标资源
        chain.doFilter(request, response);

        // 放行后：处理响应
    }
}
```

> 若没有调用 `chain.doFilter(request, response)`，请求将被当前过滤器拦截，不会继续访问目标资源。

### 3. 过滤器链

同一个请求匹配多个 Filter 时，这些 Filter 会组成过滤器链。

假设执行顺序为 `Filter1 → Filter2 → Servlet`：

```text
Filter1 放行前代码
    ↓
Filter2 放行前代码
    ↓
Servlet 的 doGet()/doPost()
    ↓
Filter2 放行后代码
    ↓
Filter1 放行后代码
```

执行结构类似方法嵌套：

```text
Filter1 {
    Filter2 {
        Servlet
    }
}
```

### 4. 多个 Filter 的顺序

使用注解配置时，部分 Servlet 容器会按过滤器类名的自然顺序组织过滤器链，例如字符编码顺序通常表现为：

```text
0-9 → A-Z → a-z
```

其常见起始编码值为：

| 字符范围 | 起始编码值 |
|---|:---:|
| `0-9` | 48 |
| `A-Z` | 65 |
| `a-z` | 97 |

> 注解 Filter 的顺序不应作为跨容器的可靠业务规则。若业务严格依赖过滤器顺序，应使用项目框架提供的顺序机制，或采用可明确控制顺序的配置方式。

---

## 二、Listener 监听器

Listener 用于监听 Web 应用中对象的创建、销毁或属性变化。

本节重点是 `ServletContextListener`。

### 1. ServletContext

`ServletContext` 表示整个 Web 应用：

- 一个 Web 应用通常只有一个 `ServletContext` 对象。
- 应用启动时创建。
- 应用关闭时销毁。
- 可用于读取应用资源、获取上下文信息或保存应用级共享数据。

### 2. 定义 ServletContextListener

```java
import jakarta.servlet.ServletContextEvent;
import jakarta.servlet.ServletContextListener;
import jakarta.servlet.annotation.WebListener;

@WebListener
public class ApplicationListener
        implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent event) {
        // Web 应用启动时执行
        System.out.println("应用已启动");
    }

    @Override
    public void contextDestroyed(ServletContextEvent event) {
        // Web 应用正常关闭时执行
        System.out.println("应用已关闭");
    }
}
```

| 方法 | 执行时机 |
|---|---|
| `contextInitialized()` | Web 应用初始化完成时 |
| `contextDestroyed()` | Web 应用销毁时 |

常见用途：应用启动时加载配置、初始化资源，应用关闭时释放资源。

---

## 三、MyBatis 概述

MyBatis 是一个**持久层框架**，用于简化 JDBC 数据库操作。

使用原生 JDBC 通常需要手动完成：

- 获取数据库连接。
- 创建并执行 SQL 对象。
- 设置参数。
- 遍历结果集。
- 将字段封装成 Java 对象。
- 释放数据库资源。

MyBatis 将这些重复工作封装起来，开发者主要关注：

```text
SQL + 参数 + 返回值映射
```

---

## 四、MyBatis 入门案例

### 1. 准备项目

建议先检查：

- 项目使用的 JDK 版本。
- 文件字符编码。
- IDEA 自动导包和删除无用导包设置。
- 代码提示是否忽略大小写。
- Maven 安装目录、`settings.xml` 和本地仓库。

### 2. 创建 Maven 模块并引入依赖

核心依赖包括 MyBatis 和 MySQL 驱动。以下版本仅作结构示例，应按项目要求确定具体版本：

```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>项目指定版本</version>
    </dependency>

    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>项目指定版本</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.30</version>
    </dependency>

    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>项目指定版本</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 3. 定义实体类

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Integer id;
    private String username;
    private String password;
}
```

### 4. 创建 jdbc.properties

建议将数据库连接信息单独放入 `src/main/resources/jdbc.properties`：

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.username=root
jdbc.password=1234
```

数据库连接四要素：

| 配置 | 示例 |
|---|---|
| 驱动类 | `com.mysql.cj.jdbc.Driver` |
| URL | `jdbc:mysql://localhost:3306/mybatis` |
| 用户名 | `root` |
| 密码 | `1234` |

> 实际项目不应将生产数据库密码提交到公开代码仓库。

### 5. 创建 mybatis-config.xml

核心配置文件通常位于 `src/main/resources`：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <properties resource="jdbc.properties"/>

    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

    <typeAliases>
        <package name="com.edu.entity"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <package name="com.edu.mapper"/>
    </mappers>
</configuration>
```

### 6. 定义 Mapper 接口

```java
package com.edu.mapper;

import com.edu.entity.User;
import java.util.List;

public interface UserMapper {
    List<User> selectAll();
}
```

### 7. 定义 SQL 映射文件

创建 `UserMapper.xml`：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.edu.mapper.UserMapper">
    <select id="selectAll" resultType="User">
        select id, username, password
        from user
    </select>
</mapper>
```

### 8. 编写单元测试

```java
@Test
void selectAll() throws IOException {
    //1.加载配置文件,从classpath下加载  
String resource = "mybatis-config.xml";  
InputStream inputStream = Resources.getResourceAsStream(resource);  
//2.创建SqlSessionFactory对象  
SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(inputStream);  
//3.通过工厂生产SqlSession对象  
SqlSession sqlSession=sqlSessionFactory.openSession();  
//4.通过sqlSession创建代理对象  
UserMapper mapper = sqlSession.getMapper(UserMapper.class);  
//5.执行方法  
List<User> users = mapper.selectAll();  
System.out.println(users);  
//6.关闭资源  
sqlSession.close();

    
}
```

执行主线：

```text
读取核心配置
    ↓
创建 SqlSessionFactory
    ↓
打开 SqlSession
    ↓
获取 Mapper 代理对象
    ↓
调用方法执行 SQL
    ↓
提交事务（写操作需要）并关闭资源
```

---

## 五、Mapper 代理开发

### 1. 代理开发的含义

开发者只需要定义 Mapper 接口和 SQL 映射，MyBatis 会在运行时为接口生成实现对象，即 Mapper 代理对象。

```java
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
```

不需要手动编写 `UserMapperImpl`。

### 2. Mapper 接口与 XML 的对应规则

| Mapper XML | Mapper 接口 | 要求 |
|---|---|---|
| `namespace` | 接口全限定名 | 必须一致 |
| SQL 标签的 `id` | 接口方法名 | 必须一致 |
| `resultType` / `resultMap` | 方法返回值类型 | 必须匹配 |
| SQL 参数 | 方法形参 | 类型与名称应正确对应 |

示例：

```xml
<mapper namespace="com.edu.mapper.UserMapper">
    <select id="selectById" resultType="User">
        select * from user where id = #{id}
    </select>
</mapper>
```

```java
public interface UserMapper {
    User selectById(Integer id);
}
```

`parameterType` 通常可以省略，因为 MyBatis 能根据 Mapper 方法参数推断。

---

## 六、MyBatis 核心配置

### 1. 使用包扫描加载 Mapper

```xml
<mappers>
    <package name="com.edu.mapper"/>
</mappers>
```

使用包扫描时，通常要求：

- Mapper 接口名和 XML 文件名相同。
- XML 在资源目录中拥有与接口包名相同的目录结构。

例如：

```text
Java 接口：src/main/java/com/edu/mapper/UserMapper.java
映射文件：src/main/resources/com/edu/mapper/UserMapper.xml
```

### 2. 设置类型别名

```xml
<typeAliases>
    <package name="com.edu.entity"/>
</typeAliases>
```

配置后，包内类可以使用短类名：

```xml
<select id="selectAll" resultType="User">
```

MyBatis 类型别名通常不区分大小写，但建议保持与类名一致，以提高可读性。

### 3. 加载属性文件

```xml
<properties resource="jdbc.properties"/>
```

读取属性值：

```xml
<property name="url" value="${jdbc.url}"/>
```

> 配置文件中的 `${}` 表示读取配置属性；SQL 中的 `${}` 则是字符串替换，两者所处语境不同。

### 4. 核心配置顺序

MyBatis 配置文件的顶层标签有固定顺序。常见排列为：

```text
properties
settings
typeAliases
environments
mappers
```

标签顺序不正确时，配置文件可能无法通过校验。

---

## 七、参数占位符

`#{}` 和 `${}` 都能向 SQL 传递内容，但处理方式和安全性不同。

### 1. `#{}`：参数占位

```xml
select * from user where id = #{id}
```

执行效果类似：

```sql
select * from user where id = ?
```

特点：

- 使用预编译参数绑定。
- 自动处理大多数值的类型和引号。
- 能有效降低 SQL 注入风险。
- 传递普通数据值时应优先使用。

### 2. `${}`：字符串替换

```xml
select * from user order by ${columnName}
```

MyBatis 会先将内容直接拼接到 SQL 文本中。

特点：

- 不生成参数占位符。
- 不会自动添加引号。
- 存在 SQL 注入风险。
- 仅在表名、列名、排序方向等无法使用 `?` 占位的位置谨慎使用。

> 即使底层执行对象可能仍为 `PreparedStatement`，`${}` 的内容也已经提前拼入 SQL，因此不能获得 `#{}` 的参数绑定安全性。

### 3. 对比总结

| 对比项 | `#{}` | `${}` |
|---|---|---|
| 处理方式 | 参数占位与绑定 | 原样字符串替换 |
| SQL 中表现 | `?` | 直接拼接内容 |
| 自动处理引号 | 是 | 否 |
| SQL 注入风险 | 较低 | 较高 |
| 常见用途 | 普通条件值 | 动态表名、列名等结构部分 |

记忆口诀：

> **传数据用 `#{}`，拼结构才考虑 `${}`。**

---

## 八、字段名与属性名映射

数据库常使用下划线命名，Java 属性通常使用小驼峰命名：

```text
数据库字段：brand_name
Java 属性：brandName
```

若名称不一致，可使用以下三种方式解决。

### 1. SQL 别名

```sql
select id,
       brand_name as brandName,
       company_name as companyName,
       ordered,
       description,
       status
from tb_brand
where id = #{id}
```

优点：简单直接。缺点：每条相关 SQL 可能都需要重复书写别名。

### 2. resultMap 映射

```xml
<resultMap id="brandMap" type="Brand">
    <id column="id" property="id"/>
    <result column="brand_name" property="brandName"/>
    <result column="company_name" property="companyName"/>
</resultMap>

<select id="selectById" resultMap="brandMap">
    select id,
           brand_name,
           company_name,
           ordered,
           description,
           status
    from tb_brand
    where id = #{id}
</select>
```

| 属性 | 含义 |
|---|---|
| `column` | 数据库字段名 |
| `property` | Java 实体类属性名 |

`resultMap` 适合复杂映射或需要重复使用映射规则的场景。

### 3. 开启下划线转驼峰

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

开启后可自动转换：

```text
brand_name → brandName
company_name → companyName
```

### 4. 三种方案对比

| 方案 | 适用场景 |
|---|---|
| SQL 别名 | 少量字段、单条 SQL 快速处理 |
| `resultMap` | 复杂映射、关联映射、复用映射规则 |
| 下划线转驼峰 | 项目统一采用数据库下划线与 Java 驼峰规范 |

---

## 九、MyBatis 日志

日志可以显示执行的 SQL、参数和查询结果，便于调试。

### 1. 标准输出日志

在 `<settings>` 中配置：

```xml
<setting name="logImpl" value="STDOUT_LOGGING"/>
```

优点是配置简单，适合入门调试；正式项目通常使用专门日志框架。

### 2. 集成 SLF4J / Logback

导图中的依赖示例：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```

MyBatis 配置：

```xml
<setting name="logImpl" value="SLF4J"/>
```

> 实际项目应统一选择与当前 Java 版本、SLF4J 版本及项目依赖兼容的日志组件版本。

---

## 十、Lombok

Lombok 通过注解在编译阶段生成重复代码，可以简化实体类和日志记录器的书写。

### 1. 引入依赖

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
</dependency>
```

### 2. 简化实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Integer id;
    private String username;
}
```

| 注解 | 作用 |
|---|---|
| `@Data` | 生成 Getter、Setter、`toString()`、`equals()`、`hashCode()` 等 |
| `@NoArgsConstructor` | 生成无参构造方法 |
| `@AllArgsConstructor` | 生成包含全部字段的构造方法 |

### 3. 简化日志记录器

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class UserService {

    public void execute() {
        log.info("开始执行业务");
    }
}
```

`@Slf4j` 会自动生成名为 `log` 的日志记录器。

---

## 十一、重点速记

> [!TIP]
> **Filter 处理通用 Web 逻辑，Listener 监听应用生命周期，MyBatis 负责持久层数据访问。**

- `@WebFilter("/*")`：拦截当前应用中的全部路径。
- `chain.doFilter()`：放行到下一个过滤器或目标资源。
- 过滤器执行遵循“先进后出”：放行前顺序执行，放行后逆序执行。
- `ServletContext` 代表整个 Web 应用。
- `contextInitialized()`：应用启动时执行。
- `contextDestroyed()`：应用关闭时执行。
- MyBatis 是持久层框架，用于简化 JDBC。
- Mapper 代理：只定义接口和 XML，MyBatis 自动生成接口代理对象。
- `namespace` 对应 Mapper 接口全限定名。
- SQL 标签 `id` 对应接口方法名。
- `sqlSession.getMapper(XxxMapper.class)` 获取 Mapper 代理对象。
- 包扫描要求接口与 XML 名称相同，且资源目录结构与接口包一致。
- `<properties>` 加载数据库属性文件。
- `<typeAliases>` 简化实体类全限定名。
- `#{}` 是安全的参数绑定；`${}` 是直接字符串替换。
- 字段映射可用 SQL 别名、`resultMap` 或下划线转驼峰。
- `STDOUT_LOGGING` 适合简单调试，正式项目通常使用日志框架。
- Lombok 的正确注解名是 `@AllArgsConstructor`。

---

> 本文根据思维导图 **《mybatis》（Day 48）** 整理，并对原有知识点进行了结构化归纳和方法名称规范化。
