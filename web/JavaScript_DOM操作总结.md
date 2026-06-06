# JavaScript DOM 操作总结

> 本文档整理自 Day09 & Day10 的学习思维导图，涵盖 DOM 概述、获取元素、操作元素、定时器、事件监听、事件对象及 JS 模块化等核心知识点。

---

## 目录

1. [DOM 概述](#一dom-概述)
2. [获取 DOM 元素](#二获取-dom-元素)
3. [操作 DOM 元素](#三操作-dom-元素)
4. [定时器](#四定时器)
5. [箭头函数](#五箭头函数)
6. [事件监听](#六事件监听)
7. [事件类型](#七事件类型)
8. [事件对象](#八事件对象)
9. [this 对象](#九this-对象)
10. [JS 模块化](#十js-模块化)

---

## 一、DOM 概述

> **Day 09**

**DOM**（Document Object Model，文档对象模型）基于**面向对象**的编程思想，将 HTML 文档抽象成一个个对象，通过对象的属性和方法对文档进行**增删改查**操作。

| 概念 | 说明 |
|------|------|
| 顶级对象 | `document`，代表整个 HTML 文档 |
| 核心思想 | HTML 中的每个标签都是一个对象，可通过 JS 操控 |

---

## 二、获取 DOM 元素

> **Day 09**

### 2.1 新语法（推荐）

```js
// 获取匹配的第一个元素
document.querySelector('CSS选择器')

// 获取匹配的所有元素（返回伪数组）
document.querySelectorAll('CSS选择器')
```

**注意事项：**

1. 参数是一个**字符串**
2. id 选择器加 `#`，class 选择器加 `.`
3. 多个选择器用 `,` 分隔，表示"或"的关系
4. 多个选择器的匹配顺序与 HTML 文档中标签顺序有关，与参数书写顺序无关
5. `querySelectorAll` 返回的是**伪数组**，没有 `push`、`pop` 等数组方法
6. 可通过 `document.body` 快速获取 `<body>` 标签

### 2.2 旧语法（了解）

```js
document.getElementById('id属性值')          // 通过 id 获取（不写 #）
document.getElementsByClassName('class值')   // 通过 class 获取（不写 .）
document.getElementsByTagName('标签名')       // 通过标签名获取
document.getElementsByName('name属性值')      // 通过 name 属性获取
```

---

## 三、操作 DOM 元素

> **Day 09**

### 3.1 操作常见属性

```js
// 通用语法
元素对象.属性名 = 属性值

// 操作元素内容
元素.innerText = '文本'    // 纯文本，不解析标签
元素.innerHTML = '<b>粗体</b>'  // 会解析 HTML 标签

// 修改图片
img.src = '新图片路径'

// 修改表单
input.type = 'password'
input.value = '新内容'
```

### 3.2 操作元素样式

#### 方式一：直接修改 style 属性

```js
元素.style.样式属性名 = 属性值

// 示例
div.style.fontSize = '20px'       // 多单词属性名用小驼峰
div.style.backgroundColor = 'red'
```

#### 方式二：替换 className（会覆盖原有 class）

```js
元素.className = 'class名称'
// ⚠️ 会覆盖元素原有的所有 class，使用时需注意
```

#### 方式三：classList 操作（推荐）

```js
元素.classList.add('class名')     // 追加，不覆盖原有 class
元素.classList.remove('class名')  // 删除指定 class
元素.classList.toggle('class名')  // 切换：有则删除，无则添加
```

| 方式 | 特点 | 推荐场景 |
|------|------|----------|
| `style.属性名` | 单个样式修改灵活 | 少量样式动态修改 |
| `className` | 会覆盖原有 class | 完全替换样式 |
| `classList` | 追加/删除/切换，不影响其他 class | 多数情况下首选 |

### 3.3 自定义属性

```html
<!-- HTML 中定义，以 data- 开头 -->
<div data-id="1" data-type="banner"></div>
```

```js
// JS 中通过 dataset 获取
元素.dataset         // 返回包含所有自定义属性的对象
元素.dataset.id      // 获取 data-id 的值 → '1'
元素.dataset.type    // 获取 data-type 的值 → 'banner'
```

---

## 四、定时器

> **Day 09**

### 4.1 间隔定时器（循环执行）

```js
// 开启：每隔指定毫秒数循环执行一次
let timerId = setInterval(函数, 毫秒值)

// 关闭
clearInterval(timerId)
```

### 4.2 延迟定时器（执行一次）

```js
// 开启：延迟指定毫秒数后执行一次
let timerId = setTimeout(函数, 毫秒值)

// 关闭
clearTimeout(timerId)
```

| 对比 | `setInterval` | `setTimeout` |
|------|---------------|--------------|
| 执行次数 | 循环执行 | 只执行一次 |
| 清除方法 | `clearInterval` | `clearTimeout` |
| 返回值 | 定时器 ID | 定时器 ID |

---

## 五、箭头函数

> **Day 09**

箭头函数是一种简洁的**匿名函数**写法。

```js
// 完整语法
(形参1, 形参2) => {
  函数体
}

// 示例：配合定时器使用
setInterval(() => {
  console.log('每秒执行一次')
}, 1000)
```

> ⚠️ 箭头函数**没有自己的 `this`**（详见第九节）

---

## 六、事件监听

> **Day 10**

### 6.1 基本概念

| 概念 | 说明 |
|------|------|
| **事件** | 发生在元素身上的事情，例如按钮被点击 |
| **事件监听** | 监听元素上发生的事，一旦触发就执行指定函数 |
| **回调函数** | 作为参数被传递的函数 |

### 6.2 新语法（推荐）

```js
元素.addEventListener('事件类型', 回调函数)

// 示例
btn.addEventListener('click', function () {
  console.log('按钮被点击了')
})
```

### 6.3 旧语法（了解）

```js
元素.on事件类型 = 回调函数

// 示例
btn.onclick = function () {
  console.log('点击')
}
```

### 6.4 新旧语法对比

| 对比项 | 旧语法 `on事件` | 新语法 `addEventListener` |
|--------|----------------|--------------------------|
| 绑定多个处理函数 | ❌ 后面会覆盖前面 | ✅ 可以绑定多个，互不影响 |
| 推荐程度 | 了解即可 | ✅ 推荐使用 |

---

## 七、事件类型

> **Day 10**

### 7.1 鼠标事件

```js
元素.addEventListener('click', fn)       // 点击
元素.addEventListener('mouseenter', fn)  // 鼠标进入
元素.addEventListener('mouseleave', fn)  // 鼠标移出
```

### 7.2 焦点事件

```js
元素.addEventListener('focus', fn)  // 获得焦点
元素.addEventListener('blur', fn)   // 失去焦点
```

### 7.3 键盘事件

```js
元素.addEventListener('keydown', fn)  // 键盘按下
元素.addEventListener('keyup', fn)    // 键盘抬起
```

### 7.4 输入事件

```js
// 只能绑定在用户可输入的元素（input、textarea）上
元素.addEventListener('input', fn)  // 每次输入内容变化时触发
```

**键盘与输入事件执行顺序：**

```
keydown  →  input  →  keyup
```

> 💡 `keydown` 无法获取到用户**最后一次输入**的值；`input` 和 `keyup` 可以获取到，**建议键盘监听选用 `keyup`**。

### 7.5 表单提交事件

```js
form.addEventListener('submit', function (e) {
  // 阻止表单默认提交行为（刷新页面 + 跳转）
  e.preventDefault()
  // 在此处用 JS 处理表单数据
})
```

> ⚠️ 表单默认行为：将数据按 `method` 指定的方式提交到 `action` 指定的地址，并刷新页面。实际开发中**必须阻止此默认行为**，改用 JS 处理。

### 7.6 JS 模拟用户操作

```js
元素.click()   // 模拟点击
元素.focus()   // 模拟聚焦
```

---

## 八、事件对象

> **Day 10**

**事件对象**是回调函数的**第一个形参**，封装了与事件相关的属性和方法。

```js
元素.addEventListener('click', function (event) {
  console.log(event)         // 事件对象
  console.log(event.target)  // 触发事件的元素（事件源）
})
```

### 8.1 常用属性

| 属性 | 说明 | 适用事件 |
|------|------|----------|
| `event.target` | 事件源对象（触发事件的元素） | 所有事件 |
| `event.key` | 用户按下的键名（如 `'Enter'`、`'a'`） | 键盘事件 |

### 8.2 常用方法

| 方法 | 说明 |
|------|------|
| `event.preventDefault()` | 阻止事件的默认行为（如表单提交、链接跳转） |

---

## 九、this 对象

> **Day 10**

`this` 的指向取决于**调用方式**：

| 调用场景 | `this` 指向 |
|----------|-------------|
| 对象的方法中 | 该对象本身 |
| 事件回调函数中 | 事件源对象（触发事件的元素） |
| 全局作用域下的普通函数 | `window` 对象 |
| **箭头函数** | ❌ 箭头函数**没有自己的 `this`**，会继承外层作用域的 `this` |

```js
btn.addEventListener('click', function () {
  console.log(this)  // → btn 元素

  setTimeout(() => {
    console.log(this)  // → 仍是 btn（箭头函数继承外层 this）
  }, 1000)
})
```

---

## 十、JS 模块化

> **Day 10**

**模块化**是指将一个大的 JS 文件**按功能拆分成多个小的 JS 文件**，通过 `export` 和 `import` 关键字实现导出与导入。

### 10.1 导出（export）

```js
// checkUsername.js
export function checkUsername() {
  let value = document.querySelector('#username').value
  if (value.length < 4 || value.length > 16) {
    document.querySelector('#name_msg').innerHTML = '用户名不合法'
    return false
  } else {
    document.querySelector('#name_msg').innerHTML = ''
    return true
  }
}
```

### 10.2 导入（import）

```js
// main.js
import { checkUsername } from './checkUsername.js'
```

### 10.3 在 HTML 中使用模块

```html
<!-- 必须加 type="module" -->
<script src="main.js" type="module"></script>
```

> ⚠️ 模块化 JS **必须通过服务器（Server）运行**，直接用浏览器打开本地文件会报跨域错误（`file://` 协议不支持模块化）。

---

*整理完毕 · JavaScript DOM Day09 ~ Day10*
