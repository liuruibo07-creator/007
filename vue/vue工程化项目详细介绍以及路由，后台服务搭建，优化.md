# Vue 工程化开发总结 · Day13 ~ Day15

> 本文档整理自三份学习思维导图，内容涵盖 **async/await 异步语法**、**Vue 工程化入门**、**项目目录结构与加载流程**、**API 风格对比**、**Element Plus 组件库**、**Vue Router 路由**、**后台服务搭建** 及**项目优化**等核心知识点。

---

## 目录

**Day 13 · Vue 工程化入门**

1. [async / await](#一async--await)
2. [Vue 工程化概述](#二vue-工程化概述)
3. [Node.js 安装（nvm 方式）](#三nodejs-安装nvm-方式)
4. [创建 Vue 项目](#四创建-vue-项目)

**Day 14 · 项目结构 & Element Plus**

5. [Vue 项目目录结构](#五vue-项目目录结构)
6. [Vue 项目加载流程](#六vue-项目加载流程)
7. [API 风格：选项式 vs 组合式](#七api-风格选项式-vs-组合式)
8. [Element Plus 整合](#八element-plus-整合)

**Day 15 · 路由 & 后台 & 优化**

9. [前后端分离开发](#九前后端分离开发)
10. [页面布局（Element Plus）](#十页面布局element-plus)
11. [Vue Router 路由](#十一vue-router-路由)
12. [后台服务搭建](#十二后台服务搭建)
13. [部门管理与项目优化](#十三部门管理与项目优化)
14. [跨域问题](#十四跨域问题)

---

## 一、async / await

> **Day 13**

### 作用

将**异步操作变为同步写法**，让代码逻辑更清晰、更易读。

### 使用方式

```js
// async 修饰方法，表示该方法是异步方法
async function getData() {
    // await 加在 ajax 异步请求前，替代 .then()
    const result = await axios.get('/api/emps/list')
    console.log(result.data)
}
```

| 关键字 | 位置 | 作用 |
|--------|------|------|
| `async` | 修饰函数 | 声明当前函数为异步函数 |
| `await` | 放在异步请求前 | 等待结果返回，替代 `.then()` 链式调用 |

---

## 二、Vue 工程化概述

> **Day 13**

**Vue 工程化** 是使用 Vue 框架开发企业级前端项目的开发模式，具有以下四大特征：

| 特征 | 说明 |
|------|------|
| **模块化** | 将代码拆分成独立模块，按需引用 |
| **组件化** | 将 UI 拆分成可复用的组件 |
| **规范化** | 统一的目录结构与开发规范 |
| **自动化** | 自动构建、打包、热更新 |

---

## 三、Node.js 安装（nvm 方式）

> **Day 13**

**nvm**（Node Version Manager）是管理 Node.js 版本的工具，推荐通过它安装 Node.js。

### 安装步骤

```
① 检查电脑上是否有 Node.js，如有则先卸载
② 双击安装 nvm，设置 nvm 安装路径和 Node.js 安装路径（必须是空目录）
③ 修改 settings.txt，配置 Node.js 和 npm 的加速镜像
④ 以管理员身份打开 CMD，配置 npm 淘宝镜像加速
⑤ 通过 nvm 安装 Node.js：nvm install 20.19.5
⑥ 切换版本：nvm use 20.19.5
⑦ 验证安装：node -v  和  npm -v
```

---

## 四、创建 Vue 项目

> **Day 13**

### 步骤

```bash
# 第一步：安装并执行脚手架 create-vue
npm create vue@latest

# 第二步：按脚手架提示定制项目（入门案例全选 No）

# 第三步：用 VSCode 打开项目文件夹

# 第四步：安装所有依赖
npm install

# 第五步：运行项目
npm run dev

# 第六步：浏览器访问
# http://localhost:5173
```

### 常见问题排查

| 问题 | 解决方案 |
|------|----------|
| 打开的不是项目文件夹，`package.json` 无法识别 | 重新用 VSCode 打开正确的项目根目录 |
| 没有管理员权限报错 | 以**管理员身份**打开 VSCode |
| 集成终端执行命令失败 | 将终端类型从 **PowerShell 改为 CMD** |

---

## 五、Vue 项目目录结构

> **Day 14**

```
vue-project/
├── index.html              # SPA 单页应用的入口文件（localhost:5173 默认访问此文件）
├── vite.config.js          # Vite 构建工具的配置文件
├── package.json            # 项目包文件：基本信息、dependencies（生产依赖）、scripts（脚本）
├── package-lock.json       # 锁定依赖的具体版本
├── node_modules/           # 所有依赖包的存放位置
├── public/                 # 公共静态资源（打包时原样复制进去）
└── src/                    # 源代码目录
    ├── main.js             # 入口 JS 文件
    ├── App.vue             # 根组件（SFC 单文件组件）
    ├── components/         # 通用组件（如文件上传组件）
    ├── views/              # 页面级自定义组件
    └── assets/             # 静态资源（打包时会被处理，非原样复制）
```

### package.json 与 package-lock.json 的区别

| 文件 | 依赖版本 | 说明 |
|------|----------|------|
| `package.json` | **范围版本**（如 `^4.0.0`） | 记录项目基本信息与依赖范围 |
| `package-lock.json` | **精确版本** | 锁定实际安装的版本，保证团队环境一致 |

### App.vue 单文件组件（SFC）结构

```vue
<script>    <!-- 脚本区域：定义数据、方法、钩子函数 -->
</script>

<template>  <!-- 模板区域：定义 HTML 结构 -->
</template>

<style>     <!-- 样式区域：定义 CSS 样式 -->
</style>
```

---

## 六、Vue 项目加载流程

> **Day 14**

```
① 浏览器访问 localhost:5173
        ↓
② 加载 index.html
   └── 包含 id="app" 的 <div>
   └── <script> 引入 main.js
        ↓
③ 加载 main.js
   └── 创建 Vue 应用实例
   └── 导入 App.vue 根组件
   └── 将根组件渲染到 id="app" 的 div 中
        ↓
④ 页面呈现效果
```

---

## 七、API 风格：选项式 vs 组合式

> **Day 14**

Vue 支持两种 API 风格：

### 选项式 API（Options API）— Vue 2 采用

```vue
<script>
export default {
  data() {
    return { /* 数据 */ }
  },
  methods: { /* 方法 */ },
  mounted() { /* 钩子函数 */ }
}
</script>
```

### 组合式 API（Composition API）— Vue 3 采用 ✅ 推荐

```vue
<script setup>
import { ref, onMounted } from 'vue'

const userList = ref([])   // ref 可存放基本类型和引用类型

function search() { /* 方法 */ }

onMounted(() => {           // 钩子函数
  search()
})
</script>
```

### 两种风格对比

| 对比项 | 选项式 API | 组合式 API |
|--------|------------|------------|
| Vue 版本 | Vue 2 为主 | Vue 3 推荐 |
| 代码组织 | **分散**（按选项类型区分） | **集中**（按功能逻辑聚合） |
| 访问数据 | 通过 `this.xxx` | 通过 `变量名.value` |
| script 标签 | 不加 `setup` | **必须加 `setup`** |
| setup 本质 | — | 是一个语法糖，等价于 `export default { setup() { return {...} } }` |

---

## 八、Element Plus 整合

> **Day 14**

### 8.1 什么是 Element Plus

基于 **Vue 3** 的 UI 组件库，提供开箱即用的美观组件。

### 8.2 整合步骤

#### 第一步：安装

```bash
npm install element-plus --save
```

> 💡 **`--save` / `-S`**：生产依赖，打包时会带入（默认行为）
> **`--save-dev` / `-D`**：开发依赖，打包时不带入

#### 第二步：在 main.js 中引入

```js
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'

app.use(ElementPlus)
```

### 8.3 常用组件速查

| 功能 | 组件标签 |
|------|----------|
| **表格** | `<el-table>`、`<el-table-column>` |
| **分页条** | `<el-pagination>` |
| **对话框** | `<el-dialog>` |

### 8.4 使用组件的步骤

```
① 查看原型图，确定需要使用哪种组件
② 前往 Element Plus 官网找到匹配的组件
③ 复制示例代码到 Vue 组件中，查看效果
④ 调整属性和样式，满足实际需求
```

---

## 九、前后端分离开发

> **Day 15**

### 概念

将**前端代码与后端代码分开编写**的开发模式，双方通过接口文档对接。

### 开发流程

```
需求分析 → 系统分析（定义接口文档）→ 前后端并行开发 → 各自自测 → 前后端联调
```

### 接口四要素

| 要素 | 说明 |
|------|------|
| **请求方式** | `GET`（查询）、`POST`（新增）、`DELETE`（删除）、`PUT`（修改） |
| **请求路径** | 接口的 URL 地址 |
| **请求参数** | 请求携带的数据 |
| **响应结果** | 后端返回的数据格式 |

---

## 十、页面布局（Element Plus）

> **Day 15**

### Container 布局

```html
<el-container>
  <el-header>顶部</el-header>
  <el-container>
    <el-aside>侧边栏</el-aside>
    <el-main>主内容区</el-main>
  </el-container>
  <el-footer>底部</el-footer>
</el-container>
```

> 💡 **布局规律：** `el-container` 中若**包含** `el-header` 或 `el-footer` → **垂直布局**；若**不包含** → **水平布局**

### 菜单组件

| 组件 | 作用 |
|------|------|
| `<el-menu>` | 菜单根容器 |
| `<el-sub-menu>` | 子菜单（可展开/折叠） |
| `<el-menu-item>` | 菜单项（可点击跳转） |

> 💡 `<el-menu>` 开启 **`router` 属性**后，`index` 属性值等同于路由 `path`，点击菜单项自动跳转。

---

## 十一、Vue Router 路由

> **Day 15**

### 11.1 什么是路由

**路由**是 **URL 路径与页面组件的对应关系**。

### 11.2 集成步骤

#### 第一步：安装

```bash
npm install vue-router@4
```

#### 第二步：创建路由实例（`src/router/index.js`）

```js
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
    // history 模式：web（不带 #，Vue 3 推荐）
    history: createWebHistory(),

    // 路由表
    routes: [
        {
            path: '/login',
            component: () => import('../views/LoginPage.vue')
        },
        {
            path: '/',
            component: () => import('../views/Layout.vue'),
            // 嵌套路由（子路由）
            children: [
                {
                    path: '/dept',
                    component: () => import('../views/DeptPage.vue')
                },
                {
                    path: '/emp',
                    component: () => import('../views/EmpPage.vue')
                }
            ]
        }
    ]
})

export default router
```

#### 第三步：在 `main.js` 中导入

```js
import router from './router/index.js'
app.use(router)
```

### 11.3 路由三大核心

| 组成 | 说明 |
|------|------|
| **路由实例** | 通过 `createRouter()` 创建，包含路由表和 history 模式 |
| **`<router-view>`** | 视图组件，渲染与当前路径对应的页面组件（**只渲染直接子组件**） |
| **`<router-link>`** | 链接组件，通过 `to` 属性指定跳转路径 |

### 11.4 History 模式对比

| 模式 | 路径特征 | 适用版本 |
|------|----------|----------|
| **Hash 模式** | 路径中带 `#`，如 `/#/dept` | Vue 2 较多使用 |
| **Web 模式** | 路径中不带 `#`，如 `/dept` | Vue 3 推荐 ✅ |

### 11.5 嵌套路由

> 通过 `children` 属性实现父子路由嵌套，父组件用 `<router-view>` 渲染子页面。

```
App.vue 的 <router-view>  →  渲染 Layout 组件
Layout.vue 的 <router-view>  →  渲染 DeptPage / EmpPage 组件
```

---

## 十二、后台服务搭建

> **Day 15**

### 12.1 JDK 17 安装

```
① 下载 JDK 17 安装包，按向导安装
② 配置环境变量：
   - 系统变量新建：JAVA_HOME = JDK 安装路径
   - 修改 Path，添加：%JAVA_HOME%\bin
③ 验证：cmd 中运行 java -version 和 javac -version
```

**注意事项：**

| 注意点 | 说明 |
|--------|------|
| 高版本 JDK 自动配置的 Path | 目录下命令少，建议**删除后手动配置** |
| 不建议配置 `classpath` | — |
| 建议配置在**系统变量**中 | 对所有用户生效 |
| CMD 命令查找顺序 | ① 当前目录 → ② Path 环境变量从上到下 → ③ 提示找不到命令 |
| 大小写区分 | Windows **不区分**大小写；Linux **区分**大小写 |

---

### 12.2 MySQL 8 绿色版安装

**安装版 vs 绿色版区别：**

| 类型 | 格式 | 特点 |
|------|------|------|
| 安装版 | `.exe` / `.msi` | 有安装向导，控制面板可见，需从控制面板卸载 |
| 绿色版 | `.zip` | 解压即安装，控制面板不可见，直接删除文件夹卸载 |

**绿色版配置步骤：**

```bash
# 1. 配置环境变量
#    新建 MYSQL_HOME = MySQL 解压路径
#    修改 Path，添加 %MYSQL_HOME%\bin

# 2. 初始化数据（在 MySQL 目录下执行）
mysqld --initialize-insecure
# → 在安装目录创建 data 文件夹，存储数据库和表信息

# 3. 注册 MySQL 服务
mysqld -install
# → 在系统服务中注册名为 mysql 的服务

# 4. 启动 MySQL 服务
net start mysql

# 5. 修改 root 密码
mysqladmin -u root password 1234
```

---

### 12.3 启动 Spring Boot 项目

```bash
java -jar tlias.jar
```

---

## 十三、部门管理与项目优化

> **Day 15**

### 优化一：封装 axios 工具类（`request.js`）

```js
import axios from 'axios'

const request = axios.create({
    baseURL: 'http://localhost:8080'   // 统一配置请求基础路径
})

// 添加响应拦截器（统一处理响应数据）
request.interceptors.response.use(
    response => response.data,
    error => Promise.reject(error)
)

export default request
```

**优化效果：**

| 优化前 | 优化后 |
|--------|--------|
| 每个请求都要写完整 URL | 只写接口路径，`baseURL` 统一管理 |
| 每次手动取 `response.data` | 响应拦截器自动处理 |

### 优化二：按模块封装 API 文件

将每个业务模块的所有 axios 请求封装到独立的 JS 文件中：

```
src/
└── api/
    ├── dept.js    # 部门相关接口
    └── emp.js     # 员工相关接口
```

```js
// dept.js 示例
import request from '@/utils/request.js'

export const getDeptList = () => request.get('/depts')
export const deleteDept = (id) => request.delete(`/depts/${id}`)
```

---

## 十四、跨域问题

> **Day 15**

### 什么是跨域

违反了浏览器的**同源策略**（协议、域名、端口三者有一不同即为跨域）。

### 解决方案：代理服务器

在 `vite.config.js` 中配置代理，将跨域请求转发给后端服务器。

```js
// vite.config.js
export default {
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080',  // 后端地址
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
}
```

**原理：**

```
浏览器 → 代理服务器（同域） → 后端服务器
        （前端与代理同源）    （服务器之间无跨域限制）
```

> ✅ 服务器与服务器之间通信**没有跨域限制**，代理服务器作为中间层绕过浏览器的同源策略。


