# Day 42｜JDBC 学习总结

> **学习主线：** JDBC 基础 → 核心 API → 事务控制 → SQL 预编译 → 数据库连接池 → 工具类封装

---

## 一、JDBC 概述

JDBC（Java Database Connectivity）是 Java 提供的一套用于操作**关系型数据库**的标准接口。

- Java 定义统一的 JDBC 接口与规范。
- 各数据库厂商提供对应的驱动 JAR 包。
- 开发者只需要编写一套 JDBC 风格的代码，就可以操作 MySQL 等不同的关系型数据库。

> [!TIP]
> JDBC 屏蔽了不同数据库在底层实现上的差异，降低了 Java 程序切换数据库的成本。

---

## 二、JDBC 快速入门

### 1. 项目准备

项目名示例：`javaweb-code`

- 项目名建议全部使用小写字母。
- 多个单词之间使用中横线 `-` 分隔。
- 检查项目使用的 JDK 版本。
- 检查 IDEA 的常规配置。
- 导入对应数据库的 JDBC 驱动包。

### 2. JDBC 基本开发流程

1. 注册数据库驱动。
2. 获取数据库连接。
3. 定义 SQL 语句。
4. 获取执行 SQL 的 `Statement` 或 `PreparedStatement` 对象。
5. 执行 SQL。
6. 处理返回结果。
7. 释放数据库资源。

执行 DML 语句后返回的 `count` 表示**受影响的行数**，通常可以通过 `count > 0` 判断操作是否成功。

### 3. 常见注意事项

- MySQL 8 推荐使用驱动类名：

  ```java
  com.mysql.cj.jdbc.Driver
  ```

- 如果 IDEA 中 SQL 语句显示为红色，可能是尚未配置数据库或 IDEA 没有正确识别 SQL 方言，可在数据库相关设置中选择对应的数据源或方言。

---

## 三、JDBC 核心 API

### 1. `DriverManager`：驱动管理与连接创建

#### 注册驱动

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

该语句会加载 `Driver` 类，类加载时会执行静态代码块，从而完成驱动注册。也可以通过 `registerDriver(驱动对象)` 注册驱动。

> [!NOTE]
> MySQL 8 的驱动通常支持自动加载，因此手动注册驱动可以省略；显式编写有助于理解 JDBC 的执行过程。

#### 获取数据库连接

```java
Connection conn = DriverManager.getConnection(url, username, password);
```

连接 URL 的基本格式：

```text
jdbc:mysql://IP或域名:端口号/数据库名称?k1=v1&k2=v2
```

示例：

```text
jdbc:mysql://localhost:3306/test
```

---

### 2. `Connection`：数据库连接

`Connection` 表示 Java 程序与数据库之间的一次连接，主要用于创建 SQL 执行对象和控制事务。

#### 获取 SQL 执行对象

```java
Statement stmt = conn.createStatement();
PreparedStatement pstmt = conn.prepareStatement(sql);
```

#### 事务控制

| 操作 | 方法 | 说明 |
|---|---|---|
| 开启事务 | `setAutoCommit(false)` | 关闭自动提交 |
| 提交事务 | `commit()` | 保存本次事务中的修改 |
| 回滚事务 | `rollback()` | 撤销本次事务中的修改 |

典型结构：

```java
try {
    conn.setAutoCommit(false);

    // 执行多条数据库操作

    conn.commit();
} catch (Exception e) {
    conn.rollback();
    throw e;
}
```

---

### 3. `Statement`：执行普通 SQL

`Statement` 用于向数据库发送 SQL 语句。

| SQL 类型 | 方法 | 返回值 |
|---|---|---|
| DML：增、删、改 | `executeUpdate(sql)` | `int`，受影响的行数 |
| DQL：查询 | `executeQuery(sql)` | `ResultSet`，查询结果集 |

```java
int count = stmt.executeUpdate(sql);
ResultSet rs = stmt.executeQuery(sql);
```

---

### 4. `ResultSet`：处理查询结果

`ResultSet` 用于封装查询返回的结果集。获取结果集后，内部游标位于第一行数据之前，需要逐行移动并读取数据。

#### `next()`

```java
boolean hasNext = rs.next();
```

它会：

1. 将游标移动到下一行。
2. 判断当前行是否存在数据。

常用遍历方式：

```java
while (rs.next()) {
    int id = rs.getInt("id");
    String name = rs.getString("name");
}
```

#### `getXXX()`

```java
rs.getXXX(字段序号);
rs.getXXX("字段名称");
```

- `XXX` 表示目标 Java 数据类型，例如 `getInt()`、`getString()`。
- 字段序号从 `1` 开始。
- 推荐使用**字段名称**读取数据，可读性更强。

---

### 5. `PreparedStatement`：预编译 SQL

`PreparedStatement` 是 `Statement` 的子接口，主要用于：

1. 执行 SQL 语句。
2. 预编译带参数的 SQL，提高重复执行效率。
3. 通过参数绑定避免用户输入被当作 SQL 结构执行，从而防止 SQL 注入。

SQL 注入是指攻击者通过构造特殊输入，改变原有 SQL 语义并攻击数据库。

#### 使用占位符

在 SQL 中使用 `?` 作为参数占位符，再通过 `setXXX()` 赋值：

```java
String sql = "SELECT * FROM user WHERE username = ? AND password = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, username);
pstmt.setString(2, password);
ResultSet rs = pstmt.executeQuery();
```

注意：

- `setXXX()` 的第一个参数是占位符的位置。
- 占位符位置从 `1` 开始。
- 执行时不再传入 SQL：

  ```java
  pstmt.executeUpdate(); // 执行 DML
  pstmt.executeQuery();  // 执行 DQL
  ```

> [!IMPORTANT]
> 实际开发中优先使用 `PreparedStatement`，不要通过字符串拼接把用户输入直接放入 SQL。

---

## 四、数据库连接池

### 1. 基本概念

数据库连接池采用**池化思想**，预先创建并管理一批数据库连接。当程序需要连接时从池中获取，使用完毕后归还，从而实现连接复用。

主要优点：

- 减少频繁创建和销毁连接的开销。
- 提高程序响应速度。
- 统一管理数据库连接。

### 2. 组成

- Java 提供标准的 `DataSource` 接口。
- 第三方厂商提供具体实现，例如 Druid。

### 3. 集成 Druid 连接池

1. 导入 Druid 的 JAR 包。
2. 导入 `druid.properties` 配置文件。
3. 加载属性配置文件。
4. 使用 `DruidDataSourceFactory` 创建 `DataSource`。
5. 通过 `getConnection()` 获取连接。

核心代码：

```java
Properties properties = new Properties();
properties.load(
    JdbcUtils.class.getClassLoader()
        .getResourceAsStream("druid.properties")
);

DataSource dataSource =
    DruidDataSourceFactory.createDataSource(properties);

Connection conn = dataSource.getConnection();
```

---

## 五、综合实践：封装 `JdbcUtils`

为了减少重复代码，可以抽取 JDBC 工具类，统一管理连接池、连接获取和资源释放。

### 设计要点

- 将构造方法私有化，防止外部创建工具类对象。
- 使用静态代码块加载配置并创建连接池，确保只初始化一次。
- 提供静态 `getConnection()` 方法，从连接池获取连接。
- 提供重载的 `release()` 方法，按需关闭 `ResultSet`、`Statement` 和 `Connection`。
- 对异常使用 `try-catch-finally` 或 `try-with-resources` 妥善处理。

### 资源释放顺序

```text
ResultSet → Statement → Connection
```

使用连接池时，调用 `Connection.close()` 通常不是销毁物理连接，而是把连接**归还连接池**。

现代 Java 更推荐使用 `try-with-resources` 自动释放资源：

```java
String sql = "SELECT * FROM user WHERE id = ?";

try (
    Connection conn = dataSource.getConnection();
    PreparedStatement pstmt = conn.prepareStatement(sql)
) {
    pstmt.setInt(1, 1);

    try (ResultSet rs = pstmt.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("name"));
        }
    }
}
```

---

## 六、知识点速查

| 对象 | 核心职责 | 常用方法 |
|---|---|---|
| `DriverManager` | 管理驱动、创建连接 | `getConnection()` |
| `Connection` | 表示数据库连接、控制事务 | `createStatement()`、`prepareStatement()`、`commit()`、`rollback()` |
| `Statement` | 执行普通 SQL | `executeUpdate(sql)`、`executeQuery(sql)` |
| `PreparedStatement` | 执行预编译 SQL、绑定参数 | `setXXX()`、`executeUpdate()`、`executeQuery()` |
| `ResultSet` | 封装并读取查询结果 | `next()`、`getXXX()` |
| `DataSource` | 管理数据库连接池 | `getConnection()` |

---

## 七、学习总结

JDBC 开发的核心流程可以概括为：

```text
加载驱动 → 获取连接 → 编写 SQL → 创建执行对象
→ 绑定参数 → 执行 SQL → 处理结果 → 释放资源
```

实际项目中的推荐实践：

1. 使用 `PreparedStatement` 执行 SQL。
2. 使用事务保证多步数据库操作的一致性。
3. 使用 Druid 等连接池复用数据库连接。
4. 封装工具类或数据访问层，减少重复代码。
5. 使用 `try-with-resources` 自动关闭资源。

> **一句话记忆：** JDBC 负责统一访问数据库，`PreparedStatement` 负责安全执行 SQL，事务负责保证一致性，连接池负责提升连接使用效率。
