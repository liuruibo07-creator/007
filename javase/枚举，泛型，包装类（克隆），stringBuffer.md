# Day 27 · Java 进阶知识总结

> 涵盖：**枚举** · **泛型** · **Object 类** · **包装类** · **可变字符串**

---

## 一、枚举（Enum）

### 1.1 如何定义枚举

通过 `enum` 关键字来定义枚举。

**语法规则：**
- 第一行**只能**写枚举项，多个枚举项之间用逗号分隔
- 从第二行开始可以写类的成员（字段、方法等）
- 构造方法是**私有的**
- 枚举项的完整语法：`枚举项名称(构造方法的实参列表)`；若调用无参构造，建议将 `()` 省略

```java
public enum Season {
    SPRING, SUMMER, AUTUMN, WINTER; // 第一行只写枚举项

    // 从第二行开始可以写类的成员
    private String description;

    // 构造方法是私有的
    private Season() { }
}
```

---

### 1.2 枚举的特点

| 特点 | 说明 |
|------|------|
| 本质是最终类 | 枚举本质上是一个 `final` 类，同时继承 `java.lang.Enum` 父类 |
| 枚举项是对象 | 每一个枚举项都是该类的一个对象，并且是 `public static final` 修饰的 |
| 可以定义成员 | 因为本质是 Java 类，所以普通类中能定义的，枚举都可以定义 |
| 内置方法 | `values()` 获取所有枚举对象数组；`valueOf()` 获取单个枚举对象 |
```
public enum A {  
    X("aa"),  
    Y("BB"),  
    Z("CC");  
    //构造方法  
    private A(){  
  
    }  
    //有参构造  
    private A(String name){  
        this.name=name;  
    }  
    private String name;  
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public void show(){  
        System.out.println("name:"+name);  
    }  
}

public class Test {  
    public static void main(String[] args) {  
        // 定义枚举类并认识枚举类的特点  
        //  
        A[] a=A.values();  
        for(int i=0;i<a.length;i++){  
            System.out.println(a[i]);  
        }  
        A x=A.valueOf("Y");  
        System.out.println(x);  
    }  
}
```
---

### 1.3 枚举的应用场景

> **当一个类的对象数量有限且固定时**，使用枚举。

例如：季节（春夏秋冬）、方向（东南西北）、状态码等。

---

### 1.4 枚举实现单例模式

枚举天然支持**饿汉式单例**，是实现单例模式最简洁、最安全的方式：

```java
public enum Singleton {
    INSTANCE; // 唯一实例

    public void doSomething() {
        System.out.println("单例方法");
    }
}

// 使用
Singleton.INSTANCE.doSomething();
```

> 优点：线程安全、防止反序列化破坏单例、代码简洁。

---

## 二、泛型（Generics）

### 2.1 什么是泛型

- JDK 5 引入的新特性
- 用来修饰**类、接口、方法**
- 在**编译阶段**检查数据类型，提高代码安全性和复用性

---

### 2.2 自定义泛型类

**定义语法：**

```java
类名<类型参数, ...>
```

**类型何时确定？** 创建泛型类的对象时，类型参数就确定下来了。

**应用场景：** 当数据类型不确定时，可以将类定义为泛型类。

```java
public class Box<T> {
    private T value;

    public T getValue() { return value; }
    public void setValue(T value) { this.value = value; }
    
}

// 使用
Box<String> strBox = new Box<>();  // 此时 T 确定为 String
Box<Integer> intBox = new Box<>();
```

---

### 2.3 自定义泛型方法

**定义语法：**

```java
修饰符 <T> 返回值类型 方法名(T t)
```

**类型何时确定？** 方法调用时，根据**实参的数据类型**来确定。

**应用场景：** 当方法形参类型不确定时，定义为泛型方法。

```java
public <T> void print(T t) {
    System.out.println(t);
}
```

---

### 2.4 自定义泛型接口

**定义语法：**

```java
接口名<类型参数, ...>
```

**类型何时确定？** 有两种情况：

| 情况 | 说明 |
|------|------|
| 第一种 | 定义实现类时**指定**数据类型 |
| 第二种 | 实现类也不确定数据类型，声明为泛型类（泛型延续） |

**应用场景：** 当接口中数据类型不确定时。

```java
// 泛型接口
public interface Comparable<T> {
    int compareTo(T o);
}

// 实现类指定类型
public class Student implements Comparable<Student> {
    public int compareTo(Student o) { ... }
}

// 实现类继续延续泛型
public class MyList<T> implements List<T> { ... }
```

---

### 2.5 泛型的继承与通配符

#### 继承

> 泛型本身**不支持继承**，但**数据支持继承**。

```java
// 错误示例：List<Object> 和 List<String> 没有继承关系
List<Object> list = new ArrayList<String>(); // ❌ 编译错误

// 正确：数据类型可以继承
String str = "hello"; // String 继承自 Object ✅
```

#### 通配符 `?`

| 通配符 | 含义 |
|--------|------|
| `?` | 表示**任意类型** |
| `? extends Ye` | 表示 `Ye` 类**以及** `Ye` 类的**子类**（上界） |
| `? super Ye` | 表示 `Ye` 类**以及** `Ye` 类的**父类**（下界） |

```java
// 接受 Number 及其子类（Integer, Double 等）
public void process(List<? extends Number> list) { }

// 接受 Integer 及其父类（Number, Object）
public void add(List<? super Integer> list) { }
```

---

### 2.6 泛型擦除

> 泛型**只存在于源文件中**，一旦编译之后，字节码文件中就不再存在泛型信息。

这意味着：泛型是编译期的类型检查机制，运行时 JVM 并不知道泛型的具体类型。

---

## 三、Object 类

所有 Java 类的顶级父类，提供了一组通用方法。

### 3.1 toString()

| 项目 | 说明 |
|------|------|
| 默认行为 | 返回对象的**地址值**（内存地址） |
| 建议操作 | **重写**该方法，返回对象的**属性值**，便于调试和日志输出 |

```java
@Override
public String toString() {
    return "Person{name=" + name + ", age=" + age + "}";
}
```

---

### 3.2 equals()

| 项目 | 说明 |
|------|------|
| 默认行为 | 比较两个对象的**地址值**是否相等（即 `==`） |
| 建议操作 | **重写**该方法，改为比较对象的**属性值** |

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Person)) return false;
    Person person = (Person) o;
    return age == person.age && Objects.equals(name, person.name);
}
```

---

### 3.3 clone()

克隆出一个与原对象一样的**新对象**。

#### 浅克隆 vs 深克隆

| 类型 | 说明 |
|------|------|
| **浅克隆** | 无论是基本类型还是引用类型，直接**复制值**（引用类型复制地址） |
| **深克隆** | 基本类型复制数据值；引用类型**创建新的对象或数组** |
| Object 的 clone | 默认是**浅克隆** |

#### 如何让一个类具有克隆能力

```java
// 步骤1：实现 Cloneable 接口（标记接口，没有任何内容）
public class Person implements Cloneable {

    // 步骤2：重写 clone() 方法，并建议将修饰符改为 public
    @Override
    public Person clone() throws CloneNotSupportedException {
        return (Person) super.clone(); // 浅克隆
    }

    // 步骤3：如果想实现深克隆，需要手动编写代码
    public Person deepClone() {
        Person copy = new Person();
        copy.name = this.name;
        copy.address = new Address(this.address); // 引用类型需要新建对象
        return copy;
    }
}
```

---

## 四、包装类（Wrapper Class）

### 4.1 为什么设计包装类

Java 的设计理念是**万物皆对象**，但基本数据类型不是对象。包装类的出现是为了让基本类型也能以对象的形式参与面向对象编程（如存入集合、使用泛型等）。

---

### 4.2 基本类型与包装类对应关系

| 基本类型 | 包装类 |
|----------|--------|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

---

### 4.3 Integer 详解

#### 创建 Integer 对象

```java
// 方式1：new（已过时，不推荐）
Integer i1 = new Integer(10);

// 方式2：valueOf 静态方法
Integer i2 = Integer.valueOf(10);

// 方式3：直接赋值（推荐）—— 自动装箱
Integer i3 = 10;
```

#### 自动装箱与拆箱

| 操作 | 说明 | 底层实现 |
|------|------|----------|
| **装箱** | 基本类型 → 包装类型 | 调用 `valueOf()` 方法 |
| **拆箱** | 包装类型 → 基本类型 | 调用 `intValue()` 方法 |

```java
// 装箱（自动）
Integer a = 10;           // 底层：Integer.valueOf(10)

// 拆箱（自动）
int b = a;                // 底层：a.intValue()

// 运算时自动拆箱
int c = a + 5;
```

#### int 与 String 的相互转换

```java
// int --> String
String s1 = Integer.toString(10);   // "10"
String s2 = 10 + "";               // "10"（字符串拼接）

// String --> int
int n1 = Integer.parseInt("10");    // 10
int n2 = Integer.valueOf("10");     // 10（返回 Integer，自动拆箱）
```

---

## 五、可变字符串

### 5.1 Java 中的可变字符串类

| 类 | 线程安全 | 性能 | 推荐场景 |
|----|----------|------|----------|
| `StringBuilder` | ❌ 线程不安全 | 更高 | **单线程**字符串操作（推荐） |
| `StringBuffer` | ✅ 线程安全 | 较低 | 多线程环境下的字符串操作 |

> `StringBuilder` 和 `StringBuffer` 功能完全一样，唯一区别是线程安全性。

---

### 5.2 什么时候使用可变字符串

1. **做字符串拼接时** —— 避免频繁创建 `String` 对象（`String` 是不可变的，每次 `+` 都会产生新对象）
2. **对字符串内容进行反转时**

---

### 5.3 如何创建可变字符串对象

```java
// 创建空的可变字符串
StringBuilder sb1 = new StringBuilder();

// 创建并初始化内容
StringBuilder sb2 = new StringBuilder("Hello");
```

---

### 5.4 常用方法

| 方法 | 说明 |
|------|------|
| `append(任意类型)` | 字符串拼接，可拼接任意类型；返回 `this`（对象本身），建议链式调用 |
| `reverse()` | 将字符串反转 |
| `length()` | 返回字符串长度 |
| `toString()` | 将可变字符串转换成 `String` |

#### 链式编程示例

```java
String result = new StringBuilder()
        .append("Hello")
        .append(", ")
        .append("World")
        .append("!")
        .reverse()        // 反转
        .toString();      // 转回 String

System.out.println(result); // !dlroW ,olleH
```

---

## 核心知识点速查

### 枚举 vs 常量

| 对比 | 常量（`static final`） | 枚举 |
|------|----------------------|------|
| 类型安全 | ❌ | ✅ |
| 可扩展性 | 低 | 高（可添加方法、字段） |
| switch 支持 | ✅ | ✅ |
| 单例实现 | 复杂 | 简单 |

### 泛型三种形式对比

| 形式 | 语法 | 类型确定时机 |
|------|------|-------------|
| 泛型类 | `class Box<T>` | 创建对象时 |
| 泛型方法 | `<T> void print(T t)` | 方法调用时 |
| 泛型接口 | `interface List<T>` | 实现类定义或创建对象时 |

### StringBuilder 方法链

```java
// 典型用法：链式调用
String s = new StringBuilder("原始")
        .append("追加内容")
        .insert(0, "插入到开头")
        .reverse()
        .toString();
```
