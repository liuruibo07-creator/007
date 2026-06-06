# JavaScript BOM 知识总结 · Day11

> 本文档整理自 Day11 学习思维导图，涵盖 JS 组成结构、BOM 核心对象详解、本地存储、JSON 格式与转换，以及数组的 map / join 方法。

---

## 目录

1. [JS 的组成](#一js-的组成)
2. [BOM 概述](#二bom-概述)
3. [BOM 五大核心对象](#三bom-五大核心对象)
4. [本地存储](#四本地存储)
5. [JSON](#五json)
6. [数组新方法](#六数组新方法)

---

## 一、JS 的组成

JavaScript 由三部分组成：

| 简称 | 全称 | 说明 |
|------|------|------|
| **ES** | ECMAScript | JS 的核心语法规范 |
| **BOM** | Browser Object Model | 浏览器对象模型 |
| **DOM** | Document Object Model | 文档对象模型 |

---

## 二、BOM 概述

**BOM（浏览器对象模型）** 基于面向对象的编程思想，将浏览器抽象为一个个对象，通过这些对象的**属性和方法**来操控浏览器行为。

---

## 三、BOM 五大核心对象

### 3.1 document — HTML 文档

代表整个 HTML 文档，是 DOM 操作的入口对象（详见 DOM 章节）。

---

### 3.2 location — 地址栏

#### 常用属性

| 属性 | 说明 | 示例 |
|------|------|------|
| `location.href` | 完整的 URL 地址，可读可写（赋值即跳转） | `https://example.com/page?id=1#section` |
| `location.search` | URL 中 `?` 开始的部分（查询参数） | `?id=1&name=tom` |
| `location.hash` | URL 中 `#` 开始的部分（锚点） | `#section` |

#### 常用方法

| 方法 | 说明 | 等效快捷键 |
|------|------|-----------|
| `location.reload()` | 普通页面刷新 | `F5` |
| `location.reload(true)` | 强制刷新（清除缓存） | `Ctrl + F5` |

```js
// 页面跳转
location.href = 'https://www.baidu.com'

// 获取查询参数
console.log(location.search)   // → ?id=1

// 刷新页面
location.reload()
```

---

### 3.3 navigator — 浏览器及设备信息

#### 常用属性

| 属性 | 说明 |
|------|------|
| `navigator.userAgent` | 返回浏览器及操作系统的标识字符串 |

```js
// 常用于判断用户设备类型
if (/Mobi|Android/i.test(navigator.userAgent)) {
    console.log('移动端设备')
} else {
    console.log('PC 端设备')
}
```

---

### 3.4 history — 历史记录

| 方法 | 说明 |
|------|------|
| `history.forward()` | 前进（等同于浏览器前进按钮） |
| `history.back()` | 后退（等同于浏览器后退按钮） |
| `history.go(1)` | 前进一步 |
| `history.go(-1)` | 后退一步 |

```js
history.back()    // 返回上一页
history.forward() // 前进到下一页
history.go(-2)    // 后退两步
```

---

### 3.5 screen — 屏幕信息

提供用户屏幕相关的信息，如屏幕宽高、像素深度等。

---

## 四、本地存储

### 4.1 两种方案对比

| 对比项 | `localStorage` | `sessionStorage` |
|--------|----------------|------------------|
| **存储位置** | 磁盘文件 | 内存 |
| **生命周期** | **永久存储**，刷新或关闭页面数据依然存在 | **临时存储**，页面关闭数据即丢失 |
| **适用场景** | 用户偏好、登录信息等持久化数据 | 临时表单数据、一次性状态 |

---

### 4.2 操作语法

> `key` 和 `value` **都是字符串类型**，存取对象/数组时需配合 JSON 转换。

```js
// 新增 / 修改（key 不存在则新增，已存在则修改）
localStorage.setItem('key', 'value')

// 查询
localStorage.getItem('key')

// 删除
localStorage.removeItem('key')
```

> `sessionStorage` 使用方式与 `localStorage` 完全相同，仅将 `localStorage` 替换为 `sessionStorage` 即可。

---

## 五、JSON

### 5.1 JSON 是什么

JSON 是一种**具有特殊格式要求的字符串**，能清晰地描述 JavaScript 对象的内容，广泛用于数据传输与存储。

---

### 5.2 JSON 格式要求

```json
{"name":"小米", "price":1999, "isNew":true, "hobby":[], "info":{}, "remark":null}
```

| 规则 | 说明 |
|------|------|
| `key` | **必须用双引号**包裹 |
| `value` 类型 | 支持：数字、字符串、布尔、数组、对象、`null` |
| 字符串 value | 使用**双引号** |
| 数字 / 布尔 value | **直接写**，无需引号 |
| 数组 value | 使用 `[]` |
| 对象 value | 使用 `{}` |
| 空值 | 使用 `null` |

> ⚠️ JSON 中**不支持** `undefined`、函数、`Symbol` 类型。

---

### 5.3 JS 对象与 JSON 的相互转换

| 转换方向 | 方法 | 说明 |
|----------|------|------|
| JS 对象 → JSON 字符串 | `JSON.stringify(obj)` | 序列化 |
| JSON 字符串 → JS 对象 | `JSON.parse(jsonStr)` | 反序列化 |

```js
const user = { name: '小明', age: 18 }

// JS 对象 → JSON 字符串
const jsonStr = JSON.stringify(user)
console.log(jsonStr)   // → '{"name":"小明","age":18}'

// JSON 字符串 → JS 对象
const obj = JSON.parse(jsonStr)
console.log(obj.name)  // → '小明'
```

---

### 5.4 JSON 的使用场景

| 场景 | 说明 |
|------|------|
| **本地存储** | 将对象/数组先 `JSON.stringify` 再存入 localStorage |
| **前后端通信** | 作为数据载体，在客户端与服务端之间传递结构化数据 |

```js
// 存储对象到 localStorage
const info = { name: '小明', score: 95 }
localStorage.setItem('info', JSON.stringify(info))

// 从 localStorage 读取对象
const result = JSON.parse(localStorage.getItem('info'))
console.log(result.name)  // → '小明'
```

---

## 六、数组新方法

### 6.1 map() — 映射处理

**作用：** 对数组中的每个元素依次处理，返回一个由**处理结果**组成的**新数组**，**不修改原数组**。

**语法：**

```js
数组名.map(function(元素, 索引) {
    return 处理之后的结果
})

// 箭头函数简写
数组名.map((元素, 索引) => 处理之后的结果)
```

**示例：**

```js
const prices = [100, 200, 300]

// 所有价格打九折
const discounted = prices.map(price => price * 0.9)
console.log(discounted)   // → [90, 180, 270]
console.log(prices)       // → [100, 200, 300]（原数组不变）
```

---

### 6.2 join() — 拼接为字符串

**作用：** 将数组所有元素用指定分隔符拼接，返回**字符串类型**的结果，**不修改原数组**。

**语法：**

```js
// 不传参数：默认用逗号分隔
数组名.join()

// 传入自定义分隔符
数组名.join('分隔符')
```

**示例：**

```js
const fruits = ['苹果', '香蕉', '橙子']

console.log(fruits.join())       // → '苹果,香蕉,橙子'（默认逗号）
console.log(fruits.join(' | '))  // → '苹果 | 香蕉 | 橙子'
console.log(fruits.join(''))     // → '苹果香蕉橙子'（无分隔符）
```

---

### 6.3 map 与 join 联用

**经典组合：** 先用 `map` 处理数组元素，再用 `join` 拼接成字符串（常用于动态生成 HTML 列表）。

```js
const users = ['小明', '小红', '小刚']

const html = users
    .map(name => `<li>${name}</li>`)
    .join('')

console.log(html)
// → '<li>小明</li><li>小红</li><li>小刚</li>'
```

---

*整理完毕 · JavaScript BOM Day11*
