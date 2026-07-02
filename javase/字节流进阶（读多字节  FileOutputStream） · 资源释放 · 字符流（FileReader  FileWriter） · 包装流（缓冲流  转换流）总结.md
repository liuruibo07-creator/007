# 📚 Day 33 — Java 学习笔记总结

> 本文涵盖：字节流进阶（读多字节 / FileOutputStream） · 资源释放 · 字符流（FileReader / FileWriter） · 包装流（缓冲流 / 转换流）

---

## 一、字节流

### 1.1 FileInputStream — 一次读取多个字节

| 方法 | 说明 |
|---|---|
| `read(字节数组)` | 一次读取多个字节存入数组，返回**实际读取的字节个数**，读不到数据时返回 **-1** |

```java
FileInputStream fis = new FileInputStream("D:\\test\\a.txt");
byte[] buffer = new byte[1024];
int len;
while ((len = fis.read(buffer)) != -1) {
    // 将字节数组转换为字符串（只转换有效读取的部分）
    String s = new String(buffer, 0, len);
    System.out.println(s);
}
fis.close();
```

**配套方法：**

```java
// String 构造方法：将字节数组的一部分转换为字符串
new String(字节数组, offset, length);
```

---

### 1.2 FileOutputStream — 文件字节输出流

#### 构造方法

| 构造方法 | 说明 |
|---|---|
| `new FileOutputStream(path)` | 文件不存在则**创建**；文件存在则**清空**后写入 |
| `new FileOutputStream(path, true)` | 文件存在时**不清空**，在原内容后面进行**追加**写入 |

⚠️ **注意事项：** `FileOutputStream` **不会自动创建**文件所在的目录，目录必须提前存在。

#### 写方法

| 方法 | 说明 |
|---|---|
| `write(int)` | 写一个字节 |
| `write(字节数组)` | 写一个完整的字节数组 |
| `write(字节数组, offset, length)` | 写字节数组的一部分 |

#### 写换行符

```java
// 使用系统自带的换行符（跨平台）
fos.write(System.lineSeparator().getBytes());
```

```java
FileOutputStream fos = new FileOutputStream("D:\\test\\a.txt", true);
fos.write("hello".getBytes());
fos.write(System.lineSeparator().getBytes());  // 写换行符
fos.write("world".getBytes());
fos.close();
```

---

## 二、资源释放

### 2.1 try-catch-finally（传统方式）

```java
FileInputStream fis = null;
try {
    fis = new FileInputStream("a.txt");
    // 读取操作
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (fis != null) {
        try {
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### 2.2 try-with-resource（JDK 7+ 推荐方式）

#### 格式

```java
try (资源定义; 资源定义) {
    // 使用资源
} catch (异常) {
    // 处理异常
}
```

#### 特点

> 资源使用完毕后，会**自动调用 `close()`** 进行关闭，无需手动在 `finally` 中释放。

#### 什么是"资源"？

> 只有**实现了 `AutoCloseable` 接口**的类，才能被当作"资源"放入 `try()` 中。

```java
// 示例
try (FileInputStream fis = new FileInputStream("a.txt");
     FileOutputStream fos = new FileOutputStream("b.txt")) {
    // 使用 fis 和 fos
} catch (IOException e) {
    e.printStackTrace();
}
// 无需 finally，流会自动关闭
```

---

## 三、字符流

### 3.1 本质

> **字符流 = 字节流 + 编码表**

### 3.2 使用场景

> 当对**文本文件**进行单独的读或写操作时，**推荐使用字符流**（可以避免中文乱码问题）。

---

### 3.3 FileReader

| 方法 | 说明 |
|---|---|
| （构造方法） | `new FileReader(path)` |
| `read()` | 读取**一个字符**，返回字符对应编码值，读不到返回 -1 |
| `read(字符数组)` | 读取**一个字符数组**，返回实际读取的字符个数 |

```java
FileReader fr = new FileReader("a.txt");
int c;
while ((c = fr.read()) != -1) {
    System.out.print((char) c);
}
fr.close();
```

---

### 3.4 FileWriter

#### 写方法

| 方法 | 说明 |
|---|---|
| `write(int)` | 写一个字符 |
| `write(字符数组)` | 写一个字符数组 |
| `write(字符数组, offset, length)` | 写字符数组的一部分 |
| `write(String)` | 写一个字符串 |
| `write(String, offset, length)` | 写字符串的一部分 |

#### 注意事项

- 字符输出流**自带缓冲区**，写完数据后**一定要刷新或关闭**流，否则数据可能不会真正写入文件。

**`flush()` 与 `close()` 的区别：**

| 方法 | 说明 |
|---|---|
| `flush()` | 刷新缓冲区，调用后**还可以继续**使用该流写入数据 |
| `close()` | 关闭流，调用后**不能再使用**该流；`close()` 内部**会自动调用一次 `flush()`** |

```java
FileWriter fw = new FileWriter("a.txt");
fw.write("Hello Java");
fw.flush();          // 刷新，仍可继续写
fw.write(" World");
fw.close();           // 关闭，内部已自动flush
```

---

## 四、包装流

### 4.1 特点

> 对**原始流**进行包装，从而对功能进行**增强**；创建包装流对象时，需要**传入原始流对象**作为参数。

---

### 4.2 缓冲流

| 缓冲流 | 说明 |
|---|---|
| `BufferedInputStream` | 自带一个 **8192 长度**的字节数组作为缓冲区 |
| `BufferedOutputStream` | 自带一个 **8192 长度**的字节数组作为缓冲区 |
| `BufferedReader` | 自带一个 **8192 长度**的字符数组作为缓冲区；特有方法 `readLine()` |
| `BufferedWriter` | 自带一个 **8192 长度**的字符数组作为缓冲区；特有方法 `newLine()` |

#### BufferedReader 特有方法

```java
// 一次可以读取一行数据，读不到返回 null
String line = bufferedReader.readLine();
```

#### BufferedWriter 特有方法

```java
// 写一个跨平台的换行符
bufferedWriter.newLine();
```

```java
// 缓冲流使用示例
try (BufferedReader br = new BufferedReader(new FileReader("a.txt"));
     BufferedWriter bw = new BufferedWriter(new FileWriter("b.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        bw.write(line);
        bw.newLine();
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

---

### 4.3 转换流

#### InputStreamReader（字节流 → 字符流）

```java
// 构造方法：可自定义编码表进行读取
new InputStreamReader(inputStream, charset);
```

- 作用：把**字节流转换成字符流**。
- 可以**自定义编码表**来读取数据，解决默认编码与文件编码不一致导致的乱码问题。

#### OutputStreamWriter（字符流 → 字节流）

```java
// 构造方法：可自定义编码表进行写入
new OutputStreamWriter(outputStream, charset);
```

- 作用：把**字符流转换成字节流**。
- 可以**自定义编码表**来写入数据。

```java
// 示例：以 GBK 编码读取文件
InputStreamReader isr = new InputStreamReader(
    new FileInputStream("a.txt"), "GBK");
```

#### 补充说明

> 从 **JDK 11** 开始，`FileReader` 和 `FileWriter` 也**支持自定义编码表**（此前只能使用系统默认编码）。

---

### 4.4 打印流 & 序列化流

> 本节仅提及标题，具体内容将在后续章节展开。

- **打印流**（如 `PrintStream` / `PrintWriter`）
- **序列化流**（如 `ObjectInputStream` / `ObjectOutputStream`）

---

## 📝 本日知识点总览

```
day33
├── 字节流
│   ├── FileInputStream：read(字节数组) 返回读取个数，配合 new String(数组,offset,length)
│   └── FileOutputStream
│       ├── 构造：new(path) 清空 / new(path,true) 追加
│       ├── 注意：不会自动创建目录
│       └── 写方法：write(int) / write(数组) / write(数组,offset,length)
│
├── 资源释放
│   ├── try-catch-finally：传统写法，手动 close
│   └── try-with-resource（推荐）
│       ├── 格式：try(资源定义;资源定义){ } catch(异常){ }
│       ├── 特点：自动调用 close
│       └── 资源要求：必须实现 AutoCloseable 接口
│
├── 字符流（= 字节流 + 编码表）
│   ├── 使用场景：文本文件单独读写，推荐字符流
│   ├── FileReader：read() 读一个字符 / read(字符数组)
│   └── FileWriter
│       ├── 写方法：write(int/字符数组/字符数组片段/String/String片段)
│       └── 注意：有缓冲区，须 flush 或 close；close 内部会自动 flush
│
└── 包装流（对原始流增强，构造时传入原始流对象）
    ├── 缓冲流
    │   ├── BufferedInputStream / BufferedOutputStream：8192字节缓冲区
    │   ├── BufferedReader：8192字符缓冲区，特有 readLine()
    │   └── BufferedWriter：8192字符缓冲区，特有 newLine()
    ├── 转换流
    │   ├── InputStreamReader：字节流→字符流，可自定义编码表
    │   ├── OutputStreamWriter：字符流→字节流，可自定义编码表
    │   └── 补充：JDK11起 FileReader/FileWriter 也支持自定义编码表
    ├── 打印流（待展开）
    └── 序列化流（待展开）
```
