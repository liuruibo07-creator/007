# Day28 Java 知识点总结

> 来源：`day28.xmind`  
> 主题：常用工具类、精确小数计算、日期时间 API、数组工具类

## 一、整体知识结构

本节内容主要围绕 Java 常用工具类和日期时间处理展开，重点包括：

| 模块 | 核心内容 |
| --- | --- |
| `Math` | 数学工具类 |
| `System` | 系统工具类、时间原点 |
| `BigDecimal` | 解决小数精度问题 |
| 时间基础概念 | GMT、UTC、CST、中国标准时间 |
| 旧日期时间 API | `Date`、`SimpleDateFormat`、`Calendar` |
| 新日期时间 API | `LocalDate`、`LocalTime`、`LocalDateTime`、`Instant`、`ZoneId` 等 |
| `Arrays` | 数组内容展示、拷贝、排序 |

---

## 二、Math 数学工具类

### 1. 是什么

`Math` 是 Java 提供的数学工具类，用于完成常见数学运算。

### 2. 常用方法

![](assets/day28_summary/file-20260618232147717.png)

---

## 三、System 系统工具类

### 1. 是什么

`System` 是 Java 提供的系统工具类。

### 2.常用方法
![](assets/day28_summary/file-20260618232358408.png)
### 3. 时间原点

计算机中的时间原点为：

```text
1970-1-1 0:0:0
```

该时间点常用于时间戳、毫秒值等时间计算场景。

---

## 四、BigDecimal 精确小数计算

### 1. 是什么

`BigDecimal` 用来解决小数计算中的精度问题。

### 2. 如何创建对象
常见创建方式：

```java
new BigDecimal("0.1");
BigDecimal.valueOf(0.1);
```
重点结论：

> 创建 `BigDecimal` 时，推荐使用字符串格式的构造方法。

也就是：

```java
new BigDecimal("0.1");
```
### 3. 常用方法与舍入模式
![](assets/day28_summary/file-20260618232509392.png)

`BigDecimal` 涉及计算时经常需要指定舍入模式，常见模式包括：

| 舍入模式 | 含义 |
| --- | --- |
| `RoundingMode.UP` | 进一：无论是否大于 5 都进一位 |
| `RoundingMode.FLOOR` | 去尾：无论是否大于 5 都丢弃 |
| `RoundingMode.HALF_UP` | 四舍五入 |
```
public class Test2 {  
    public static void main(String[] args) {  
        BigDecimal a=new BigDecimal("0.3");  
        BigDecimal b=new BigDecimal("0.4");  
        System.out.println(a.add(b));  
        BigDecimal c=a.subtract(b);  
        System.out.println(c);  
        BigDecimal res=a.multiply(b);  
        System.out.println(res);  
        BigDecimal e=new BigDecimal("10");  
        BigDecimal f=new BigDecimal("3");  
        //进一法  RoundingMode.UP不管是大于等于5还是小于5都进一位  
        BigDecimal res2=e.divide(f,2,RoundingMode.UP);  
        System.out.println(res2);  
        // 去尾法  RoundingMode.FLOOR       直接将后面的小数截取不要了  
        BigDecimal res3=e.divide(f,6, RoundingMode.FLOOR);  
        System.out.println(res3);  
        // 四舍五入 RoundingMode.HALF_UP    四舍五入 判断小数点后第三位数字是否大于等于5 如果是则进一 如果不是则不进一  
        BigDecimal res4=e.divide(f,6, RoundingMode.HALF_UP);  
        System.out.println(res4);  
    }  
}
```
### 4. 重要特点

`BigDecimal` 和 `String` 一样，是不可变类。  
每次计算或修改后，通常会返回一个新的对象。

---
## 五、时间基础概念

### 1. 世界标准时间

常见标准时间概念：

- `GMT` 格林威治
- `UTC` 原子钟
### 2. 中国标准时间

中国标准时间为：

```text
CST = UTC + 8小时
```

原因是中国位于东八区。

---
## 六、旧日期时间 API

## 6.1 Date

### 1. 是什么

`Date` 用于表示时间对象，也可以理解为时间戳或某一个时间点。

### 2. 如何创建对象

```java
new Date();
```

表示当前系统时间。

```java
new Date(long 毫秒值);
new Date(0L);//原点时间
```

表示指定毫秒值对应的时间。

### 3. 常用方法

| 方法 | 作用 |
| --- | --- |
| `getTime()` | 获取时间的毫秒值 |
| `setTime(long 毫秒值)` | 设置时间的毫秒值 |

## 6.2 SimpleDateFormat

### 1. 是什么

`SimpleDateFormat` 用来实现 `Date` 和 `String` 之间的相互转换。

- `Date` 转 `String`：格式化
- `String` 转 `Date`：解析

### 2. 如何创建对象

```java
new SimpleDateFormat(String pattern);
```

常用格式符号：

| 符号 | 含义 |
| --- | --- |
| `y` | 年份 |
| `M` | 月 |
| `d` | 日 |
| `H` | 小时 |
| `m` | 分钟 |
| `s` | 秒 |

### 3. 常用方法

| 方法 | 作用 |
| --- | --- |
| `String format(Date date)` | 格式化，将 `Date` 转为 `String` |
| `Date parse(String text)` | 解析，将 `String` 转为 `Date` |
### 4.示例代码
```java
public class SimpleDateFormatDemo {  
    public static void main(String[] args) throws ParseException {  
        //创建SimpleDateFormat对象  
        SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
        //作用1:将Date对象格式化为String  
        Date d1=new Date();  
        String str=sdf.format(d1);  
        System.out.println(str);  
        //作用2:将时间格式的字符串Date对象  
        String str2="2020-01-01 00:00:00";  
        Date d2=sdf.parse(str2);  
        System.out.println(d2);  
    }  
}
```
## 6.3 Calendar

思维导图中列出了 `Calendar` 的三个学习方向：

- 是什么
- 如何创建对象
- 常用方法

但具体内容未展开。

---

## 七、新日期时间 API

## 7.1 LocalDate / LocalTime / LocalDateTime

### 1. 概念

| 类 | 表示内容 |
| --- | --- |
| `LocalDate` | 本地年月日 |
| `LocalTime` | 本地时分秒 |
| `LocalDateTime` | 本地年月日时分秒 |

### 2. 创建对象

| 方法 | 作用 |
| --- | --- |
| `now()` | 创建当前时间对象 |
| `of(xxx)` | 创建指定时间对象 |
```java
//1.获取当前时间的日历对象(包含 年月日)  
LocalDate nowDate = LocalDate.now();  
//System.out.println("今天的日期:" + nowDate);  
//2.获取指定的时间的日历对象  
LocalDate ldDate = LocalDate.of(2023, 1, 1);  
System.out.println("指定日期:" + ldDate);
```
### 3. 常用方法

| 类型 | 方法 |
| --- | --- |
| 查询 | `getXXX` |
| 修改 | `withXXX` |
| 比较 | `isBefore`、`isAfter`、`isEqual` |
| 加减计算 | `minusXXX`、`plusXXX` |
| 解析 | `parse` |

## 7.2 DateTimeFormatter

### 1. 创建对象

```java
DateTimeFormatter.ofPattern(String pattern);
```

### 2. 格式化

```java
format
```

## 7.3 Instant / ZoneId / ZonedDateTime

### Instant

`Instant` 表示时间点或时间戳，不包含时区。

常用创建方法：

- `now()`
- `ofEpochMilli()`
- `ofEpochSecond()`

### ZoneId

`ZoneId` 表示时区。

常用方法：

- `systemDefault()`
- `of("Asia/Shanghai")`
- `getAvailableZoneIds()`

### ZonedDateTime

`ZonedDateTime` 表示带时区的时间。

创建对象：

- `now()`
- `of(xxxx)`

常用方法：

| 类型 | 方法 |
| --- | --- |
| 查询 | `getXXX` |
| 修改 | `withXXX` |
| 比较 | `isBefore`、`isAfter`、`isEqual` |
| 加减计算 | `minusXXX`、`plusXXX` |

## 7.4 Period / Duration / ChronoUnit

### Period

主要用于计算两个日期之间的差值。

```java
Period.between(start, end);
```

计算逻辑：

```text
end - start
```

特点：相减之后获取年月日，不会累计。

### Duration

主要用于计算两个时间之间的差值。

```java
Duration.between(start, end);
```

计算逻辑：

```text
end - start
```

特点：相减之后获取时分秒，会进行累计。

### ChronoUnit

`ChronoUnit` 既可以计算年月日，也可以计算时分秒之间的间隔。

特点：相减之后获取年月日、时分秒，会进行累计。

---

## 八、Arrays 工具类

## 8.1 返回数组内容

将数组以 `[元素1, 元素2, ...]` 的格式返回：

```java
Arrays.toString(类型[] arr);
```

## 8.2 拷贝数组

### 完全拷贝

```java
Arrays.copyOf(类型[] arr, int newLength);
```

说明：第二个参数是新数组的长度。

### 部分拷贝

```java
Arrays.copyOfRange(类型[] arr, 起始索引, 结束索引);
```

说明：包头不包尾。

## 8.3 数组排序

### 基本类型排序

```java
Arrays.sort(基本类型[] arr);
```

作用：按照从小到大升序排序。

### 引用类型排序

引用类型排序有两种方式：

| 排序方式 | 方法 |
| --- | --- |
| 自然排序接口 `Comparable` | `Arrays.sort(引用类型[] arr)` |
| 比较器排序接口 `Comparator` | `Arrays.sort(T[] a, Comparator<? super T> c)` |

### 排序规则

| 方法返回值 | 含义 |
| --- | --- |
| 正数 | 当前要插入的元素比被比较元素大，放在被比较元素右边 |
| 负数 | 当前要插入的元素比被比较元素小，放在被比较元素左边 |
| 零 | 当前要插入的元素与被比较元素相等，放在被比较元素右边 |

---

## 九、复习重点

1. `BigDecimal` 用于解决小数计算精度问题，创建时优先使用字符串构造。
2. `Date`、`SimpleDateFormat`、`Calendar` 属于旧日期时间 API。
3. 新日期时间 API 更清晰地拆分了日期、时间、时区和时间戳。
4. `Period` 偏向日期差值，`Duration` 偏向时间差值，`ChronoUnit` 使用范围更广。
5. `Arrays` 工具类重点掌握数组转字符串、数组拷贝和数组排序。

