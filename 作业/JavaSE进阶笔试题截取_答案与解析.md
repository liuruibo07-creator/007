# JavaSE 进阶笔试题答案与解析

> 来源：`JavaSE进阶笔试题截取.docx`  
> 说明：原文主要由 48 张题目截图组成，部分题号重复或不连续。本文按照题目在文档中的出现顺序重新编号，并在必要处指出原题表述中的歧义。

## 一、面向对象与基础语法

### 第 1 题：接口中只能包含什么类型的成员变量和成员方法？

**答案：** 接口中的成员变量默认为 `public static final`，成员方法传统上默认为 `public abstract`。

**解析：**

- 接口字段本质上是公开静态常量，声明时必须初始化。
- JDK 8 起，接口还可以有 `default` 方法和静态方法；JDK 9 起还可以有私有方法。因此，“接口只能有抽象方法”只适用于早期 Java 语法或传统笔试语境。

---

### 第 2 题：判断题

1. 抽象类可以有构造函数：**正确**。
2. 一个接口可以实现另一个接口：**错误**。
3. 一个接口可以继承多个其他接口：**正确**。
4. 父类中的 `private` 方法可以在子类中访问：**错误**。
5. `protected` 方法在同一个包中的类都可以访问：**正确**。

**解析：**

- 抽象类不能直接实例化，但其构造方法会在创建子类对象时被调用。
- 接口与接口之间使用 `extends`，不是 `implements`。
- 接口支持多继承，例如 `interface C extends A, B`。
- `private` 成员只能在声明它的类内部直接访问。
- `protected` 成员可由同包类访问，也可由不同包的子类在继承语境下访问。

---

### 第 3 题：`String` 与 `StringBuffer` 有什么区别？如何把 `StringBuffer` 转成 `String`？

**答案：**

- `String` 是不可变字符串，每次拼接或修改通常会产生新对象。
- `StringBuffer` 是可变字符串，可以在原对象上追加、插入、删除或修改内容。
- `StringBuffer` 的方法使用同步机制，线程安全，但通常比 `StringBuilder` 慢。
- 转换方法：`buffer.toString()`。

```java
StringBuffer buffer = new StringBuffer("hello");
String str = buffer.toString();
```

---

### 第 4 题：简述 `final`、`finally`、`finalize` 的含义

**答案：**

- `final`：关键字。修饰变量时表示只能赋值一次；修饰方法时表示不能被子类重写；修饰类时表示不能被继承。
- `finally`：异常处理结构中的代码块，一般用于执行资源释放等收尾操作。除 JVM 被强制终止等极端情况外，通常都会执行。
- `finalize()`：`Object` 中曾用于对象回收前清理的方法，但执行时机不可靠，并且已经被弃用，不应作为资源释放方案。

---

### 第 5 题：拆分、排序并重新拼接字符串

题目要求将 `"219,23,9,10,32"` 拆分、按数字升序排列，再转换成 `"9,10,23,32,219"`。

```java
import java.util.Arrays;
import java.util.stream.Collectors;

public class SortNumberString {
    public static void main(String[] args) {
        String source = "219,23,9,10,32";

        String result = Arrays.stream(source.split(","))
                .map(Integer::parseInt)
                .sorted()
                .map(String::valueOf)
                .collect(Collectors.joining(","));

        System.out.println(result);
    }
}
```

**输出：**

```text
9,10,23,32,219
```

**解析：** 必须先转成整数再排序。如果直接按字符串排序，会采用字典顺序，结果不符合数字大小关系。

---

### 第 6 题：方法中的局部变量能否使用 `static`？

**答案：A，编译错误。**

**解析：** 题目在实例方法 `aMethod()` 内声明了 `static int i = 0;`。Java 不允许使用 `static` 修饰局部变量，`static` 只能修饰类级别的成员。因此程序无法通过编译。

---

### 第 7 题：关于继承的正确说法

**答案：A、D。**

- A：Java 类只支持单继承，一个类只能直接继承一个父类。
- B：错误，一个类可以实现多个接口。
- C：错误，一个类可以在继承一个类的同时实现一个或多个接口。
- D：在题目语境下正确，单继承结构相对简单，可以避免多重类继承带来的成员冲突和“菱形继承”问题。

---

### 第 8 题：异常代码的输出结果

**答案（按逻辑执行顺序）：**

```text
ThreeException
Finally
No Exception
Finally
```

**解析：**

1. 第一次循环时 `count++ == 0` 为真，比较后 `count` 变为 1，抛出 `ThreeException`。
2. 第一个 `catch` 捕获该异常，输出 `ThreeException`。
3. `finally` 输出 `Finally`，此时 `count` 不等于 2，不退出。
4. 第二次循环时条件为假，但自增后 `count` 变为 2，输出 `No Exception`。
5. `finally` 再输出 `Finally`，随后因 `count == 2` 执行 `break`。

> 原代码混用了 `System.out` 和 `System.err`。在某些控制台环境下，两种输出流的显示先后可能受缓冲影响，但程序逻辑顺序如上。

---

### 第 9 题：下列属于面向对象特征的是哪些？

选项：重载、重写、封装、继承。

**答案：封装、继承。**

**解析：** 面向对象三大基本特征通常是封装、继承和多态。重载和重写是方法层面的语法机制，其中方法重写是实现运行时多态的重要条件，但本身通常不单列为面向对象基本特征。

---

### 第 10 题：访问修饰符作用范围由大到小

**答案：D，`public > protected > default > private`。**

**解析：**

- `public`：任何位置。
- `protected`：同包，以及不同包的子类在继承条件下。
- `default`：仅同包；这里的 default 指不写访问修饰符。
- `private`：仅当前类内部。

---

### 第 11 题：关于 `final` 关键字，正确的是哪些？

**答案：A、C。**

- A 正确：方法内部可以定义 `final` 局部变量。
- B 错误：空白 `final` 变量可以在构造方法等规定位置完成一次初始化，不一定必须在声明的同一行赋值。
- C 正确：`final` 方法不能被子类重写。
- D 错误：`final` 变量不一定是 `static`；只有 `static final` 才是类级常量。

---

### 第 12 题：创建 `B` 对象时构造方法的输出

**答案：B，输出 `AB`。**

**解析：** 子类构造方法会先隐式执行 `super()`，因此先调用父类 `A` 的构造方法并输出 `A`，再执行子类 `B` 的构造方法并输出 `B`。

---

### 第 13 题：`hashCode()` 和 `equals()` 的区别与联系

**答案：**

- `equals()` 用于判断两个对象在业务逻辑上是否相等。
- `hashCode()` 返回对象的哈希值，哈希集合和哈希映射用它快速确定对象可能所在的位置。
- 如果两个对象通过 `equals()` 判断相等，它们的 `hashCode()` 必须相同。
- 两个对象的 `hashCode()` 相同，不代表 `equals()` 一定为真，因为可能发生哈希冲突。
- 重写 `equals()` 时通常必须同时重写 `hashCode()`，否则对象放入 `HashSet`、`HashMap` 后可能出现不符合预期的行为。

---

### 第 14 题：编写一个单例类

**答案示例（饿汉式）：**

```java
public final class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

**解析：**

1. 私有构造方法阻止外部随意创建对象。
2. 静态字段保存唯一实例。
3. 静态方法向外提供该实例。
4. 类使用 `final` 可防止通过继承破坏单例语义。

---

### 第 15 题：静态代码块、实例代码块和构造方法的输出顺序

**答案：**

```text
this is test1!
this is test2!
this is test3!
```

**解析：** 类首次加载时执行静态代码块；创建对象时先执行实例代码块，再执行构造方法。

---

### 第 16 题：`Math.round(11.5)` 和 `Math.round(-11.5)` 等于多少？

**答案：**

```text
Math.round(11.5)  = 12
Math.round(-11.5) = -11
```

**解析：** `Math.round(x)` 的计算规律可理解为 `floor(x + 0.5)`。因此负数的 `.5` 情况不是简单地“绝对值四舍五入后加负号”。

---

### 第 17 题：Overload 与 Override 的区别；重载能否改变返回值类型？

**答案：**

| 对比项 | Overload（重载） | Override（重写） |
|---|---|---|
| 发生位置 | 通常在同一个类中，也可涉及继承关系 | 父子类之间 |
| 方法名 | 相同 | 相同 |
| 参数列表 | 必须不同 | 必须相同 |
| 返回类型 | 可以不同，但不能只靠返回类型区分重载 | 相同或为协变返回类型 |
| 绑定时期 | 编译期 | 运行期多态 |
| 访问权限 | 无特殊大小关系 | 子类方法不能缩小父类方法的访问权限 |

**结论：** 重载方法可以有不同的返回值类型，但参数列表也必须不同。仅改变返回值类型不能构成重载，因为调用时编译器无法仅依据返回值选择方法。

---

### 第 18 题：多线程有几种实现方法？

**答案：** 常见方式有四类：

1. 继承 `Thread`，重写 `run()`。
2. 实现 `Runnable`，把任务传给 `Thread`。
3. 实现 `Callable<V>`，结合 `FutureTask<V>` 获取任务返回值和异常。
4. 使用线程池提交 `Runnable` 或 `Callable` 任务，这是实际项目中更常见的管理方式。

**解析：** 严格来说，前三种主要描述“任务与线程如何创建”，线程池描述“线程资源如何统一管理”。面试时应根据题目课程口径回答三种或四种，并说明分类依据。

---

### 第 19 题：`throw` 与 `throws` 的区别

**答案：**

| 对比项 | `throw` | `throws` |
|---|---|---|
| 位置 | 方法体内部 | 方法声明的参数列表之后 |
| 作用 | 真正抛出一个异常对象 | 声明方法可能向调用者抛出异常 |
| 后接内容 | 一个异常对象 | 一个或多个异常类型 |
| 示例 | `throw new IOException();` | `void read() throws IOException` |

**解析：** `throw` 是执行动作；`throws` 是风险声明。受检异常如果没有在方法内部捕获，通常需要通过 `throws` 继续交给调用者处理。

---

### 第 20 题：生成 10 个 0～99 的随机数并进行冒泡排序

```java
import java.util.Arrays;
import java.util.Random;

public class BubbleSortDemo {
    public static void main(String[] args) {
        int[] numbers = new int[10];
        Random random = new Random();

        for (int i = 0; i < numbers.length; i++) {
            numbers[i] = random.nextInt(100);
        }

        System.out.println("排序前：" + Arrays.toString(numbers));

        for (int i = 0; i < numbers.length - 1; i++) {
            boolean swapped = false;
            for (int j = 0; j < numbers.length - 1 - i; j++) {
                if (numbers[j] > numbers[j + 1]) {
                    int temp = numbers[j];
                    numbers[j] = numbers[j + 1];
                    numbers[j + 1] = temp;
                    swapped = true;
                }
            }
            if (!swapped) {
                break;
            }
        }

        System.out.println("排序后：" + Arrays.toString(numbers));
    }
}
```

**解析：** 外层循环控制排序轮数，内层循环比较相邻元素。每完成一轮，当前最大值会移动到未排序区间末尾；`swapped` 可以在数组已有序时提前结束。

---

### 第 21 题：简述 `final` 关键字的作用

**答案：**

- 修饰变量：只能赋值一次。引用变量不能再指向其他对象，但其所指对象内部状态仍可能改变。
- 修饰方法：不能被子类重写，但可以被继承和重载。
- 修饰类：不能被继承，例如 `String`。

---

### 第 22 题：关于 `super()` 和 `this()`，正确的是哪一项？

**答案：A。**

**解析：**

- `super()` 用于调用父类构造方法，显式调用时必须位于子类构造方法第一条语句。
- `this()` 用于调用本类的其他构造方法，也必须位于第一条语句。
- 因为两者都必须位于第一条语句，所以不能同时出现在同一个构造方法中。
- 两者都不能在静态上下文中使用。

---

### 第 23 题：`Integer a = 500; Integer b = 500; a == b` 的结果

**答案：`false`。**

**解析：** `Integer.valueOf()` 默认缓存 `-128～127` 范围内的整数对象。`500` 通常不在缓存范围内，装箱时会得到两个不同对象；`==` 比较的是引用地址。比较数值应使用 `a.equals(b)` 或先拆箱。

---

### 第 24 题：关于抽象的说法，正确的是哪一项？

**答案：D，声明抽象方法不可写方法体大括号。**

```java
public abstract void work();
```

**解析：**

- `abstract` 只能修饰类和方法，不能修饰字段。
- 抽象方法没有方法体，声明以分号结尾。
- 抽象方法所在的类必须是抽象类，但抽象类可以没有抽象方法。

---

### 第 25 题：关于 `List<String> data = new ArrayList<>();`，错误的是哪一项？

**答案：C，`data.length()` 可以获得 List 长度。**

**解析：**

- 集合元素数量使用 `data.size()`。
- 数组长度使用 `array.length`。
- 字符串长度使用 `string.length()`。
- `data.add("test111")` 会把元素添加到列表尾部。

> B 项若严格讨论实现细节，`new ArrayList<>()` 默认构造时并不会立刻创建容量为 10 的底层数组；现代 JDK 通常在第一次添加元素时才分配默认容量。因此 B 的描述也不够严谨。若本题要求单选，出题者通常预期答案为 C。

---

### 第 26 题：Java 类中用于定义常量的修饰符是什么？

**答案：通常使用 `public static final`。**

```java
public static final double PI = 3.1415926;
```

**解析：** `final` 保证变量只能赋值一次，`static` 表示常量属于类，`public` 表示允许外部访问。常量名通常采用全大写加下划线的形式。

---

### 第 27 题：`static` 是什么意思？能否重写 `private` 或 `static` 方法？

**答案：**

- `static` 表示成员属于类，而不是属于某一个对象；可通过类名访问。
- `private` 方法对子类不可见，不能被继承，因此不能被重写。子类中同名方法只是一个全新的方法。
- `static` 方法属于类，其调用在编译期确定，不能形成运行时重写。子类定义同签名静态方法称为“方法隐藏”。

---

### 第 28 题：面向对象特征、构造器重写、接口与抽象类

1. **面向对象特征：** 封装、继承、多态；有些教材还会补充抽象。
2. **构造器能否被重写：** 不能。构造器不能被继承，因此不存在重写；同一个类中可以通过不同参数进行构造器重载。
3. **接口能否继承接口：** 能，使用 `extends`，并且支持继承多个接口。
4. **抽象类能否实现接口：** 能，使用 `implements`。抽象类可以暂不实现接口中的全部抽象方法，由具体子类继续实现。

---

### 第 29 题：列出常见的运行时异常

**答案示例：**

- `NullPointerException`：对空引用调用成员。
- `ArrayIndexOutOfBoundsException`：数组索引越界。
- `StringIndexOutOfBoundsException`：字符串索引越界。
- `ArithmeticException`：整数除以零等算术错误。
- `ClassCastException`：不合法的引用类型强制转换。
- `NumberFormatException`：字符串无法转换为目标数字。
- `IllegalArgumentException`：方法参数不合法。
- `ConcurrentModificationException`：以不允许的方式并发修改集合。
- `NoSuchElementException`：请求了不存在的元素。

**解析：** 运行时异常是 `RuntimeException` 及其子类，编译器不强制要求捕获或声明。

---

### 第 30 题：`String`、`StringBuffer`、`StringBuilder` 的区别

| 类型 | 是否可变 | 线程安全 | 性能特点 | 适用场景 |
|---|---|---|---|---|
| `String` | 不可变 | 因不可变而天然安全 | 频繁拼接时可能产生大量对象 | 少量、不频繁变化的字符串 |
| `StringBuffer` | 可变 | 是，方法通常带同步 | 相对较慢 | 多线程共享并修改字符串缓冲区 |
| `StringBuilder` | 可变 | 否 | 通常最快 | 单线程下频繁拼接字符串 |

---

### 第 31 题：如何遍历 `Map`？

**答案：**

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);

// 方式一：keySet
for (String key : map.keySet()) {
    System.out.println(key + "=" + map.get(key));
}

// 方式二：entrySet，通常更适合同时取键和值
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + "=" + entry.getValue());
}

// 方式三：forEach
map.forEach((key, value) -> System.out.println(key + "=" + value));
```

**解析：** 只需要键时使用 `keySet()`；只需要值时可以使用 `values()`；键和值都需要时通常优先 `entrySet()` 或 `forEach()`。

---

### 第 32 题：如何跳出多重循环？

**答案：** 可以使用带标签的 `break`，也可以把循环封装到方法中并使用 `return`。

```java
outer:
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        if (i == 3 && j == 3) {
            break outer;
        }
    }
}
```

**解析：** 普通 `break` 只会结束当前最内层循环；`break outer` 会结束标签所标记的外层循环。

---

### 第 33 题：如何决定选用 `HashMap` 还是 `TreeMap`？

**答案：**

- 只要求快速增删改查，不要求键有序：优先 `HashMap`，平均时间复杂度通常为 O(1)。
- 需要按照键的自然顺序或自定义比较规则排序：使用 `TreeMap`，常见操作时间复杂度为 O(log n)。
- 需要范围查询、首尾键、相邻键等有序映射能力：使用 `TreeMap`。
- 两者通常都不是线程安全的；并发环境应考虑同步方案或并发集合。

---

### 第 34 题：选择题

1. Java 中 `main()` 函数的返回值类型是什么？**D，`void`。**
2. 异常处理中，资源释放、关闭文件和关闭数据库等通常由什么完成？**C，`finally` 子句。**

**补充：** 现代 Java 更推荐使用 try-with-resources 自动关闭实现了 `AutoCloseable` 的资源。

---

### 第 35 题：Java 支持哪些数据类型？什么是自动拆装箱？

**答案：**

Java 数据类型分为两大类：

1. 基本数据类型：`byte`、`short`、`int`、`long`、`float`、`double`、`char`、`boolean`。
2. 引用数据类型：类、接口、数组、枚举等。

自动装箱是编译器自动把基本类型转换成对应包装类对象；自动拆箱则相反。

```java
Integer a = 10; // 自动装箱，相当于 Integer.valueOf(10)
int b = a;      // 自动拆箱，相当于 a.intValue()
```

**注意：** 对 `null` 包装类进行自动拆箱会抛出 `NullPointerException`。

---

### 第 36 题：进程和线程的区别

**答案：**

- 进程是正在运行的程序实例，是操作系统进行资源分配的基本单位之一。
- 线程是进程中的执行单元，是 CPU 调度的基本单位之一。
- 一个进程可以包含多个线程。
- 同一进程中的线程共享堆内存和打开的资源，但各自拥有独立的栈和程序计数器。
- 进程之间隔离性较强；线程之间共享数据更方便，但需要处理线程安全问题。

---

### 第 37 题：`String` 常用方法有哪些？如何快速反转字符串？

**常用方法：**

| 方法 | 作用 |
|---|---|
| `length()` | 获取字符串长度 |
| `charAt(index)` | 获取指定索引字符 |
| `substring(...)` | 截取子串 |
| `equals()` / `equalsIgnoreCase()` | 比较内容 |
| `contains()` | 判断是否包含子串 |
| `indexOf()` / `lastIndexOf()` | 查找位置 |
| `startsWith()` / `endsWith()` | 判断开头或结尾 |
| `replace()` / `replaceAll()` | 替换内容 |
| `split()` | 按规则切分 |
| `trim()` / `strip()` | 去除首尾空白 |
| `toUpperCase()` / `toLowerCase()` | 转换大小写 |
| `isEmpty()` / `isBlank()` | 判断空串或空白串 |
| `valueOf()` | 将其他类型转为字符串 |

快速反转：

```java
String source = "Java";
String reversed = new StringBuilder(source).reverse().toString();
System.out.println(reversed); // avaJ
```

---

### 第 38 题：什么是跨平台性？原理是什么？

**答案：** Java 跨平台性是指同一份编译后的 Java 程序可以在安装了对应 JVM 的不同操作系统上运行，即“一次编译，到处运行”。

**原理：**

```text
Java 源文件
   ↓ javac 编译
平台无关的字节码 .class
   ↓ 不同平台各自的 JVM 解释或即时编译
Windows / Linux / macOS 上的机器指令
```

Java 字节码相对平台无关，真正与操作系统和硬件适配的是各个平台的 JVM。

---

### 第 39 题：`Integer a = 127` 与 `Integer b = 127` 相等吗？

**答案：**

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b);      // true
System.out.println(a.equals(b)); // true
```

**解析：** `127` 位于 `Integer` 默认缓存范围 `-128～127` 内，两次自动装箱通常返回同一个缓存对象，因此 `==` 为真。但业务代码比较包装类数值时仍应使用 `equals()`，不要依赖缓存细节。

---

### 第 40 题：字符串与字符数组作为参数后的输出

**答案：B，`hello and cb`。**

**解析：**

- Java 参数传递统一采用值传递，传入引用类型时，复制的是引用值。
- `str = "test ok"` 只让方法内的局部变量 `str` 指向新字符串，不会改变成员变量 `ex.str`。
- `ch[0] = 'c'` 通过复制得到的引用修改了原字符数组对象，因此 `ab` 变成 `cb`。

---

### 第 41 题：匿名内部类重写 `equals()` 后的结果

**答案：B，`true`。**

**解析：** 匿名内部类重写了 `equals(Object obj)`，并且无条件返回 `true`。因此调用 `o.equals("Fred")` 时执行的是重写后的方法，结果为 `true`。

---

### 第 42 题：关于 `protected` 成员变量，正确的是哪一项？

**答案：A。**

**解析：** `protected` 成员可以由当前类、同包中的其他类，以及不同包中符合继承访问规则的子类访问。不同包子类访问时存在限定：通常需要通过继承得到的当前子类对象来访问，不能任意通过父类对象访问其受保护成员。

---

### 第 43 题：关于构造方法，正确的是哪一项？

**答案：A，一个类至少有一个构造方法。**

**解析：** 如果源代码没有显式声明任何构造方法，编译器会提供一个默认无参构造方法。构造方法没有返回值类型，连 `void` 也不能写；其访问权限不一定是 `public`；也不要求必须显式定义。

---

### 第 44 题：`A` 派生出 `B`，`B` 派生出 `C`，哪些声明正确？

```java
A a0 = new A();
A a1 = new B();
A a2 = new C();
```

**答案：D，三行声明都正确。**

**解析：** 子类对象可以自动向上转型为任意祖先类型。`B` 是 `A` 的子类，`C` 同时也是 `A` 的间接子类，因此三个赋值均可通过编译并正常创建对象。

---

### 第 45 题：输入大于 10 时使用 `throw` 显式抛出异常

**答案：A。**

```java
if (i > 10) {
    throw new Exception("something's wrong!");
}
```

**解析：** `throw` 后面必须跟一个异常对象，创建对象需要使用 `new 异常类型(...)`。如果当前方法没有捕获该受检异常，还需要在方法声明中使用 `throws Exception`。

若题目严格要求“自定义异常”，应先定义异常类：

```java
class InputTooLargeException extends Exception {
    public InputTooLargeException(String message) {
        super(message);
    }
}

if (i > 10) {
    throw new InputTooLargeException("something's wrong!");
}
```

---

### 第 46 题：字符串拼接表达式的输出

已知：

```java
int x = 20, y = 5;
System.out.println(x + y + "" + (x + y) + y);
```

**答案：D，`25255`。**

**解析：** 运算从左到右：先算整数 `x + y` 得到 `25`；遇到空字符串后转为字符串拼接；括号中的 `x + y` 为 `25`；最后拼接 `y`，得到 `"25" + "25" + "5"`。

---

### 第 47 题：关于内部类访问权限，正确的是哪一项？

**答案：B，成员内部类前可以使用 `public`、`protected` 和 `private`。**

**解析：**

- 普通外部顶级类只能使用 `public` 或包访问权限，不能使用 `protected`、`private`。
- 成员内部类与成员变量类似，可以使用四种访问级别。
- 局部内部类不能使用 `public`、`protected`、`private` 等访问修饰符。

---

### 第 48 题：`wait()`、`notifyAll()` 多线程代码的输出顺序

**答案：第二项。**

```text
Thread 2 sent notify.
Thread 1 wake up.
```

**解析：**

1. 题目保证 `t1` 在 `t2` 启动前已经运行到等待状态。
2. `t1` 调用 `obj.wait()` 后释放 `obj` 的监视器锁并进入等待。
3. `t2` 获得锁并调用 `notifyAll()`，这只会唤醒等待线程，不会立即释放锁。
4. `t2` 继续执行打印语句，退出 `synchronized` 后才释放锁。
5. `t1` 重新竞争并获得锁，然后从 `wait()` 之后继续执行并打印。

因此一定先输出线程 2 的通知信息，再输出线程 1 的唤醒信息。

## 二、知识点汇总

| 模块 | 涉及题号 | 重点 |
|---|---|---|
| 面向对象 | 1、2、7、9～14、17、21、22、24、27、28、42～44、47 | 封装、继承、多态、抽象类、接口、构造方法、访问权限 |
| 字符串 | 3、5、30、37、40、46 | 不可变性、可变字符串、参数传递、字符串拼接 |
| 异常 | 4、8、19、29、34、45 | 异常体系、抛出、声明、捕获、资源释放 |
| 包装类 | 23、35、39 | 自动装拆箱、整数缓存、`==` 与 `equals()` |
| 集合 | 13、25、31、33 | List、Map、哈希规则、集合选型和遍历 |
| 算法 | 5、20、32 | 数字排序、冒泡排序、多重循环控制 |
| 静态与代码块 | 6、15、26、27 | `static` 使用范围、初始化顺序、常量 |
| 多线程 | 18、36、48 | 线程创建、进程与线程、等待唤醒机制 |
| Java 平台 | 38 | 字节码、JVM 与跨平台原理 |

## 三、容易出错的结论速查

1. 接口与接口之间用 `extends`，类实现接口用 `implements`。
2. Java 类只支持单继承，但可以实现多个接口。
3. 构造方法不能被继承，因此不能被重写，只能重载。
4. `private` 方法不能重写，`static` 方法只能隐藏，不能形成运行时多态。
5. 重载不能只改变返回值类型，参数列表必须不同。
6. 重写 `equals()` 时通常必须同时重写 `hashCode()`。
7. 包装类比较数值优先使用 `equals()`，不要依赖整数缓存和 `==`。
8. `String` 重新赋值不会修改原字符串对象，数组元素则可以通过引用修改。
9. `wait()` 会释放锁，`notify()`/`notifyAll()` 不会立即释放锁。
10. `finally` 通常会执行，但资源管理优先使用 try-with-resources。
