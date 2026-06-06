# Web 开发学习笔记

---

## 一、Web 基础概念

| 问题 | 答案 |
|------|------|
| 什么是 Web？ | 通过浏览器访问的网站，如京东、淘宝、学生管理系统等 |
| 网站由什么组成？ | 网页 |
| 网页由什么组成？ | HTML（结构）+ CSS（美化）+ JavaScript（交互） |

---

## 二、HTML

### 2.1 基本概念

**HTML**（HyperText Markup Language）即超文本标记语言。

- **超文本**：超越普通文本的限制，可以嵌入图片、音频、视频、超链接等
- **标记语言**：由一个个标签构成

### 2.2 标签分类

| 类型 | 示例 |
|------|------|
| 单标签 | `<br>` |
| 双标签 | `<h1></h1>` |

---

### 2.3 常用标签

#### 骨架标签

```html
<html>   <!-- 网页根元素 -->
<head>   <!-- 网页头部，内容给浏览器看 -->
<title>  <!-- 网页标题 -->
<body>   <!-- 网页主体，内容给用户看 -->
```

#### 标题标签

- 标签：`<h1>` ~ `<h6>`
- 特点：加粗、依次变小、独占一行
- ⚠️ 注意：网页中 `<h1>` 建议只写一个

```html
<h1>一级标题（最大）</h1>
<h2>二级标题</h2>
<h3>三级标题</h3>
<h4>四级标题</h4>
<h5>五级标题</h5>
<h6>六级标题（最小）</h6>
```

#### 段落标签

- 标签：`<p>`
- 特点：独占一行，段落之间有空隙

```html
<p>这是一个段落。段落之间会自动换行并产生间距。</p>
<p>这是第二个段落，它与上一个段落之间有默认空隙。</p>
```

#### 其他基础标签

| 标签 | 作用 |
|------|------|
| `<br>` | 换行 |
| `<hr>` | 水平线 |
| `<strong>` | 加粗 |
| `<em>` | 倾斜 |
| `<ins>` | 下划线 |

```html
<p>第一行<br>第二行（使用了 br 换行）</p>
<hr>
<p>这是一段<strong>加粗</strong>、<em>倾斜</em>和<ins>下划线</ins>的文字。</p>
```

---

#### 图片标签 `<img>`

| 属性 | 说明 | 是否必填 |
|------|------|----------|
| `src` | 图片路径 | ✅ 必填 |
| `alt` | 图片加载失败时的替代文字 | 建议填写 |
| `title` | 鼠标悬停时的提示文字 | 可选 |

```html
<img src="images/photo.jpg" alt="风景照片" title="这是一张风景照">
<!-- 图片加载失败时显示 alt 文字；鼠标悬停时显示 title 文字 -->
```

#### 超链接标签 `<a>`

| 属性 | 说明 |
|------|------|
| `href` | 跳转路径，路径不确定时写 `#` |
| `target` | `_self`（当前页，默认） / `_blank`（新页面打开） |

```html
<a href="https://www.baidu.com" target="_blank">在新窗口打开百度</a>
<a href="#">暂不确定的链接</a>
<a href="tel:10086">拨打电话</a>
```

#### 音频标签 `<audio>`

| 属性 | 说明 |
|------|------|
| `src` | 音频路径 |
| `controls` | 显示控制面板（属性名与值相同可省略值） |
| `loop` | 循环播放 |
| `autoplay` | 自动播放（浏览器默认禁止） |

```html
<audio src="music.mp3" controls loop></audio>
<audio controls>
  <source src="music.mp3" type="audio/mpeg">
  <source src="music.ogg" type="audio/ogg">
  您的浏览器不支持音频播放
</audio>
```

#### 视频标签 `<video>`

| 属性 | 说明 |
|------|------|
| `src` | 视频路径 |
| `controls` | 显示控制面板 |
| `loop` | 循环播放 |
| `autoplay` | 自动播放（浏览器默认禁止，加 `muted` 可实现静音自动播放） |

```html
<video src="movie.mp4" controls muted autoplay loop width="600"></video>
<!-- muted + autoplay 可实现静音自动播放 -->
```

---

#### 列表标签

**无序列表**
```html
<ul>
  <li>列表项</li>   <!-- ul 内只能写 li，li 内可写任意内容 -->
</ul>
```

**有序列表**
```html
<ol>
  <li>列表项</li>   <!-- ol 内只能写 li，li 内可写任意内容 -->
</ol>
```

**定义列表**
```html
<dl>
  <dt>标题</dt>     <!-- dl 内只能写 dt 和 dd -->
  <dd>描述内容</dd>
</dl>
```

---

#### 表格标签

```html
<table>
  <tr>
    <th>表头单元格</th>
  </tr>
  <tr>
    <td>普通单元格</td>
  </tr>
</table>
```

**单元格合并**

| 属性 | 说明 |
|------|------|
| `rowspan` | 跨行合并 |
| `colspan` | 跨列合并 |

> ⚠️ 使用注意：将合并属性加在最上方或最左边的单元格上，然后删除多余单元格。

```html
<table border="1">
  <tr>
    <th colspan="2">跨两列的表头</th>
  </tr>
  <tr>
    <td rowspan="2">跨两行的单元格</td>
    <td>普通单元格 1</td>
  </tr>
  <tr>
    <td>普通单元格 2</td>
  </tr>
</table>
```

---

#### 表单标签

**`<form>` 属性**

| 属性 | 说明 |
|------|------|
| `action` | 表单提交的后台地址 |
| `method` | 请求方式：`get`（参数显示在地址栏）/ `post`（参数隐藏，需开发者工具查看），默认 `get` |

**表单项标签**

| 标签 | type 值 | 说明 |
|------|---------|------|
| `<input>` | `text` | 单行文本框（默认） |
| `<input>` | `password` | 密码框 |
| `<input>` | `checkbox` | 多选框（`checked` 表示选中） |
| `<input>` | `radio` | 单选框（`checked` 表示选中） |
| `<select>` | — | 下拉框，嵌套 `<option>`（`selected` 表示选中） |
| `<textarea>` | — | 多行文本域，支持 `placeholder` |
| `<button>` | `submit` | 提交按钮 |
| `<button>` | `reset` | 重置按钮 |
| `<button>` | `button` | 普通按钮，通常配合 JS 使用 |
| `<label>` | — | 扩大点击范围，提升用户体验，直接包裹目标内容 |

> 💡 `text` 和 `password` 类型支持 `placeholder` 属性显示提示文字。

```html
<form action="/submit" method="post">
  <!-- 文本框 -->
  <label>用户名：<input type="text" placeholder="请输入用户名"></label>
  <!-- 密码框 -->
  <label>密码：<input type="password" placeholder="请输入密码"></label>
  <!-- 单选框：name 相同才能互斥 -->
  <label><input type="radio" name="gender" value="male" checked> 男</label>
  <label><input type="radio" name="gender" value="female"> 女</label>
  <!-- 多选框 -->
  <label><input type="checkbox" checked> 同意用户协议</label>
  <!-- 下拉框 -->
  <select>
    <option value="beijing">北京</option>
    <option value="shanghai" selected>上海</option>
    <option value="guangzhou">广州</option>
  </select>
  <!-- 多行文本域 -->
  <textarea placeholder="请输入留言" rows="4" cols="40"></textarea>
  <!-- 按钮 -->
  <button type="submit">提交</button>
  <button type="reset">重置</button>
</form>
```

---

### 2.4 路径

#### 绝对路径

| 类型 | 示例 |
|------|------|
| 本地绝对路径（Windows） | `C:/images/1.jpg` |
| 本地绝对路径（Linux） | `/opt/images/1.jpg` |
| 网络绝对路径 | `https://example.com/1.jpg` |

#### 相对路径

不以盘符或协议开头，相对于自身位置查找。

| 符号 | 含义 |
|------|------|
| `..` | 上一级目录 |
| `.` | 当前目录 |

> ✅ **总结**：本地资源用相对路径，网络资源用绝对路径。

---

## 三、CSS

### 3.1 基本概念

**CSS**（Cascading Style Sheets）即层叠样式表。

### 3.2 引入方式

| 方式 | 位置 | 适用场景 |
|------|------|----------|
| 内部样式 | `<head>` 中的 `<style>` 标签 | 学习阶段 |
| 外部样式 | 外部 `.css` 文件，通过 `<link>` 引入 | 实际工作 |
| 行内样式 | 标签的 `style` 属性 | 配合 JS 使用 |

```html
<!-- 1. 内部样式 -->
<style>
  p { color: red; }
</style>

<!-- 2. 外部样式：style.css 中写 p { color: red; }，然后引入 -->
<link rel="stylesheet" href="style.css">

<!-- 3. 行内样式 -->
<p style="color: red; font-size: 20px;">行内样式文字</p>
```

### 3.3 语法格式

```css
选择器 {
  属性名: 属性值;
  /* ... */
}
```

---

### 3.4 选择器

#### 基础选择器

| 选择器 | 写法 | 说明 |
|--------|------|------|
| 通配符 | `*` | 选中所有元素 |
| 标签 | `div` | 选中指定标签 |
| 类 | `.className` | 选中指定 class |
| ID | `#idName` | 选中指定 id |

```css
*                { margin: 0; padding: 0; }  /* 清除所有边距 */
div              { color: #333; }             /* 所有 div */
.box             { width: 200px; }            /* class="box" */
#header          { height: 60px; }            /* id="header" */
```

#### 层次选择器

| 选择器 | 写法 | 说明 |
|--------|------|------|
| 后代 | `A B` | 选中 A 内所有后代 B |
| 子代 | `A > B` | 只选中 A 的直接子元素 B |
| 相邻兄弟 | `A + B` | 选中紧邻 A 之后的 B |
| 通用兄弟 | `A ~ B` | 选中 A 之后所有的 B |

```css
/* 给定 HTML：<div><p><span>文字</span></p><span>文字2</span></div> */

div span     { color: red; }   /* 选中所有后代 span：文字、文字2 */
div > span   { color: blue; }  /* 只选直接子 span：文字2 */
```

#### 伪类选择器

| 选择器 | 说明 |
|--------|------|
| `:hover` | 鼠标悬停状态 |
| `:first-child` | 父元素中的第一个子元素 |
| `:last-child` | 父元素中的最后一个子元素 |
| `:nth-child(N)` | 父元素中的第 N 个子元素（N 为正整数） |
| `:first-of-type` | 同类型中的第一个 |
| `:last-of-type` | 同类型中的最后一个 |
| `:nth-of-type(N)` | 同类型中的第 N 个 |

`:nth-child()` 公式用法：

| 公式 | 含义 |
|------|------|
| `2n` | 偶数项 |
| `2n+1` | 奇数项 |
| `5n` | 5 的倍数项 |

```css
a:hover           { color: orange; }           /* 鼠标悬停 */
li:first-child    { font-weight: bold; }       /* 第一个 li */
li:nth-child(2n)  { background: #f0f0f0; }    /* 偶数行背景色 */
```

#### 属性选择器

| 写法 | 说明 |
|------|------|
| `[属性名]` | 含有该属性的元素 |
| `[属性名="xxx"]` | 属性值等于 xxx |
| `[属性名^="xxx"]` | 属性值以 xxx 开头 |
| `[属性名$="xxx"]` | 属性值以 xxx 结尾 |
| `[属性名*="xxx"]` | 属性值包含 xxx |

```css
input[type="text"]    { border: 1px solid #ccc; }   /* 精确匹配 */
a[href^="https"]      { color: green; }              /* 以 https 开头 */
a[href$=".pdf"]       { color: red; }                /* 以 .pdf 结尾 */
input[name*="user"]   { background: yellow; }        /* 包含 user */
```

#### 伪元素选择器

在标签内部创建一个虚拟元素。

| 选择器 | 说明 |
|--------|------|
| `::before` | 在元素内容最前面创建 |
| `::after` | 在元素内容最后面创建 |

```css
.box::before {
  content: "★";    /* 必须在前面加内容 */
  color: red;
}
.box::after {
  content: "→";
  color: blue;
}
/* 结果：★ 原始内容 → */
```

#### 并集 & 交集选择器

| 类型 | 写法 | 说明 |
|------|------|------|
| 并集 | `A, B` | 多个选择器共享相同样式 |
| 交集 | `AB`（紧挨着写） | 同时满足 A 和 B 的元素 |

```css
h1, h2, h3 { color: #333; }       /* 并集：一级到三级标题共用颜色 */
div.box    { border: 1px solid; }  /* 交集：既是 div 又有 class="box" */
```

---

### 3.5 常见样式属性

#### 字体 font

| 属性 | 说明 | 常用值 |
|------|------|--------|
| `font-size` | 字号大小 | `16px` 等 |
| `font-weight` | 是否加粗 | `700` / `bold` / `400` / `normal` |
| `font-style` | 是否倾斜 | `italic` / `normal` |
| `font-family` | 字体 | `楷体`、`Arial` 等 |
| `line-height` | 行高（字体高度 + 上下边距） | `1.5`、`24px` 等 |

```css
p {
  font-size: 16px;
  font-weight: 700;         /* bold */
  font-style: italic;       /* 倾斜 */
  font-family: "Microsoft YaHei", Arial, sans-serif;
  line-height: 1.5;         /* 1.5 倍行高 */
}
```

**font 复合属性**（顺序有要求）：
```css
font: italic bold 16px/1.5 楷体;
/*   倾斜  加粗  字号/行高  字体 */
```

#### 文本 text

| 属性 | 说明 | 常用值 |
|------|------|--------|
| `text-align` | 水平对齐 | `left` / `right` / `center` |
| `text-indent` | 文本缩进 | `2em`（1em = 1个字符大小） |
| `text-decoration` | 文本修饰 | `underline` / `none` |
| `color` | 文字颜色 | 英文单词 / `#hex` / `rgb(x,y,z)` |
| `text-shadow` | 文字阴影 | — |

```css
h1 {
  text-align: center;                  /* 居中对齐 */
  text-decoration: underline;          /* 下划线 */
  color: #333;                         /* 文字颜色 */
  text-shadow: 2px 2px 4px #999;      /* 水平偏移 垂直偏移 模糊半径 颜色 */
}
p {
  text-indent: 2em;                    /* 首行缩进两个字符 */
}
```

#### 背景 background

| 属性 | 说明 | 常用值 |
|------|------|--------|
| `background-color` | 背景色 | — |
| `background-image` | 背景图 | `url(路径)` |
| `background-repeat` | 背景图平铺 | 默认平铺 / `no-repeat` |
| `background-position` | 背景图位置 | 数值 / 英文单词（第一个水平，第二个垂直） |
| `background-size` | 背景图缩放 | `cover` / `100%` |

```css
.box {
  background-color: #f5f5f5;                    /* 背景色 */
  background-image: url("images/bg.png");       /* 背景图 */
  background-repeat: no-repeat;                 /* 不平铺 */
  background-position: center;                  /* 居中显示 */
  background-size: cover;                       /* 等比铺满 */
}
```

**background 复合属性**（顺序无要求）：
```css
background: #fff url(bg.png) no-repeat center/cover;
/*          背景色  背景图      平铺      位置/缩放 */
```

---

### 3.6 显示模式

| 模式 | 典型元素 | 特点 |
|------|----------|------|
| 块级（block） | `<div>` | 独占一行，默认宽 100%，高由内容撑开，**可设置宽高** |
| 行内（inline） | `<span>` | 一行可放多个，宽高由内容撑开，**不可设置宽高** |
| 行内块（inline-block） | `<img>` | 一行可放多个，宽高由内容撑开，**可设置宽高** |

**切换显示模式：**
```css
display: block;         /* 切换为块级 */
display: inline;        /* 切换为行内 */
display: inline-block;  /* 切换为行内块 */
```

```html
<div class="block">独占一行的块级元素</div>
<span class="inline">行内元素1</span>
<span class="inline">行内元素2，与上一个在同一行</span>
<div class="inline-block">行内块，可设宽高，共处一行</div>
```
```css
.block        { display: block; width: 200px; height: 100px; background: #e0e0e0; }
.inline       { display: inline; background: lightblue; }
.inline-block { display: inline-block; width: 150px; height: 60px; background: lightgreen; }
```

---

### 3.7 盒子模型

#### 组成结构

```
┌─────────────────────────────┐
│           margin            │  外边距
│  ┌───────────────────────┐  │
│  │        border         │  │  边框
│  │  ┌─────────────────┐  │  │
│  │  │     padding     │  │  │  内边距
│  │  │  ┌───────────┐  │  │  │
│  │  │  │  content  │  │  │  │  内容
│  │  │  └───────────┘  │  │  │
│  │  └─────────────────┘  │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

#### padding 内边距写法

```css
padding: 20px;              /* 上下左右 20px */
padding: 10px 20px;         /* 上下 10px，左右 20px */
padding: 10px 20px 30px;    /* 上 10px，左右 20px，下 30px */
padding: 10px 20px 30px 40px; /* 上 右 下 左（顺时针） */
```

#### border 边框写法

```css
border: 1px solid red;        /* 四边 */
border-top: 1px solid red;    /* 上边框 */
border-bottom: 1px solid red; /* 下边框 */
border-right: 1px solid red;  /* 右边框 */
border-left: 1px solid red;   /* 左边框 */
```

#### 盒子大小计算

- 盒子大小由 **内容 + 内边距 + 边框** 决定，与外边距无关
- **标准盒子**（默认）：添加 padding 和 border 会撑大盒子
- **怪异盒子**：大小固定为设置的宽高，底层自动做减法

```css
box-sizing: border-box; /* 切换为怪异盒子 */
```

#### 版心居中

```css
margin: 0 auto; /* 左右外边距设为 auto 即可居中 */
```
