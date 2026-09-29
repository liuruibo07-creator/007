# Day 76｜HarmonyOS 开发环境与 ArkTS、ArkUI 基础

> 本篇整理 HarmonyOS 入门阶段的核心内容：**DevEco Studio 环境准备、项目结构、ArkTS 基础语法以及 ArkUI 声明式页面开发**。

---

## 📌 内容总览

| 模块 | 学习重点 |
|---|---|
| 开发环境 | DevEco Studio 下载、安装路径及插件建议 |
| 项目创建 | Empty Ability 模板、项目参数和设备类型 |
| 工程结构 | `entry` 模块、ArkTS 源码、页面和资源目录 |
| ArkTS | 数据类型、变量、常量、数组、接口、对象和函数 |
| ArkUI | 页面组件、容器组件、内容组件及通用属性 |

---

## 一、开发环境准备

### 1. 下载与安装 DevEco Studio

DevEco Studio 是 HarmonyOS 应用开发的主要集成开发环境。

- 通常可从下载中心获取最新版本。
- 如需指定版本，可进入下载中心的“所有工具”页面选择。
- **安装路径不要包含中文或空格**，避免工具链、SDK 或构建脚本解析路径时出现问题。
- 可以按需要安装 AI 编码辅助插件，提高代码补全和问题排查效率。

> 安装完成后，还需要根据项目要求下载对应版本的 HarmonyOS SDK。

---

## 二、创建 HarmonyOS 项目

### 2. 选择项目模板

入门项目使用 **Empty Ability（空 Ability）** 模板。该模板结构简单，默认仅包含一个 `entry` 主模块，适合学习页面、组件和应用入口。

### 3. 项目参数说明

| 参数 | 含义与注意事项 |
|---|---|
| 项目名称 | 不建议包含中文或空格 |
| 包名 | 不要包含中文和空格，通常采用“反向域名 + 项目名”的形式 |
| 保存路径 | 路径中不要包含中文或空格 |
| Compatible SDK | 应用兼容的 SDK 版本；设备运行环境需要满足相应版本要求 |
| 模块名称 | 入口模块通常命名为 `entry`，同样避免中文或空格 |
| 设备类型 | 可根据目标选择 Phone、Tablet、2in1、Car、Wearable、TV 等 |

> 项目名称、包名和磁盘目录是最容易埋下环境问题的地方，创建项目时应一次检查清楚。

---

## 三、HarmonyOS 工程结构

一个 HarmonyOS 项目可以包含多个模块，但必须有且仅有一个 `entry` 入口模块。应用的入口、主要页面、代码、资源和模块配置通常都位于 `entry` 下。

```text
entry/
└─ src/
   └─ main/
      ├─ ets/
      │  ├─ entryability/
      │  │  └─ EntryAbility.ets
      │  ├─ pages/
      │  │  └─ Index.ets
      │  ├─ components/
      │  ├─ utils/
      │  └─ model/
      ├─ resources/
      │  ├─ base/
      │  │  ├─ element/
      │  │  │  ├─ string.json5
      │  │  │  ├─ color.json5
      │  │  │  └─ float.json5
      │  │  ├─ media/
      │  │  └─ profile/
      │  │     └─ main_pages.json
      └─ module.json5
```

### 4. 主要目录与文件

| 路径 | 作用 |
|---|---|
| `ets/entryability/EntryAbility.ets` | 应用 UIAbility 入口及生命周期相关代码 |
| `ets/pages/Index.ets` | 默认首页页面组件 |
| `ets/components/` | 自定义、可复用的普通组件，通常使用 `@Component` |
| `ets/utils/` | 工具函数和公共方法 |
| `ets/model/` | 数据模型与实体类 |
| `resources/base/element/string.json5` | 文本和多语言字符串资源 |
| `resources/base/element/color.json5` | 颜色资源 |
| `resources/base/element/float.json5` | 尺寸、字体大小等资源 |
| `resources/base/media/` | 图片、图标和 SVG 等媒体资源 |
| `resources/base/profile/main_pages.json` | 页面路由注册表，登记 `@Entry` 页面 |
| `module.json5` | 模块的核心配置文件 |

> 打包时，`entry/src/main` 下的业务代码、页面、资源和配置会共同参与生成应用安装包。

---

## 四、默认 `Index.ets` 页面

下面是默认页面结构的简化示例：

```ts
@Entry
@Component
struct Index {
  @State message: string = 'Hello World';

  build() {
    RelativeContainer() {
      Text(this.message)
    }
    .height('100%')
    .width('100%')
  }
}
```

### 5. 核心语法说明

| 语法 | 作用 |
|---|---|
| `@Entry` | 标记页面入口组件，可作为页面加载与显示 |
| `@Component` | 标记一个自定义 UI 组件 |
| `struct Index` | 定义名为 `Index` 的组件结构体 |
| `@State` | 声明状态变量；状态变化时触发相关 UI 刷新 |
| `build()` | 描述组件 UI 结构的必需方法 |
| `RelativeContainer()` | 相对布局容器 |
| `Text()` | 显示文本内容 |
| `.height()`、`.width()` | 链式设置组件属性 |

页面是一类可以被路由加载的特殊组件，但普通组件不一定是页面。

---

## 五、ArkTS 基础语法

ArkTS 是 HarmonyOS 的主力应用开发语言。它在 TypeScript 基础上扩展了声明式 UI 和状态管理能力，便于开发多设备应用。

### 6. 常用数据类型

| 类型 | 含义 | 示例 |
|---|---|---|
| `string` | 字符串 | `'HarmonyOS'` |
| `number` | 数字 | `100`、`3.14` |
| `boolean` | 布尔值 | `true`、`false` |

### 7. 变量与常量

```ts
let userName: string = '张三';
let age: number = 18;
let enabled: boolean = true;

const appName: string = 'MyApplication';
```

- `let` 定义可以重新赋值的变量。
- `const` 定义不能重新赋值的常量。
- 推荐显式写出类型，便于阅读和编译期检查。

### 8. 数组

```ts
let scores: number[] = [80, 90, 100];
let names: string[] = ['张三', '李四', '王五'];
```

数组中的元素需要符合声明的元素类型。

### 9. 接口与对象

接口用于约束对象的结构：

```ts
interface User {
  name: string;
  age: number;
  enabled: boolean;
}
```

官方风格通常使用分号分隔成员；逐行声明时语法也能识别，但保持统一格式更易阅读。

根据接口创建对象：

```ts
let user: User = {
  name: '张三',
  age: 18,
  enabled: true
};
```

对象中的多个属性必须使用逗号分隔，不能省略。

### 10. 普通函数

```ts
function add(a: number, b: number): number {
  return a + b;
}

let result: number = add(10, 20);
```

函数调用时，实参的数量、类型和顺序应与形参保持一致。

### 11. 箭头函数

```ts
let multiply = (a: number, b: number): number => {
  return a * b;
};

let result: number = multiply(3, 4);
```

变量 `multiply` 保存函数，因此也可作为函数名进行调用。

---

## 六、ArkUI 组件基础

ArkUI 使用声明式方式描述页面。组件大体可以分为**容器组件**和**内容组件**。

### 12. 容器组件

#### `Column`：垂直排列

```ts
Column() {
  Text('第一行')
  Text('第二行')
}
```

`Column` 中的子组件默认从上到下排列。

#### `Row`：水平排列

```ts
Row() {
  Text('左侧')
  Text('右侧')
}
```

`Row` 中的子组件默认从左到右排列。

### 13. 内容组件

`Text` 用于显示文字：

```ts
Text('你好，HarmonyOS')
```

也可以绑定状态变量：

```ts
Text(this.message)
```

### 14. 常用通用属性

容器组件和内容组件通常都能设置以下属性：

```ts
Text('Hello')
  .width('100%')
  .height(50)
  .backgroundColor('#F5F5F5')
```

| 属性 | 作用 |
|---|---|
| `width` | 设置组件宽度 |
| `height` | 设置组件高度 |
| `backgroundColor` | 设置组件背景颜色 |

> 原思维导图中写作 `backgroudColor`，正确属性名是 `backgroundColor`。

### 15. 简单页面示例

```ts
@Entry
@Component
struct Index {
  @State title: string = 'HarmonyOS 入门';

  build() {
    Column() {
      Text(this.title)
        .width('100%')

      Row() {
        Text('ArkTS')
        Text('ArkUI')
      }
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F5F5F5')
  }
}
```

这个页面展示了 `@Entry`、`@Component`、`@State`、`Column`、`Row`、`Text` 和链式属性设置之间的关系。

---

## 🔗 知识串联

```text
DevEco Studio 与 SDK
        ↓
创建 Empty Ability 项目
        ↓
在 entry 模块编写 ArkTS
        ↓
通过 @Entry + @Component 定义页面
        ↓
在 build() 中使用 ArkUI 组件描述界面
        ↓
通过 @State 驱动页面更新
```

> **一句话总结：**ArkTS 负责表达数据和业务逻辑，ArkUI 负责用声明式组件描述界面，`entry` 模块则承载应用入口、页面、资源和核心配置。

---

## ✅ 复习清单

- [ ] 安装路径、项目路径和包名中避免中文及空格
- [ ] 能使用 Empty Ability 模板创建项目并选择目标设备类型
- [ ] 能说明 `entry` 模块以及 `ets`、`resources`、`module.json5` 的作用
- [ ] 能解释 `@Entry`、`@Component`、`@State` 和 `build()`
- [ ] 能使用 ArkTS 定义变量、常量、数组、接口、对象和函数
- [ ] 能区分普通函数与箭头函数
- [ ] 能使用 `Column`、`Row` 和 `Text` 编写简单页面
- [ ] 能通过 `width`、`height`、`backgroundColor` 设置组件样式
