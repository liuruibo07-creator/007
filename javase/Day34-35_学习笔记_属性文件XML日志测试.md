# 📘 Day 34-35 学习笔记：属性文件、XML、日志框架与单元测试

> 涵盖内容：属性文件（Properties）、XML 文件解析、日志框架（SLF4J + Logback）、单元测试（JUnit5）

---

## 目录

- [一、属性文件（Properties）](#一属性文件properties)
- [二、XML 文件](#二xml-文件)
- [三、日志框架](#三日志框架)
- [四、单元测试](#四单元测试)
- [五、知识速查表](#五知识速查表)

---

## 一、属性文件（Properties）

### 1.1 什么是属性文件

属性文件是 Java 中一种常见的**配置文件**格式，专门用来保存 `key=value` 结构的数据。

| 特点 | 说明 |
|---|---|
| 内容格式 | `key=value` |
| key 唯一性 | key **不能重复** |
| 文件后缀 | `.properties` |
| 主要作用 | 作为项目的**配置文件** |

### 1.2 核心操作类：`Properties`

```
Properties 本质上是什么？
└── 一个双列集合（Map）类
```

`Properties` 类实现了 `Map` 接口，是一个专门用来读写属性文件的双列集合类。

### 1.3 写数据

```java
Properties pro = new Properties();
pro.setProperty(String key, String value);  // 添加/修改属性
pro.store(输出流, "注释内容");                // 将数据写入属性文件
```

### 1.4 读数据

```java
Properties pro = new Properties();
pro.load(输入流);                             // 从属性文件加载数据
String value = pro.getProperty(String key);   // 根据 key 获取 value
```

> 💡 **小结**：Properties 的读写都需要借助 IO 流（`load` 需要输入流，`store` 需要输出流），这也是它和 IO 体系结合最紧密的地方。

---

## 二、XML 文件

### 2.1 什么是 XML

**XML（可扩展标记语言，eXtensible Markup Language）**

```
XML
├── 标记语言 → 通过“标签”来组织数据
└── 可扩展   → 标签名可以自定义
```

### 2.2 XML 的书写特点

| 序号 | 规则 |
|---|---|
| 1 | 必须在**第一行第一列**写文档声明 |
| 2 | **有且仅有一个**根标签 |
| 3 | 标签成对出现：有开始标签，也有结束标签 |
| 4 | 标签可以自定义**属性** |
| 5 | 文件后缀为 `.xml` |
| 6 | 特殊字符 `<`、`&` 需用转义符代替：`&lt;`、`&amp;` |
| 7 | 若特殊字符较多，可使用 **CDATA 区** 包裹 |
```java
<?xml version="1.0" encoding="UTF-8" ?>  
  
<!-- 规则1：必须在第一行第一列写文档声明 (如上) -->  
<!-- 规则2：有且仅有一个根标签 -->  
<students>  
  
    <!-- 规则3：标签成对出现：有开始标签，也有结束标签 -->  
    <student>  
        <!-- 规则4：标签可以自定义属性 -->  
        <name id="1">张三</name>  
        <age>20</age>  
        <gender>男</gender>  
        <info>成绩优秀 &amp; 表现积极</info>  
  
        <!-- 规则6：特殊字符 < 、& 需用转义符号代替：&lt;、&amp; -->  
        <score>数学：90分，排名 &lt; 前十名</score>  
  
        <!-- 规则7：若特殊字符较多，可使用 CDATA 区包裹 -->  
        <description><![CDATA[  
            学生评价：  
            成绩 >= 90 分为优秀  
            80 <= 成绩 < 90 为良好  
            特殊符号：< > & " ' 等都可以直接写  
        ]]></description>  
    </student>  
    <student>        
        <name id="2">李四</name>  
        <age>21</age>  
        <gender>女</gender>  
        <info>性格开朗 &amp; 团队合作能力强</info>  
        <score>英语：88分，排名 &lt; 前五名</score>  
        <description><![CDATA[  
            备注：  
            该学生参加了多个竞赛项目            获得 <一等奖> & <二等奖>  
        ]]></description>  
    </student>  
</students>
```
### 2.3 XML 的作用

主要用作项目的**配置文件**（与属性文件类似，但结构更复杂、层级更丰富）。

### 2.4 XML 的读取方式

使用 **DOM4J** 框架来解析读取 XML 数据。

### 2.5 DOM 解析思想

> 核心思路：将 XML 文件**一次性读取到内存中**，形成一棵**树形结构**，文档的每一部分都用一个对象来描述。

| XML 组成部分 | 对应的 Java 对象 |
|---|---|
| 整个文档 | `Document` |
| 元素 / 标签 | `Element` |
| 属性 | `Attribute` |
| 文本内容 | `Text` |

**DOM4J 解析步骤：**

```java
//获取解析器对象
SAXReader saxReader=new SAXReader();

// 1. 读取 xml 文件，返回 Document 对象
Document document = new SAXReader().read(xmlFile);

// 2. 通过 Document 对象获取根元素 Root Element
Element root = document.getRootElement();

// 3. 获取子标签
List<Element> list = 标签a.elements("标签名");   // 获取指定名称的所有子标签（List集合）
Element one = 标签a.element("标签名");            // 获取指定名称的单个子标签
Attribute attr = 标签a.attribute("属性名");        // 获取指定名称的属性对象

// 4. 获取文本内容
String text = 标签.getText();
```

### 2.6 约束文件

约束文件的作用：**限制 XML 文档的书写规范**（哪些标签可以出现、顺序、层级等）。

| 约束类型 | 特点 |
|---|---|
| **DTD** | 就是一个普通的文本文件，语法相对简单 |
| **Schema** | ① 支持数据类型校验；② 本身也是一个 XML 文件，也需要被其它 Schema 文件约束；③ 一个 XML 文件可以同时被多个 Schema 文件约束 |

---

## 三、日志框架

### 3.1 日志实现的组成

日志功能采用**接口与实现分离**的设计模式：

| 组成部分 | 说明 |
|---|---|
| **SLF4J** | 日志**接口**规范（Simple Logging Facade for Java） |
| **Logback** | SLF4J 接口的**具体实现类** |

### 3.2 集成日志的步骤

```
Step 1  导入 slf4j 和 logback 的 jar 包
Step 2  将 logback.xml 配置文件导入到 src 目录下
Step 3  在需要记录日志的类中定义日志记录器对象
Step 4  在需要记录日志的方法中，通过日志记录器对象记录日志
```

**Step 3 示例代码**（⚠️ 注意面向接口编程，全程使用的是 `slf4j` 中的 API）：

```java
private static final Logger log = LoggerFactory.getLogger(Test.class);
public static void main(String[] args) {  
    log.info("info: 记录info级别的日志");  
}
```

**Step 4 示例**：日志记录器对象提供了不同**日志级别**的方法，如 `log.debug(...)`、`log.info(...)` 等。

### 3.3 logback.xml 配置文件解析

| 标签 | 作用 |
|---|---|
| `<appender>` | 表示日志记录的**位置**。常用两种：① 记录到控制台；② 记录到文件中 |
| `<logger>` | **自定义**日志记录器，是 root 记录器的子类；可为某个类或某个包单独定义日志规则；通过 `level` 属性控制记录级别 |
| `<root>` | **根**日志记录器；同样通过 `level` 属性控制只记录 ≥ 该级别的日志 |

### 3.4 日志级别

日志级别由低到高排列：

```
debug  <  info  <  warn  <  error
（调试）   （信息）  （警告） （错误）
```

> 📌 级别设置为某一档时，**只会记录该级别及以上**的日志内容。例如设置为 `warn`，则 `warn` 和 `error` 会被记录，`debug`、`info` 会被忽略。

---

## 四、单元测试

### 4.1 什么是单元测试

**方法级别**的测试，通常由**开发人员**自己完成，也就是俗称的"**自测**"。

### 4.2 单元测试框架

主流框架：**JUnit**，目前通常使用 **JUnit 5** 版本。

### 4.3 集成单元测试的步骤

```
Step 1  导入 junit5 相关 jar 包
Step 2  定义单元测试类和单元测试方法
Step 3  运行单元测试方法，查看结果
```

**Step 2 详解：**

| 项目 | 建议 / 要求 |
|---|---|
| 快捷创建 | 选中被测试的类名 → 按 `Ctrl + Shift + T` |
| 类名建议 | `被测试的类名 + Test` 结尾 |
| 方法名建议 | `test + 被测试的方法名` |
| 权限要求 | 必须是 `public` 或**缺省**（不能是 private） |
| 返回值要求 | **无返回值**，写 `void` |
| 参数要求 | **无形参** |
| 有返回值判断 | 建议使用 **Assert 断言**进行结果判断 |

**Step 3 详解：**

- ✅ **绿色** → 测试**通过**
- ❌ **红色** → 测试**未通过**
- 测试粒度灵活：可以只测**一个方法**，也可以测**一个类**，甚至测**整个模块**

### 4.4 常用注解

| 注解 | 执行时机 | 执行次数 | 修饰对象 |
|---|---|---|---|
| `@BeforeEach` | 每个测试方法**之前** | 多次（每方法一次） | 实例方法 |
| `@AfterEach` | 每个测试方法**之后** | 多次（每方法一次） | 实例方法 |
| `@BeforeAll` | 所有测试方法**之前** | 仅 1 次 | **静态**方法 |
| `@AfterAll` | 所有测试方法**之后** | 仅 1 次 | **静态**方法 |

**执行顺序示意：**

```
@BeforeAll（一次）
   ↓
@BeforeEach → 测试方法1 → @AfterEach
   ↓
@BeforeEach → 测试方法2 → @AfterEach
   ↓
       ...
   ↓
@AfterAll（一次）
```

---

## 五、知识速查表

| 主题 | 核心类/框架 | 关键点一句话总结 |
|---|---|---|
| 属性文件 | `Properties` | `key=value` 格式，本质是双列集合，配合 IO 流读写 |
| XML | `DOM4J` | 树形结构解析，Document/Element/Attribute/Text 四大对象 |
| XML 约束 | DTD / Schema | DTD 是纯文本文件；Schema 支持数据类型且自身也是 XML |
| 日志 | SLF4J + Logback | 接口与实现分离；`appender` 定位置，`logger`/`root` 定级别 |
| 单元测试 | JUnit 5 | 类名 XxxTest，方法名 testXxx，public + void + 无参 |
| 测试注解 | `@BeforeEach`/`@AfterEach`/`@BeforeAll`/`@AfterAll` | Each 每方法执行一次，All 全局只执行一次（且需静态） |

---

## 🌳 整体知识树

```
Day34-35
├── 属性文件（Properties）
│   ├── 特点：key=value，key唯一，.properties后缀
│   ├── 本质：双列集合（Map）
│   ├── 写：setProperty + store
│   └── 读：load + getProperty
│
├── XML 文件
│   ├── 概念：可扩展标记语言
│   ├── 书写规则（7条）
│   ├── 解析：DOM4J（Document/Element/Attribute/Text）
│   └── 约束文件：DTD（纯文本） / Schema（支持类型，自身是XML）
│
├── 日志框架
│   ├── 组成：SLF4J（接口）+ Logback（实现）
│   ├── 集成4步骤
│   ├── 配置：appender（位置）/ logger（自定义级别）/ root（根级别）
│   └── 级别：debug < info < warn < error
│
└── 单元测试
    ├── 概念：方法级自测
    ├── 框架：JUnit5
    ├── 集成3步骤 + 编写规范
    └── 注解：@BeforeEach / @AfterEach / @BeforeAll / @AfterAll
```
