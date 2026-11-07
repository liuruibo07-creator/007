# Day 39｜数据库与 SQL 基础总结

> 本章重点：理解数据库相关概念，掌握 SQL 的分类，并熟悉 MySQL 中 DDL、DML 的常用语法。

---

## 二、SQL 语言分类

| 分类  | 名称     | 主要用途                      |
| --- | ------ | ------------------------- |
| DDL | 数据定义语言 | 操作数据库对象，如数据库、数据表、视图、存储过程等 |
| DML | 数据操作语言 | 对表中的数据进行新增、修改和删除          |
| DQL | 数据查询语言 | 查询表中的数据                   |
| DCL | 数据控制语言 | 用户及权限管理                   |

---

## 三、DDL：数据库与表结构操作

### 1. 数据库操作

```sql
-- 创建数据库：不存在时才创建
CREATE DATABASE IF NOT EXISTS db1;

-- 查看所有数据库
SHOW DATABASES;

-- 切换到指定数据库
USE db1;

-- 删除数据库：存在时才删除
DROP DATABASE IF EXISTS db1;
```

### 2. 创建数据表

设计表字段时，可以将字段分为三类：

- **基础字段**：编号、创建时间、更新时间等通用信息。
- **业务字段**：用户名、姓名、性别、职位等业务数据。
- **关系字段**：用于建立表与表之间关系的字段，如外键。

## 一、数据库基础概念

### 1. 数据库（DB）

数据库（Database，简称 **DB**）是用于存储和组织数据的“仓库”。

### 2. 数据库管理系统（DBMS）

数据库管理系统（Database Management System，简称 **DBMS**）是用于管理数据库的软件。

### 3. SQL

SQL（Structured Query Language）是操作关系型数据库的标准语言。

### 4. 关系型数据库

关系型数据库以**二维表**的形式存储数据，表与表之间可以建立关联关系。

---

#### 员工表示例

```sql
CREATE TABLE tb_emp
(
    id          INT PRIMARY KEY AUTO_INCREMENT COMMENT '编号',
    username    VARCHAR(20) UNIQUE NOT NULL COMMENT '用户名',
    name        VARCHAR(10) NOT NULL COMMENT '姓名',
    gender      TINYINT UNSIGNED NOT NULL COMMENT '性别：1表示男，2表示女',
    image       VARCHAR(255) COMMENT '头像',
    job         TINYINT UNSIGNED NOT NULL COMMENT '职位：1班主任，2讲师，3学工主管，4教研主管',
    entry_date  DATE COMMENT '入职时间',
    password    VARCHAR(32) NOT NULL DEFAULT '123456' COMMENT '密码',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP
                         ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
) COMMENT '员工表';
```

### 3. 常用数据类型

#### 数值类型

| 类别 | 常用类型 | 说明 |
| --- | --- | --- |
| 整数 | `TINYINT`、`SMALLINT`、`INT`、`BIGINT` | 默认有符号；添加 `UNSIGNED` 可设为无符号 |
| 小数 | `DOUBLE`、`DECIMAL` | `DOUBLE(M,D)` 中，`M` 表示总位数，`D` 表示小数位数 |

数值可以直接书写，不需要添加引号。

#### 字符串类型

| 类型 | 特点 | 示例 |
| --- | --- | --- |
| `VARCHAR` | 变长字符串 | `VARCHAR(10)` |
| `CHAR` | 定长字符串 | `CHAR(10)` |

字符串值通常需要使用单引号包裹，例如 `'张三'`。

#### 日期与时间类型

- `DATE`：日期。
- `TIME`：时间。
- `DATETIME`：日期和时间。

日期、时间值通常也需要使用单引号包裹，例如 `'2026-07-20'`。

### 4. 常用约束

| 约束 | SQL 关键字 | 作用 |
| --- | --- | --- |
| 非空约束 | `NOT NULL` | 字段值不能为空 |
| 唯一约束 | `UNIQUE` | 字段值不能重复 |
| 主键约束 | `PRIMARY KEY` | 唯一标识表中的一条记录 |
| 默认约束 | `DEFAULT` | 未赋值时使用默认值 |
| 外键约束 | `FOREIGN KEY` | 建立表与表之间的关联 |

补充：`AUTO_INCREMENT` 可使整数主键自动递增。

### 5. 查询数据表

```sql
-- 查看当前数据库中的所有表
SHOW TABLES;

-- 查看表结构
DESC tb_emp;

-- 查看建表语句
SHOW CREATE TABLE tb_emp;
```

在部分数据库客户端中，选中表后可使用 `Ctrl + B` 查看表结构。

### 6. 修改数据表

```sql
-- 添加字段
ALTER TABLE tb_emp
ADD qq VARCHAR(11) COMMENT 'QQ';

-- 修改字段类型
ALTER TABLE tb_emp
MODIFY qq VARCHAR(15);

-- 修改字段名称和类型
ALTER TABLE tb_emp
CHANGE qq qqq VARCHAR(11);

-- 删除字段
ALTER TABLE tb_emp
DROP COLUMN qqq;
```

### 7. 删除数据表

```sql
DROP TABLE IF EXISTS tb_emp;
```

> [!WARNING]
> `DROP` 会删除数据库或表结构及其中的数据，执行前应确认对象名称并做好备份。

---

## 四、DML：表数据的增、删、改

### 1. 新增数据（INSERT）

#### 插入单行

```sql
INSERT INTO 表名 (字段1, 字段2, ...)
VALUES (值1, 值2, ...);
```

#### 批量插入

```sql
INSERT INTO 表名 (字段1, 字段2, ...)
VALUES
    (值1, 值2, ...),
    (值1, 值2, ...);
```

> 建议显式写出字段列表，确保字段和值一一对应，也可以避免表结构变化带来的问题。

### 2. 删除数据（DELETE）

```sql
DELETE FROM 表名
WHERE 条件;
```

### 3. 修改数据（UPDATE）

```sql
UPDATE 表名
SET 字段1 = 值1,
    字段2 = 值2
WHERE 条件;
```

> [!CAUTION]
> 执行 `UPDATE` 或 `DELETE` 时如果省略 `WHERE`，将影响表中的全部数据。

---

## 五、核心知识速查

| 目标 | 关键语法 |
| --- | --- |
| 创建数据库 | `CREATE DATABASE` |
| 查看数据库 | `SHOW DATABASES` |
| 使用数据库 | `USE` |
| 删除数据库 | `DROP DATABASE` |
| 创建表 | `CREATE TABLE` |
| 查看表 | `SHOW TABLES` |
| 查看表结构 | `DESC` |
| 修改表结构 | `ALTER TABLE` |
| 删除表 | `DROP TABLE` |
| 新增数据 | `INSERT INTO` |
| 修改数据 | `UPDATE ... SET ... WHERE ...` |
| 删除数据 | `DELETE FROM ... WHERE ...` |

---

## 六、学习要点

1. 分清 **DB、DBMS、SQL** 三个概念。
2. 熟记 SQL 四大分类：**DDL、DML、DQL、DCL**。
3. 能独立完成数据库和数据表的创建、查询、修改与删除。
4. 掌握常用数据类型和约束，并能根据业务选择合适的字段定义。
5. 熟练使用 `INSERT`、`UPDATE`、`DELETE`，尤其注意字段对应关系和 `WHERE` 条件。

