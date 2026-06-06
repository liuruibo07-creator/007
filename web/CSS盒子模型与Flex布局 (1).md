# CSS 盒子模型 & Flex 弹性布局 笔记

---

## 一、盒子模型

### 1.1 什么是盒子模型

> 实现网页布局的基础，可以理解成网页中的一个矩形区域。

---

### 1.2 盒子的组成

| 层级 | 名称 | CSS 属性 |
|------|------|---------|
| 最内层 | 内容 | `content` |
| 第二层 | 内边距 | `padding` |
| 第三层 | 边框 | `border` |
| 最外层 | 外边距 | `margin` |

---

### 1.3 内容（Content）

直接通过 `height` 和 `width` 设置内容区域的高度与宽度。

---

### 1.4 内边距（Padding）

**按单一方向设置：**

```css
padding-top: 20px;
padding-bottom: 30px;
padding-left: 20px;
padding-right: 20px;
```

**简写规则：**

| 写法 | 含义 |
|------|------|
| `padding: 20px 30px` | 上下 20px，左右 30px |
| `padding: 20px 30px 10px` | 上 20px，左右 30px，下 10px |
| `padding: 10px 20px 30px 40px` | 上、右、下、左（顺时针） |

---

### 1.5 边框（Border）

边框包含三个要素：**大小 · 样式 · 颜色**

**样式可选值：**

| 值 | 效果 |
|----|------|
| `solid` | 实线 |
| `dashed` | 虚线 |
| `dotted` | 点线 |

**按单一方向设置示例：**

```css
border-top: 1px solid red;       /* 上：1px 实线 红色 */
border-bottom: 1px solid blue;   /* 下：1px 实线 蓝色 */
border-left: 1px dashed red;     /* 左：1px 虚线 红色 */
border-right: 2px dotted green;  /* 右：2px 点线 绿色 */
```

---

### 1.6 外边距（Margin）

- 设置方式与内边距相同
- **版心居中：**`margin: 0 auto`

---

### 1.7 盒子尺寸

- 盒子大小只与**内容、内边距、边框**有关，与外边距无关
- 正常情况下，修改内边距或边框会撑大盒子
- 使用 `box-sizing: border-box` 可固定盒子大小（尺寸由 `width` 和 `height` 决定，内边距和边框向内压缩）

---

### 1.8 圆角（Border-Radius）

通过 `border-radius` 设置：

| 值的数量 | 含义 |
|---------|------|
| 1 个值 | 四个方向的圆角半径相同 |
| 2 个值 | 左上+右下，左下+右上 |
| 3 个值 | 左上、右上+左下、右下 |
| 4 个值 | 左上、右上、右下、左下（顺时针） |

**常见用法：**

```css
/* 头像（正方形 → 圆形） */
border-radius: 50%;

/* 胶囊按钮（矩形） */
border-radius: 高度的一半;
```

---

### 1.9 浮动（Float）

```css
float: left;   /* 左浮动 */
float: right;  /* 右浮动 */
```

**浮动元素的特点：**
- 顶对齐
- 具有行内块元素的特点
- 脱标，不占标准流位置

---

### 1.10 浮动塌陷（Float Collapse）

**描述：** 父元素因所有子元素都进行了浮动，导致高度变为 0。

**原因：** 子元素浮动后脱标，不能撑开父元素的高度，且父元素未设置固定高度。

**四种解决方案：**

**方式一：额外标签法**

在父元素的最后添加一个块级元素，设置 `clear: both`：

```html
<div class="parent">
  <div class="float-left">左浮动元素</div>
  <div class="float-right">右浮动元素</div>
  <div style="clear: both;"></div>
</div>
```

**方式二：单伪元素法**

给父元素添加 `::after` 伪元素：

```css
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

**方式三：双伪元素法（推荐）**

同时添加 `::before` 和 `::after`，`::before` 用于解决外边距塌陷，`::after` 用于解决浮动塌陷：

```css
.clearfix::before,
.clearfix::after {
  content: "";
  /* 用 table 而非 block，是为了兼容老版本浏览器 */
  display: table;
}

.clearfix::after {
  clear: both;
}
```

> **注意：** `display: table` 是历史原因延用下来的兼容写法。解决浮动塌陷时 class 建议命名为 `clearfix`。

**方式四：overflow: hidden**

给父元素添加 `overflow: hidden`，触发 BFC 机制，使父元素成为独立容器，强制识别并包裹浮动的子元素：

```css
.parent {
  overflow: hidden;
}
```

| 方式 | 做法 | 推荐度 |
|------|------|--------|
| 额外标签法 | 在父元素末尾添加块级元素并设置 `clear: both` | 一般 |
| 单伪元素 | 父元素 `::after` 设置 `clear: both` | 一般 |
| **双伪元素** | `::before` + `::after` 组合处理 | ✅ 推荐 |
| overflow 法 | 父元素设置 `overflow: hidden` | 一般 |

---

### 1.11 标准流

> 标签在网页中默认的排列规则。

---

### 1.12 外边距塌陷

**描述：** 子元素设置了上外边距，会导致父元素一起向下移动。

**原因：** 子元素的外边距"粘连"了父元素。

**解决方案：**

| 方式 | 做法 |
|------|------|
| 预防 | 子元素不设置外边距，改为父元素设置内边距 |
| 解决 | 父元素设置 `overflow: hidden` |
| 解决 | 父元素设置边框 |

---

### 1.13 相邻盒子垂直方向外边距

> 相邻盒子在垂直方向上的外边距会合并，取**两者中的最大值**。

---

### 1.14 元素溢出（Overflow）

| 属性值 | 效果 |
|--------|------|
| `visible` | 溢出内容显示在边框外（默认） |
| `hidden` | 溢出内容隐藏，不显示滚动条 |
| `scroll` | 无论是否溢出，始终显示滚动条 |
| `auto` | 溢出时才显示滚动条 |

---

### 1.15 清除默认样式

```css
/* 清除所有默认边距 */
* {
  margin: 0;
  padding: 0;
}

/* 清除列表项符号 */
ul, ol {
  list-style: none;
}

/* 清除链接下划线 */
a {
  text-decoration: none;
}
```

---

### 1.16 盒子模型案例要点

- 先消除默认样式
- **父子元素**之间的距离用**内边距**实现，**兄弟元素**之间用**外边距**实现
- 子元素设置负外边距可实现"挤出父元素"效果
- 父子元素不要同时设置边框，否则子元素背景色会覆盖父元素边框
- **背景图片 vs `<img>` 标签：** 不重要的装饰用背景图片，重要内容用 `<img>` 标签

---

---

## 二、Flex 弹性布局

### 2.1 什么是 Flex

> 弹性布局，是目前推荐的布局方案，**不会脱标**。

给父元素添加 `display: flex` 后，子元素沿主轴排列，具有**拉伸**和**挤压**效果。

---

### 2.2 基础布局结构

```css
/* 弹性容器 */
.box {
  display: flex;
  height: 300px;
  border: 1px solid #000;
}

/* 弹性盒子（子元素） */
.box div {
  width: 200px;
  height: 100px;
  background-color: pink;
}
```

**轴的方向：**

| 轴 | 默认方向 |
|----|---------|
| 主轴 | 由左向右 |
| 侧轴 | 由上向下 |

---

### 2.3 主轴对齐方式（justify-content）

```css
justify-content: center;        /* 居中，两侧留白 */
justify-content: flex-start;    /* 从起点排列，终点留白（默认值） */
justify-content: flex-end;      /* 从终点排列，起点留白 */
justify-content: space-around;  /* 两侧留白为中间的一半 */
justify-content: space-evenly;  /* 所有留白均等 */
justify-content: space-between; /* 两端不留白，中间均等留白 */
```

---

### 2.4 侧轴对齐方式

| 属性 | 作用范围 |
|------|---------|
| `align-items` | 设置在**容器**上，控制所有子元素 |
| `align-self` | 设置在**子元素**上，单独控制某一个 |

**可选值：**

```css
align-items: stretch;     /* 无高度时拉伸铺满（默认值） */
align-items: center;      /* 居中，上下留白 */
align-items: flex-start;  /* 从侧轴起点开始 */
align-items: flex-end;    /* 从侧轴终点开始 */
```

---

### 2.5 修改主轴方向（flex-direction）

```css
flex-direction: row;     /* 水平方向（默认值） */
flex-direction: column;  /* 垂直方向 */
```

---

### 2.6 换行（flex-wrap）

```css
flex-wrap: nowrap;  /* 不换行（默认值） */
flex-wrap: wrap;    /* 自动换行 */
```

---

### 2.7 主轴伸缩比（flex）

通过 `flex` 属性设置子元素占剩余空间的份数：

```css
.item {
  flex: 1; /* 占据剩余空间的 1 份 */
}
```

> 数值表示该元素占剩余空间的**份数比例**。

---

### 2.8 多行侧轴对齐方式（align-content）

> 将多行子元素作为一个**整体**来处理。

```css
align-content: flex-start;   /* 从侧轴起点排列，终点留白（默认值） */
align-content: flex-end;     /* 从侧轴终点排列，起点留白 */
align-content: center;       /* 居中，上下留白 */
align-content: space-between;/* 两端不留白，行间均等留白 */
align-content: space-around; /* 两端留白为行间的一半 */
align-content: space-evenly; /* 所有留白均等 */
```

---

*笔记来源：XMind 脑图「盒子模型」&「Flex」*
