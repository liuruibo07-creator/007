# 📚 Day 31 — Java 学习笔记总结

> 本文涵盖：可变参数 · Collections 工具类 · Map 集合（特点 / 常用类 / 通用方法 / 三种遍历方式）

---

## 一、可变参数（Varargs）

### 1.1 什么是可变参数？

JDK 5 引入的新特性，是一种**特殊的形参**，可以定义在方法的形参列表中。

### 1.2 语法格式

```java
返回值类型 方法名(数据类型... 参数名称) {
    // 在方法内部，参数名称本质上就是一个数组
}

// 示例
public static void main(String[] args) {  
    System.out.println(getSum(1, 2));  
}

public static int getSum(int...arr){  
    int sum=0;  
    for(int i:arr){  
        sum+=i;  
    }  
    return sum;  
}
```

### 1.3 特点

- 可以接受 **0 个、1 个、多个**参数，甚至直接传入一个**数组**。
- 本质上就是一个**数组**，方法体内可以用数组的方式操作它。

### 1.4 注意事项

| 注意事项 | 说明 |
|---|---|
| 只能定义一个 | 方法形参列表中**只能有一个**可变参数 |
| 必须放在最后 | 可变参数只能定义在形参列表的**最后一位** |

```java
// ✅ 正确
public void test(String name, int... scores) { }

// ❌ 错误：可变参数不在最后
public void test(int... scores, String name) { }

// ❌ 错误：有两个可变参数
public void test(int... a, String... b) { }
```

---

## 二、Collections 工具类

### 2.1 什么是 Collections？

`java.util.Collections` 是 Java 提供的**操作单列集合的工具类**，提供了一系列静态方法，方便对集合进行批量操作。

> 注意区分：`Collection`（接口）vs `Collections`（工具类）

### 2.2 常用方法

| 方法签名 | 功能说明 |
|---|---|
| `addAll(Collection<? super T> c, T... elements)` | 为集合**批量添加**多个元素（利用可变参数） |
| `shuffle(List<?> list)` | **打乱** List 集合中元素的顺序（随机洗牌） |
| `sort(List<T> list)` | 按照元素的**自然排序规则**对 List 进行升序排序 |
| `sort(List<T> list, Comparator<? super T> c)` | 按照**比较器规则**对 List 进行排序 |

```java
List<Integer> list = new ArrayList<>();

// 批量添加
Collections.addAll(list, 3, 1, 4, 1, 5, 9, 2, 6);

// 自然排序（升序）
Collections.sort(list);

// 比较器排序（降序）
Collections.sort(list, (a, b) -> b - a);

// 打乱顺序
Collections.shuffle(list);
```

---

## 三、Map 集合（双列集合）

### 3.1 Map 集合的特点

- 由 **key（键）** 和 **value（值）** 组成，每一对称为**键值对（Entry）**。
- **key 不能重复，value 可以重复**。
- key 和 value **一一对应**，每个 key 只能映射到一个 value。

```
Map
├── key1 → value1
├── key2 → value2
└── key3 → value3
```

---

### 3.2 Map 集合的常用实现类

| 实现类 | 底层结构 | 特点 | 类比 |
|---|---|---|---|
| `HashMap` | 哈希表 | key 存取无序、不重复、无索引 | 类比 `HashSet` |
| `LinkedHashMap` | 哈希表 + 双向链表 | key 存取**有序**、不重复、无索引 | 类比 `LinkedHashSet` |
| `TreeMap` | 红黑树 | key 按内容**排序**、不重复、无索引 | 类比 `TreeSet` |

> 💡 **规律：** Map 的 key 部分等价于对应的 Set 集合，底层原理相同。

---

### 3.3 通用方法

| 方法 | 功能说明 |
|---|---|
| `put(K key, V value)` | 添加 / 修改键值对；key 已存在时**覆盖 value 并返回旧 value**，不存在时添加并返回 `null` |
| `remove(Object key)` | 根据 key 删除对应键值对，返回被删除的 value |
| `get(Object key)` | 根据 key 获取对应的 value，key 不存在时返回 `null` |
| `containsKey(Object key)` | 判断是否包含指定的 key |
| `containsValue(Object value)` | 判断是否包含指定的 value |
| `size()` | 获取 Map 中键值对的数量 |
| `isEmpty()` | 判断 Map 是否为空 |
| `clear()` | 清空所有键值对 |
| `keySet()` | 返回所有 key 组成的 `Set` 集合 |
| `values()` | 返回所有 value 组成的 `Collection` 集合 |
| `entrySet()` | 返回所有键值对组成的 `Set<Map.Entry<K,V>>` 集合 |
```java
//方法的作用: 把集合中的元素,存储到指定的数组中  
String[] arr1=c.toArray(new String[0]);  
System.out.println(Arrays.toString(arr1));
```
---

### 3.4 三种通用遍历方式

#### ① keySet 遍历（先取 key，再取 value）

```java
Map<String, Integer> map = new HashMap<>();
map.put("张三", 90);
map.put("李四", 85);

// 获取所有 key
Set<String> keys = map.keySet();

// 遍历 key，通过 key 获取 value
for (String key : keys) {
    Integer value = map.get(key);
    System.out.println(key + " → " + value);
}
```

**适用场景：** 只需要 key，或者通过 key 间接获取 value。

---

#### ② entrySet 遍历（取出所有键值对，同时获取 key 和 value）

```java
// 获取所有键值对
Set<Map.Entry<String, Integer>> entries = map.entrySet();

for (Map.Entry<String, Integer> entry : entries) {
    String key = entry.getKey();
    Integer value = entry.getValue();
    System.out.println(key + " → " + value);
}
```

**适用场景：** 需要同时操作 key 和 value，**推荐使用**，性能优于 keySet 方式。

---

#### ③ forEach 方法（最简洁）

```java
// forEach(BiConsumer<? super K, ? super V> action)
// BiConsumer：Java 提供的函数式接口（消费型），一次消费两个参数，无返回值

// 匿名内部类写法
map.forEach(new BiConsumer<String, Integer>() {
    @Override
    public void accept(String key, Integer value) {
        System.out.println(key + " → " + value);
    }
});

// Lambda 简化写法（推荐）
map.forEach((key, value) -> System.out.println(key + " → " + value));
```

**适用场景：** 代码最简洁，日常开发**最推荐**使用。

---

### 3.5 三种遍历方式对比

| 遍历方式 | 获取内容 | 代码简洁度 | 推荐场景 |
|---|---|---|---|
| `keySet()` | key → 再通过 `get()` 拿 value | 一般 | 只需要 key 时 |
| `entrySet()` | key + value 同时获取 | 较好 | 同时操作 key 和 value |
| `forEach()` | key + value 同时获取 | ⭐ 最简洁 | 日常开发首选 |

---
>案例代码

```

```
## 📝 本日知识点总览

```
day31
├── 可变参数（Varargs）
│   ├── JDK5 新特性，格式：数据类型... 参数名
│   ├── 特点：接受 0/1/多个参数或数组，本质是数组
│   └── 注意：只能有一个，且必须放在形参最后
│
├── Collections 工具类
│   ├── addAll()：批量添加元素
│   ├── shuffle()：打乱 List 顺序
│   ├── sort(list)：自然排序
│   └── sort(list, comparator)：比较器排序
│
└── Map 集合（双列集合）
    ├── 特点：key 不重复，value 可重复，一一对应
    ├── 常用实现类
    │   ├── HashMap：无序（类比 HashSet）
    │   ├── LinkedHashMap：存取有序（类比 LinkedHashSet）
    │   └── TreeMap：按 key 排序（类比 TreeSet）
    ├── 通用方法：put/remove/get/containsKey/size/clear/keySet/entrySet...
    └── 三种遍历方式
        ├── keySet()：先取所有 key，再 get(key) 取 value
        ├── entrySet()：取出 Entry 对象，同时获取 key 和 value
        └── forEach(BiConsumer)：Lambda 最简洁，日常首选
```
