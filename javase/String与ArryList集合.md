# String与ArryList集合·（Day23）

> 本文档整理自 Day23 学习思维导图，涵盖 JavaBean 规范、成员变量与局部变量对比、API 与包的概念、文档注释、String 类详解、数组与集合的区别，以及 ArrayList 集合的完整用法。

---

## 目录

1. [JavaBean](#一javabean)
2. [成员变量 vs 局部变量](#二成员变量-vs-局部变量)
3. [API 与 API 文档](#三api-与-api-文档)
4. [包（Package）](#四包package)
5. [文档注释](#五文档注释)
6. [String 类](#六string-类)
7. [数组与集合的区别](#七数组与集合的区别)
8. [ArrayList 集合](#八arraylist-集合)

---

## 一、JavaBean

### 1.1 什么是 JavaBean

JavaBean 是一种**比较特殊的类**，又称为特殊规范的**实体类**，在企业开发中与数据库表**一一对应**，专门用来封装数据。

### 1.2 JavaBean 的三个强制要求

| 要求 | 说明 |
|------|------|
| **成员变量私有化** | 所有字段用 `private` 修饰 |
| **提供 getter / setter 方法** | 对外暴露数据的读写接口 |
| **必须有无参构造方法** | 供框架通过反射创建对象 |

### 1.3 实体类的组成限制

> 实体类中**只能有**：成员变量、getter/setter 方法、构造方法，不包含业务逻辑代码。

```java
public class Student {
    // 成员变量私有化
    private String name;
    private int age;

    // 无参构造（必须提供）
    public Student() {}

    // 全参构造（建议提供）
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // getter / setter
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}
```

---

## 二、成员变量 vs 局部变量

| 对比维度 | 成员变量 | 局部变量 |
|----------|----------|----------|
| **书写位置** | 类中、方法外 | 方法体中（含形参） |
| **内存位置** | **堆内存**（随对象存在） | **栈内存**（随方法调用存在） |
| **初始值** | 有**默认值**（`int→0`，`String→null`，`boolean→false`） | **没有默认值**，使用前必须赋值 |
| **作用域** | 整个类中都可以使用 | 通常仅限于所在方法体内 |
| **生命周期** | 与**对象**共存亡 | 与**方法**共存亡 |

```java
public class Demo {
    int num = 10;          // 成员变量：在堆内存，有默认值，整个类可用

    public void method() {
        int local = 20;    // 局部变量：在栈内存，无默认值，仅方法内可用
    }
}
```

---

## 三、API 与 API 文档

| 概念 | 说明 |
|------|------|
| **API**（Application Programming Interface） | 别人写好、可以直接使用的代码（类、方法等） |
| **API 文档** | 对 API 的使用说明书，描述类、方法及其用法 |

> 💡 Java 官方提供了大量 API（如 `String`、`Math`、`Scanner`），直接查文档调用即可，无需自己实现。

---

## 四、包（Package）

### 4.1 定义包

```java
// 通过 package 关键字定义包（必须写在文件第一行）
package com.edu.javabean;
//       ↑一级  ↑二级  ↑三级
```

**命名规范：**

- 多个层级之间用 **`.`** 分隔
- 包名是标识符，建议**全部小写**
- 通常以公司域名倒写开头（如 `com.itheima.project`）

---

### 4.2 导入包

| 情况 | 是否需要导包 |
|------|-------------|
| **同一个包**下的类 | ❌ 不需要 |
| **`java.lang`** 包下的类（如 `String`、`System`） | ❌ 不需要（自动导入） |
| 其他所有情况 | ✅ 需要 `import` |

```java
import java.util.Scanner;    // 导入 Scanner 类
import java.util.ArrayList;  // 导入 ArrayList 类
```

**特殊情况：不同包下有同名类**

```java
// 采用全类名（包名 + 类名）来区分
java.util.Date d1 = new java.util.Date();
java.sql.Date  d2 = new java.sql.Date(0);
```

---

## 五、文档注释

**文档注释**（`/** ... */`）是一种特殊注释，具有以下特点：

| 特点 | 说明 |
|------|------|
| 可以写多行 | 支持多行内容 |
| 在 API 文档中显示 | 通过 `javadoc` 命令生成 HTML 格式的 API 文档 |
| 适用位置 | 给**类**和**方法**加注释 |
| 生成命令 | `javadoc -d 输出目录 文件名.java` |

```java
/**
 * 学生实体类，封装学生的基本信息
 *
 * @author 作者名
 * @version 1.0
 */
public class Student {

    /**
     * 根据姓名和年龄创建学生对象
     *
     * @param name 学生姓名
     * @param age  学生年龄
     */
    public Student(String name, int age) { ... }
}
```

---

## 六、String 类

### 6.1 创建 String 对象的两种方式

#### 方式一：直接赋值（推荐 ✅）

```java
String str = "abc";
// 双引号引起来的内容都是 String 类的实例（对象）
```

#### 方式二：使用构造方法

```java
new String()              // 空字符串
new String("abc")         // 内容为 "abc"
new String(char[] chars)  // 将字符数组拼接成字符串
new String(byte[] bytes)  // 将字节数组（编码值）转换并拼接成字符串
```

---

### 6.2 String 常用方法速查

#### 基础操作

| 方法 | 说明 | 示例 |
|------|------|------|
| `length()` | 获取字符串长度 | `"abc".length()` → `3` |
| `charAt(index)` | 获取指定索引处的字符 | `"abc".charAt(1)` → `'b'` |
| `toCharArray()` | 转换为字符数组 | `"abc".toCharArray()` → `['a','b','c']` |

#### 字符串遍历

```java
String str = "hello";

// 方式一：length() + charAt()
for (int i = 0; i < str.length(); i++) {
    System.out.println(str.charAt(i));
}

// 方式二：toCharArray() 转数组后遍历
for (char c : str.toCharArray()) {
    System.out.println(c);
}
```

#### 比较操作

| 方法 | 说明 | 注意 |
|------|------|------|
| `==` | 引用类型比较的是**地址值** | ❌ 不能用于字符串内容比较 |
| `equals(str)` | 内容比较，**区分大小写** | ✅ 推荐 |
| `equalsIgnoreCase(str)` | 内容比较，**不区分大小写** | 验证码比较常用 |

```java
String s1 = new String("abc");
String s2 = new String("abc");

System.out.println(s1 == s2);              // → false（地址不同）
System.out.println(s1.equals(s2));         // → true（内容相同）
System.out.println("ABC".equalsIgnoreCase("abc")); // → true
```

#### 截取操作

| 方法 | 说明 |
|------|------|
| `substring(beginIndex)` | 从 `beginIndex` 截取到末尾 |
| `substring(beginIndex, endIndex)` | 从 `beginIndex` 到 `endIndex`（**不含** `endIndex`） |

```java
"HelloWorld".substring(5)      // → "World"
"HelloWorld".substring(0, 5)   // → "Hello"
```

#### 查找操作

| 方法 | 说明 |
|------|------|
| `contains(子串)` | 是否包含指定子串，返回 `boolean` |
| `startsWith(前缀)` | 是否以指定字符串开头 |
| `endsWith(后缀)` | 是否以指定字符串结尾 |
| `indexOf(str)` | 第一次出现的索引位置，找不到返回 `-1` |
| `lastIndexOf(str)` | 最后一次出现的索引位置 |

#### 修改操作

| 方法 | 说明 |
|------|------|
| `replace(旧值, 新值)` | 替换**全部**匹配的旧值 |
| `replaceFirst(旧值, 新值)` | 只替换**第一个**匹配的旧值 |
| `toUpperCase()` | 转换为大写 |
| `toLowerCase()` | 转换为小写 |
| `trim()` | 去掉首尾空格 |
| `split(分隔符)` | 按分隔符切割，返回 `String[]` |

```java
"hello world".replace("l", "L")    // → "heLLo worLd"
"hello".toUpperCase()               // → "HELLO"
"  hi  ".trim()                     // → "hi"
"a,b,c".split(",")                  // → ["a", "b", "c"]
```

---

### 6.3 String 注意事项

> ⚠️ 这两点非常重要，容易出错！

**注意一：字符串常量池**

```
双引号包裹的字符串对象存放在字符串常量池中
相同内容的字符串只存储一份（池化思想，节省内存）
从 JDK 7 开始，常量池从方法区移到了堆内存中
```

```java
String s1 = "abc";
String s2 = "abc";
System.out.println(s1 == s2);  // → true（指向常量池同一个对象）

String s3 = new String("abc");
System.out.println(s1 == s3);  // → false（s3 在堆中是新对象）
```

**注意二：字符串不可变**

```
String 对象一旦创建，其内容不可改变
对字符串内容"修改"时，实际上是产生了一个新对象
使用 replace / toUpperCase 等方法后，建议用变量接收返回值
```

```java
String str = "hello";
str.toUpperCase();               // ❌ str 本身没有变化
str = str.toUpperCase();         // ✅ 用变量接收修改结果
System.out.println(str);        // → "HELLO"
```

---

## 七、数组与集合的区别

### 相同点

- 都是**容器**
- 都可以存储**多个数据**

### 不同点

| 对比项 | 数组 | 集合（ArrayList） |
|--------|------|-------------------|
| **长度** | 定义后**固定不变** | **动态可变**，可随时增删 |
| **元素类型** | 既可存基本类型，也可存引用类型 | **只能存引用类型**（基本类型需用包装类） |
| **类型限制** | 只能存**同一种**数据类型 | 可存**任意**引用类型（加泛型后限定一种） |

---

## 八、ArrayList 集合

### 8.1 概述

`ArrayList` 是 Java 集合体系中**使用频率最高**的集合类，底层基于动态数组实现。

### 8.2 创建 ArrayList 对象

```java
// 不加泛型：可以存任意引用类型（不推荐）
ArrayList list = new ArrayList();

// 加泛型（推荐 ✅）：限制集合只能存同类型数据
ArrayList<String>  list = new ArrayList<>();
ArrayList<Integer> nums = new ArrayList<>();
ArrayList<Student> students = new ArrayList<>();
```

**泛型 `<E>` 说明：**

| 概念 | 说明 |
|------|------|
| 语法 | `<E>`，其中 `E` 是**类型参数** |
| 作用 | 限制集合只能装**同类型**的引用数据类型 |
| 好处 | 编译期类型检查，避免运行时类型转换错误 |

---

### 8.3 ArrayList 常用方法

#### 增（Add）

```java
list.add(元素)             // 向集合末尾添加元素
list.add(index, 元素)      // 在指定索引位置插入元素（后面的元素后移）
```

#### 删（Remove）

```java
list.remove(index)         // 删除指定索引位置上的元素
list.remove(元素)          // 删除指定内容的元素（第一个匹配项）
```

#### 改（Set）

```java
list.set(index, 新元素)    // 将指定索引位置的元素替换为新元素
```

#### 查（Get / Size）

```java
list.get(index)            // 返回指定索引位置上的元素
list.size()                // 返回集合中元素的个数（长度）
```

---

### 8.4 遍历 ArrayList

```java
ArrayList<String> list = new ArrayList<>();
list.add("Java");
list.add("Python");
list.add("Go");

// 使用 fori 循环（快捷键：集合名.fori）
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}
```

---

### 8.5 方法总结速查表

| 方法 | 功能 | 返回值 |
|------|------|--------|
| `add(元素)` | 末尾添加 | `boolean` |
| `add(index, 元素)` | 指定位置插入 | `void` |
| `remove(index)` | 按索引删除 | 被删除的元素 |
| `remove(元素)` | 按内容删除 | `boolean` |
| `set(index, 新元素)` | 按索引修改 | 被替换的旧元素 |
| `get(index)` | 按索引查询 | 对应位置的元素 |
| `size()` | 查询长度 | `int` |

---

*整理完毕 · Java 基础 Day23*
