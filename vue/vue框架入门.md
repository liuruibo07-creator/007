# Vue 框架基础总结 · Day12

> 本文档整理自 Vue 框架学习思维导图，涵盖 Vue 概述、入门程序、常用指令、生命周期、AJAX 原理以及 Axios 的完整使用语法。

---

## 目录

1. [Vue 概述](#一vue-概述)
2. [Vue 入门程序](#二vue-入门程序)
3. [Vue 指令](#三vue-指令)
4. [单向绑定与双向绑定](#四单向绑定与双向绑定)
5. [Vue 生命周期](#五vue-生命周期)
6. [AJAX](#六ajax)
7. [Axios](#七axios)

---

## 一、Vue 概述

**Vue** 是一个用于**构建用户界面**的渐进式 JavaScript 框架。

| 关键词 | 说明 |
|--------|------|
| **构建用户界面** | 基于数据渲染视图，数据驱动页面展示 |
| **渐进式** | 可以只用其中一部分功能，也可以配合全家桶完整使用 |
| **JavaScript 框架** | 本质是对原生 JS 的封装，简化开发流程 |

---

## 二、Vue 入门程序

### 完整步骤

```html
<!-- 第一步：导入 Vue 模块 -->
<script type="module">
    import { createApp } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'

    // 第二步：基于 createApp 函数创建 Vue 实例
    createApp({

        // 第五步：定义数据模型（data 方法返回 JS 对象）
        data() {
            return {
                message: 'Hello Vue!'
            }
        }

    // 第四步：用 Vue 实例挂载视图
    }).mount('#app')
</script>

<!-- 第三步：定义被 Vue 控制的视图（通常是一个 div） -->
<div id="app">
    <!-- 第六步：通过插值表达式展示数据 -->
    <p>{{ message }}</p>
</div>
```

### 步骤总结

| 步骤 | 操作 | 说明 |
|------|------|------|
| ① | 导入 Vue 模块 | 引入 Vue 3 的 ES Module 版本 |
| ② | 创建 Vue 实例 | 调用 `createApp()` 函数 |
| ③ | 定义视图 | 通常是一个带 `id` 的 `<div>` |
| ④ | 挂载视图 | 调用 `.mount('#id')` |
| ⑤ | 定义数据模型 | 在 `data()` 方法中返回 JS 对象 |
| ⑥ | 插值表达式 | 使用 `{{ 属性名 }}` 在视图中渲染数据 |

---

## 三、Vue 指令

### 3.1 什么是指令

**指令**是加在 HTML 标签上的特殊属性，以 **`v-xxx`** 开头，用于扩展标签功能。

---

### 3.2 v-for：列表渲染

**作用：** 遍历数组或对象，重复渲染标签。

> ⚠️ 写在**需要重复渲染的标签**上；`:key` 建议使用主键 `id`，**尽量不用 `index`**。

#### 遍历数组

```html
<!-- 完整写法 -->
<li v-for="(item, index) in 数组名" :key="唯一值">
    {{ index }} - {{ item }}
</li>

<!-- 省略 index 的写法 -->
<li v-for="item in 数组名" :key="item.id">
    {{ item.name }}
</li>
```

| 变量 | 含义 |
|------|------|
| `item` | 数组中的每个元素 |
| `index` | 当前元素的索引（可省略） |

#### 遍历对象

```html
<li v-for="(value, key) in 对象名" :key="key">
    {{ key }}: {{ value }}
</li>
```

| 变量 | 含义 |
|------|------|
| `value` | 属性值 |
| `key` | 属性名 |

---

### 3.3 v-bind：属性动态绑定

**作用：** 给标签的属性**动态绑定**数据模型中的值。

```html
<!-- 完整写法 -->
<img v-bind:src="imageUrl">

<!-- 简写（省略 v-bind，推荐） -->
<img :src="imageUrl">
<a :href="link">点击跳转</a>
```

---

### 3.4 v-model：双向绑定

**作用：** 给**表单标签**进行双向数据绑定，底层操作的是 `value` 属性。

```html
<input type="text" v-model="username">
<select v-model="city">
    <option value="bj">北京</option>
</select>
```

```js
data() {
    return {
        username: '',
        city: 'bj'
    }
}
```

---

### 3.5 v-if / v-else-if / v-else 与 v-show：条件渲染

**作用：** 根据条件控制标签的**显示与隐藏**。

```html
<!-- v-if 系列 -->
<p v-if="score >= 90">优秀</p>
<p v-else-if="score >= 60">及格</p>
<p v-else>不及格</p>

<!-- v-show -->
<p v-show="isVisible">可见内容</p>
```

#### v-if vs v-show 对比

| 对比项 | `v-if` | `v-show` |
|--------|--------|----------|
| 实现原理 | 通过**创建/删除 DOM 元素**控制显隐 | 通过 **`display` 样式属性**控制显隐 |
| 初始渲染开销 | 条件为假时不渲染，开销小 | 无论条件真假都会渲染 |
| 切换开销 | 高（每次都要增删 DOM） | 低（只切换样式） |
| **适用场景** | 不频繁切换的场景 | **频繁切换**的场景 |

---

### 3.6 v-on：事件绑定

**作用：** 给标签绑定事件，触发时执行回调函数。

```html
<!-- 完整写法 -->
<button v-on:click="handleClick">点击</button>

<!-- 简写（@ 符号，推荐） -->
<button @click="handleClick">点击</button>
<input @keyup.enter="handleSearch">
```

```js
// 回调函数必须定义在 methods: {} 中
methods: {
    handleClick() {
        console.log('按钮被点击了')
    }
}
```

> ⚠️ **注意：** 回调函数必须定义在 `methods: {}` 中，不能写在 `data()` 里。

---

### 3.7 指令综合案例

**数据模型中的变量和 `methods` 中的方法都挂载到 Vue 实例上**，可通过 `this.变量名` 和 `this.方法名()` 访问，其中 `this` 就代表 Vue 实例。

| 功能 | 用到的指令 |
|------|------------|
| 将对象数组渲染成表格 | `v-for`、`v-if`/`v-show`、`{{ 三元运算符 }}` |
| 查询表单数据并打印 | `v-on`、`v-model` |

---

## 四、单向绑定与双向绑定

| 绑定类型 | 数据改变时视图变化 | 视图改变时数据变化 | 代表指令 |
|----------|-------------------|-------------------|----------|
| **单向绑定** | ✅ 是 | ❌ 否 | `v-bind`、`{{ }}` |
| **双向绑定** | ✅ 是 | ✅ 是 | `v-model` |

> 💡 **Vue 指令中只有 `v-model` 支持双向绑定**，其他指令均为单向绑定。

---

## 五、Vue 生命周期

### 5.1 什么是 Vue 生命周期

Vue 实例从**创建到销毁**的整个过程，称为 Vue 的生命周期。

```
创建阶段 → 挂载阶段 → 更新阶段 → 销毁阶段
```

### 5.2 mounted 钩子函数

| 钩子函数 | 触发时机 | 常用场景 |
|----------|----------|----------|
| `mounted` | Vue 实例挂载完毕、DOM 渲染完成后触发 | **向后台发请求获取初始数据** |

```js
createApp({
    data() {
        return { empList: [] }
    },
    mounted() {
        // 页面加载完成后自动请求数据
        axios.get('/api/emps/list').then(result => {
            this.empList = result.data.data
        })
    }
}).mount('#app')
```

---

## 六、AJAX

### 6.1 什么是 AJAX

**AJAX**（Asynchronous JavaScript and XML）是一种服务器通信技术，可以实现：

- **异步交互**：不刷新整个页面，与服务器进行数据交换
- **局部刷新**：只更新页面中需要变化的部分

### 6.2 底层实现

AJAX 基于浏览器提供的 **`XMLHttpRequest`（XHR）** 对象实现。

### 6.3 异步 vs 同步

| 类型 | 特点 | 场景 |
|------|------|------|
| **异步** | 客户端**不等待**服务器响应，继续向下执行 | AJAX 请求（推荐） |
| **同步** | 客户端**等待**服务器响应后，才继续执行 | 阻塞式操作 |

---

## 七、Axios

### 7.1 什么是 Axios

Axios 是对原生 AJAX（XHR）的**封装框架**，简化了与服务器交互的代码编写，是目前最流行的前端 HTTP 请求库。

> ✅ **推荐使用方式 2（链式简写）**，代码更简洁。

---

### 7.2 方式一：配置对象写法

#### GET 请求（不带参数）

```js
axios({
    method: 'GET',
    url: 'https://mock.apifox.cn/m1/3083103-0-default/emps/list'
}).then((result) => {
    console.log(result.data)
}).catch((err) => {
    console.log(err)
})
```

#### GET 请求（带参数 - URL 拼接）

```js
axios({
    method: 'GET',
    url: 'https://hmajax.itheima.net/api/city?pname=河北省'
}).then((result) => {
    console.log(result.data)
}).catch((err) => {
    console.log(err)
})
```

#### GET 请求（带参数 - params 对象）

```js
axios({
    method: 'GET',
    url: 'https://hmajax.itheima.net/api/city',
    params: {
        pname: '河北省'
    }
}).then((result) => {
    console.log(result.data)
}).catch((err) => {
    console.log(err)
})
```

#### POST 请求（Form 表单格式）

```js
axios({
    method: 'POST',
    url: 'https://mock.apifox.cn/m1/3083103-0-default/emps/update',
    data: 'id=1&name=zs'   // key=value 字符串格式
}).then((result) => {
    console.log(result.data)
}).catch((err) => {
    console.log(err)
})
```

#### POST 请求（JSON 格式）

```js
axios({
    method: 'POST',
    url: 'https://mock.apifox.cn/m1/3083103-0-default/emps/update',
    data: {             // JS 对象格式，自动序列化为 JSON
        id: 1,
        name: 'zs'
    }
}).then((result) => {
    console.log(result.data)
}).catch((err) => {
    console.log(err)
})
```

---

### 7.3 方式二：链式简写（推荐 ✅）

#### GET 请求（不带参数）

```js
axios.get('https://mock.apifox.cn/m1/3083103-0-default/emps/list')
    .then((result) => {
        console.log(result.data)
    }).catch((err) => {
        console.log(err)
    })
```

#### GET 请求（带参数 - URL 拼接）

```js
axios.get('https://hmajax.itheima.net/api/city?pname=河北省')
    .then((result) => {
        console.log(result.data)
    }).catch((err) => {
        console.log(err)
    })
```

#### GET 请求（带参数 - params 对象）

```js
axios.get('https://hmajax.itheima.net/api/city', {
    params: {
        pname: '河北省'
    }
}).then((result) => {
    console.log(result.data)
}).catch((err) => {
    console.log(err)
})
```

#### POST 请求（Form 表单格式）

```js
axios.post(
    'https://mock.apifox.cn/m1/3083103-0-default/emps/update',
    'id=1&name=zs'           // 第二个参数：key=value 字符串
).then((result) => {
    console.log(result.data)
}).catch((err) => {
    console.log(err)
})
```

#### POST 请求（JSON 格式）

```js
axios.post(
    'https://mock.apifox.cn/m1/3083103-0-default/emps/update',
    { id: 1, name: 'zs' }   // 第二个参数：JS 对象
).then((result) => {
    console.log(result.data)
}).catch((err) => {
    console.log(err)
})
```

---

### 7.4 两种方式对比

| 对比项 | 方式一（配置对象） | 方式二（链式简写）|
|--------|-------------------|-------------------|
| 代码量 | 较多 | 较少 |
| 可读性 | 高，参数一目了然 | 简洁 |
| 推荐程度 | 了解即可 | ✅ 推荐使用 |

### 7.5 POST 请求体格式对比

| 格式类型 | data 写法 | Content-Type |
|----------|-----------|--------------|
| **Form 表单格式** | `'key=value&key2=value2'` 字符串 | `application/x-www-form-urlencoded` |
| **JSON 格式** | `{ key: value }` JS 对象 | `application/json` |

---

*整理完毕 · Vue 框架基础 Day12*
