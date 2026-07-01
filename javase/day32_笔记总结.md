# 📚 Day 32 — Java 学习笔记总结

> 本文涵盖：Stream 流 · File 类 · 方法递归 · 字符集 · IO 流体系 · 字节流（FileInputStream）

---

## 一、Stream 流

### 1.1 概述

JDK 8 新特性，主要用来**简化集合的操作**。

### 1.2 使用步骤

```
（1）获取流
（2）调用中间方法处理数据（可链式调用多次）
（3）调用终结方法处理数据（得到最终结果）
```

---

### 1.3 如何获取流

| 数据来源 | 获取方式 |
|---|---|
| 单列集合 | `集合.stream()` |
| 双列集合 | 先转换成单列集合，再获取流 |
| 数组 | `Arrays.stream(数组)` |
| 零散数据 | `Stream.of(可变参数)` |

```java
// 单列集合
List<String> list = new ArrayList<>();
Stream<String> s1 = list.stream();

// 双列集合（先转单列）
Map<String, Integer> map = new HashMap<>();
Stream<String> s2 = map.keySet().stream();

//4.如何获取数组的Stream流  
String[] arr = {"张三", "李四", "王五", "赵六"};  
Stream<String> s1 = Arrays.stream(arr);

// 零散数据
Stream<Integer> s4 = Stream.of(1, 2, 3, 4);
```

---

### 1.4 中间方法

> **特点：** 调用后返回的**依然是流**，可以继续链式调用其他中间方法。

| 方法 | 说明 |
|---|---|
| `filter(Predicate)` | 过滤，接收**条件型**函数式接口 `Predicate`，条件成立则保留数据，否则不保留 |
| `sorted()` | 自然排序 |
| `sorted(Comparator)` | 比较器排序 |
| `limit(n)` | 获取前面 **n** 个元素 |
| `skip(n)` | 跳过前面 **n** 个元素 |
| `distinct()` | 去重 |
| `map(Function)` | 映射转换，接收**函数型**函数式接口 `Function`，对元素进行转换得到新数据 |
| `Stream.concat(流1, 流2)` | 将两个流合并为一个流 |

```java
list.stream()
    .filter(s -> s.length() > 2)     // 过滤
    .distinct()                      // 去重
    .sorted()                        // 排序
    .limit(3)                        // 取前3个
    .skip(1)                         // 跳过第1个
    .map(s -> s.toUpperCase())       // 转换
    .forEach(System.out::println);   // 终结方法遍历

// 合并流
Stream<String> merged = Stream.concat(stream1, stream2);
```

---

### 1.5 终结方法

> **特点：** 调用后返回的**不再是流**，流将被关闭，不能再继续处理。

| 方法 | 说明 |
|---|---|
| `forEach(Consumer)` | 遍历，接收**消费型**函数式接口 `Consumer` |
| `count()` | 返回流中数据的个数 |
| `collect(Collector)` | 收集流中的元素，转换为集合 |

**`collect` 常用收集方式：**

| 收集器 | 说明 |
|---|---|
| `Collectors.toList()` | 收集为 `List` 集合 |
| `Collectors.toSet()` | 收集为 `Set` 集合 |
| `Collectors.toMap(key转换规则, value转换规则)` | 收集为 `Map` 集合 |

```java
long count = list.stream().count();

List<String> resultList = list.stream()
    .filter(s -> s.length() > 2)
    .collect(Collectors.toList());

Map<String, Integer> resultMap = list.stream()
    .collect(Collectors.toMap(s -> s, s -> s.length()));
```

---

## 二、File 类

### 2.1 概述

`File` 类可以用来表示**文件或者文件夹**，本质上就是**文件或文件夹的路径**。

---

### 2.2 构造方法

| 构造方法 | 说明 |
|---|---|
| `new File(String pathname)` | 根据文件/文件夹路径创建 File 对象 |
| `new File(String parent, String child)` | 根据父路径 + 子路径字符串创建 File 对象 |
| `new File(File parent, String child)` | 根据父路径 File 对象 + 子路径字符串创建 File 对象 |

```java
File f1 = new File("D:\\test\\a.txt");
File f2 = new File("D:\\test", "a.txt");
File f3 = new File(new File("D:\\test"), "a.txt");
```

---

### 2.3 路径

| 路径类型 | 说明 |
|---|---|
| **绝对路径** | 完整的路径，如 `D:\test\a.txt` |
| **相对路径** | 简化的路径写法；在 IO 中，相对路径是**相对于项目名称**而言的 |

---

### 2.4 判断方法

| 方法 | 说明 |
|---|---|
| `exists()` | 判断文件/文件夹是否存在 |
| `isFile()` | 判断是否为**文件** |
| `isDirectory()` | 判断是否为**文件夹** |

---

### 2.5 获取信息方法

| 方法 | 说明 |
|---|---|
| `getName()` | 获取文件/文件夹的名称 |
| `length()` | 获取文件的大小（字节数），**只能用于获取文件大小**，不能获取文件夹大小 |
| `getPath()` | 获取创建 File 对象时使用的路径 |
| `getAbsolutePath()` | 获取绝对路径 |
| `lastModified()` | 获取最后修改时间（毫秒值） |

⚠️ **注意：** `length()` 方法只能用于获取**文件**的大小，不能用于文件夹。

---

### 2.6 创建方法

| 方法 | 说明 |
|---|---|
| `createNewFile()` | 创建一个新文件 |
| `mkdir()` | 创建单级文件夹 |
| `mkdirs()` | 创建多级文件夹（**推荐使用**） |

> 💡 创建文件夹时**推荐使用 `mkdirs()`**，因为它既能创建单级目录，也能创建多级目录。

---

### 2.7 删除方法

| 方法 | 说明 |
|---|---|
| `delete()` | 删除文件或文件夹 |

⚠️ **注意事项：** `delete()` 只能删除**文件**和**空文件夹**，不能直接删除非空文件夹。

---

### 2.8 遍历文件夹方法

| 方法 | 说明 |
|---|---|
| `listFiles()` | 遍历文件夹，返回该目录下所有文件/文件夹对应的 `File[]` 数组 |

**⚠️ 注意事项：**

| 情况 | 返回结果 |
|---|---|
| 路径不存在，或路径指向的是一个**文件** | 返回 **`null`** |
| 路径指向一个**空文件夹** | 返回长度为 **0** 的空数组 |
| 文件夹中含有隐藏文件/文件夹 | 该方法**同样能返回**隐藏的文件和文件夹 |

---

## 三、方法递归

### 3.1 概述

**方法调用自身**的编程技巧。

### 3.2 如何分析递归

| 分析步骤 | 说明 |
|---|---|
| ① 拆解规律 | 找到将**大问题拆解成小问题**的规律 |
| ② 明确出口 | 确定递归**什么时候停下来**（递归出口），避免出现死递归 / 栈溢出 |

```java
// 示例：递归求阶乘
public static int factorial(int n) {
    if (n == 1) {           // 递归出口
        return 1;
    }
    return n * factorial(n - 1);  // 大问题拆解为小问题
}
```

---

## 四、字符集

### 4.1 常见字符集

| 字符集 | 特点 |
|---|---|
| **ASCII** | 英文占 **1 个字节**，且最高位是 **0** |
| **GBK** | 英文占 1 个字节，**汉字占 2 个字节**，汉字第一个字节的最高位是 **1** |
| **UTF-8** | 英文占 1 个字节，**汉字占 3 个字节**，汉字第一个字节的最高位是 **1** |

> ⚠️ **注意：** 所有字符集都**兼容 ASCII 字符集**。

---

### 4.2 编解码

```
编码：字符 → 二进制    （String 类的 getBytes() 方法）
解码：二进制 → 字符    （String 类的构造方法）
```

| 操作 | 实现方式 |
|---|---|
| **编码** | 将字符变成二进制，通过 `String` 类的 `getBytes()` 方法实现 |
| **解码** | 将二进制变成字符，通过 `String` 类的**构造方法**实现 |

```java
// 编码
String str = "你好";
byte[] bytes = str.getBytes("UTF-8");

// 解码
String result = new String(bytes, "UTF-8");
```

**⚠️ 注意事项：** 中文编解码要使用**相同的字符集**，否则会出现中文乱码。

---

## 五、IO 流体系

```
IO流
├── 字节输入流  InputStream
├── 字节输出流  OutputStream
├── 字符输入流  Reader
└── 字符输出流  Writer
```

| 分类 | 顶层父类 |
|---|---|
| 字节输入流 | `InputStream` |
| 字节输出流 | `OutputStream` |
| 字符输入流 | `Reader` |
| 字符输出流 | `Writer` |

---

## 六、字节流 — FileInputStream（文件字节输入流）

### 6.1 构造方法

```java
new FileInputStream(String path);
```

⚠️ **注意：** 要求文件必须**存在**，否则抛出 `FileNotFoundException`。

---

### 6.2 读数据方法

| 方法 | 说明 |
|---|---|
| `read()` | 一次读取**一个字节**，读到文件末尾时返回 **-1** |

```java
FileInputStream fis = new FileInputStream("D:\\test\\a.txt");
int b;
while ((b = fis.read()) != -1) {
    System.out.println(b);
}
fis.close();
```

---

### 6.3 关流方法

| 方法 | 说明 |
|---|---|
| `close()` | 关闭流，释放资源 |

---

## 📝 本日知识点总览

```
day32
├── Stream 流（JDK8简化集合操作）
│   ├── 使用步骤：获取流 → 中间方法 → 终结方法
│   ├── 获取流：stream() / Arrays.stream() / Stream.of()
│   ├── 中间方法：filter / sorted / limit / skip / distinct / map / concat
│   └── 终结方法：forEach / count / collect（toList/toSet/toMap）
│
├── File 类
│   ├── 构造方法：三种重载方式
│   ├── 路径：绝对路径 / 相对路径（相对于项目）
│   ├── 判断方法：exists / isFile / isDirectory
│   ├── 获取信息：getName / length（仅限文件）/ getPath 等
│   ├── 创建方法：createNewFile / mkdir / mkdirs（推荐）
│   ├── 删除方法：delete（只能删文件和空文件夹）
│   └── 遍历方法：listFiles（不存在返回null，空文件夹返回空数组，含隐藏文件）
│
├── 方法递归
│   ├── 概念：方法调用自身
│   └── 分析：拆解规律 + 明确递归出口
│
├── 字符集
│   ├── ASCII：英文1字节，最高位0
│   ├── GBK：英文1字节，汉字2字节
│   ├── UTF-8：英文1字节，汉字3字节
│   └── 编解码：getBytes()编码 / String构造方法解码，须用相同字符集
│
├── IO 流体系
│   └── InputStream / OutputStream / Reader / Writer
│
└── 字节流 - FileInputStream
    ├── 构造方法：文件须存在，否则 FileNotFoundException
    ├── read()：一次读一个字节，末尾返回-1
    └── close()：关闭释放资源
```
