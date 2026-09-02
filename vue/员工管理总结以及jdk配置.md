# 员工管理总结以及jdk配置 学习笔记总结

> 本文档整理自 Day17 学习思维导图，内容分为两大板块：**Vue 前端项目实战**（员工管理增删改、登录退出、打包部署）与 **JavaSE 基础入门**（Java 概述、JDK 安装配置、HelloWorld、IDEA 使用）。

---

## 目录

**Vue 前端实战**

1. [新增员工](#一新增员工)
2. [修改员工](#二修改员工)
3. [删除员工](#三删除员工)
4. [登录与退出](#四登录与退出)
5. [打包与部署](#五打包与部署)

**JavaSE 基础入门**

6. [Java 概述](#六java-概述)
7. [JDK 下载与安装](#七jdk-下载与安装)
8. [命令行与常用 CMD 命令](#八命令行与常用-cmd-命令)
9. [Path 环境变量](#九path-环境变量)
10. [HelloWorld 程序](#十helloworld-程序)
11. [JDK / JRE / JVM 的关系](#十一jdk--jre--jvm-的关系)
12. [跨平台原理](#十二跨平台原理)
13. [JDK 环境变量配置](#十三jdk-环境变量配置)
14. [IDEA 开发工具](#十四idea-开发工具)

---

## — Vue 前端实战 —

## 一、新增员工

### 1.1 接口调用规范

1. 在 `emp.js` 中定义新增员工的方法，方法名建议以 `Api` 或 `Service` 结尾
2. **严格参照接口文档**编写请求

```js
request.post('/emps', data)
```

### 1.2 请求参数说明

| 请求方式 | 参数位置 | 查看方式 |
|----------|----------|----------|
| `POST` / `PUT` | **请求体**（Body） | F12 → 网络 → 载荷（Payload） |
| `GET` / `DELETE` | **请求行**（URL） | F12 → 网络 → 请求 URL |

### 1.3 请求体数据格式

| 数据类型 | 第二个参数写法 |
|----------|---------------|
| JSON | 传入 **JS 对象**，如 `{ name: '张三', age: 18 }` |
| Form Data | 传入 **`key=value` 格式字符串**，如 `name=张三&age=18` |

---

## 二、修改员工

### 2.1 回显数据（根据 ID 查询）

```js
// 手动处理日期数据，将 begin/end 合并为数组供日期范围选择器使用
expr.date = [expr.begin, expr.end]
```

### 2.2 提交修改

```js
request.put('/emps', data)
```

> 💡 `PUT` 请求与 `POST` 一样**有请求体**，参数在请求体中携带。

---

## 三、删除员工

### 3.1 接口特点

- **单个删除**与**批量删除**共用同一个接口

### 3.2 批量删除实现步骤

**第一步：添加复选框列**

```html
<el-table-column type="selection" />
```

**第二步：绑定 selection-change 事件**

```js
// 回调函数形参 val 表示当前选中的所有表格行
table.on('selection-change', (val) => {
  // val 为选中行的数组
})
```

**第三步：数组参与字符串拼接**

```js
// 数组拼接字符串时会自动去掉外层的 []
// 例：[1, 2, 3] + '' → '1,2,3'
const ids = selectedIds.join(',')
```

---

## 四、登录与退出

### 4.1 登录功能实现步骤

| 步骤 | 说明 |
|------|------|
| ① 引入登录页面 | `import loginPage from '...'` |
| ② 配置路由 | 登录页与 `layout` 路由**平级** |
| ③ 绑定登录按钮事件 | 向后台发送 Ajax 请求 |
| ④ 登录成功后处理 | 弹出成功提示 → 跳转首页 → 存储用户信息 |
| ⑤ 添加 token 到请求头 | 配置 axios **请求拦截器** |
| ⑥ 处理 401 响应 | 配置 axios **响应拦截器**，跳转登录页 |

**跳转页面：**

```js
import { useRouter } from 'vue-router'
const router = useRouter()
router.push('/index')
```

**存储用户信息到 localStorage：**

```js
localStorage.setItem('loginUser', JSON.stringify(result.data))
```

**axios 请求拦截器（自动携带 token）：**

```js
request.interceptors.request.use(
  function (config) {
    const loginUser = JSON.parse(localStorage.getItem('loginUser'))
    if (loginUser && loginUser.token) {
      config.headers.token = loginUser.token
    }
    return config
  },
  function (error) {
    return Promise.reject(error)
  }
)
```

**axios 响应拦截器（401 跳转登录）：**

```js
// 在 js 文件中获取路由实例
import router from '@/router/index.js'

if (error.response.status === 401) {
  router.push('/login')
}
```

### 4.2 退出功能实现步骤

1. **显示登录用户姓名**：从 `localStorage` 中取出并展示
2. **退出操作：**
   - 跳转到登录页面
   - 删除 `localStorage` 中的 `loginUser`

```js
localStorage.removeItem('loginUser')
router.push('/login')
```

---

## 五、打包与部署

### 5.1 打包

```bash
npm run build
```

> 执行后在项目**根目录**生成 `dist` 文件夹，即打包产物。

### 5.2 部署到 Nginx

**注意事项：**
- Nginx **安装路径不能包含中文**
- 开发环境中 Vite 配置的代理服务器**不会参与打包**，需在 Nginx 中重新配置代理

**Nginx 常用命令：**

| 操作     | 命令 / 方式                        |
| ------ | ------------------------------ |
| 启动     | 双击 `nginx.exe`                 |
| 停止     | `nginx.exe -s stop` 或任务管理器结束进程 |
| 重新加载配置 | `nginx.exe -s reload`          |
|        |                                |

**目录说明：**

| 目录 / 文件 | 说明 |
|-------------|------|
| `conf/nginx.conf` | Nginx 配置文件，配置代理将请求转发到后台 `8080` 端口 |
| `html/` | 存放前端打包后的静态文件（将 `dist` 内容放入此处） |

> 💡 Windows 下启动 Nginx 后，**任务管理器中会出现两个 nginx 进程**，属于正常现象。

---

## — JavaSE 基础入门 —

## 六、Java 概述

| 版本 | 全称 | 适用场景 |
|------|------|----------|
| **JavaSE** | Java Standard Edition | 基础版，桌面应用、基础开发 |
| **JavaEE** | Java Enterprise Edition | 企业版，Web / 后端开发 |
| **JavaME** | Java Micro Edition | 嵌入式、移动设备 |

> Java 是一门**计算机编程语言**，本课程从 JavaSE 基础开始学习。

---

## 七、JDK 下载与安装

- **下载**：前往 Oracle 官网，根据操作系统选择对应版本
- **安装注意**：安装路径**不能包含中文**

---

## 八、命令行与常用 CMD 命令

### 8.1 人机交互方式

| 方式 | 说明 |
|------|------|
| 图形化界面 | 鼠标点击操作 |
| **命令行** | 通过命令与计算机交互（开发常用） |

### 8.2 常用 CMD 命令

| 命令 | 说明 |
|------|------|
| `盘符:` | 切换到指定盘符，如 `D:` |
| `cd 目录` | 切换到指定目录 |
| `cd ..` | 返回上一级目录 |
| `cd .` | 当前目录（通常用于确认） |
| `dir` | 查看当前目录下的所有内容 |
| `cls` | 清屏 |
| `exit` | 退出 CMD |

### 8.3 路径斜线说明

| 符号 | 名称 | 建议 |
|------|------|------|
| `/` | 正斜线 | ✅ 建议使用 |
| `\` | 反斜线 | 部分场景使用 |

> 💡 打开 CMD 后**默认进入的目录**称为"家目录"（通常是当前用户目录）。

---

## 九、Path 环境变量

**作用：** 记录程序的执行路径，配置后可在**任意目录**下执行对应命令。

**配置路径：**

```
此电脑 → 属性 → 高级系统设置 → 环境变量 → 系统变量 → Path
```

---

## 十、HelloWorld 程序

### 10.1 开发步骤

```
① 编写代码  →  创建 .java 源文件
② 编译代码  →  javac HelloWorld.java  →  生成 .class 字节码文件
③ 运行代码  →  java HelloWorld
```

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

### 10.2 常见问题排查

| 问题 | 解决方式 |
|------|----------|
| 看不到文件扩展名 | 设置显示文件扩展名和隐藏文件 |
| 修改代码未生效 | 检查是否保存（`Ctrl + S`） |
| 编译报错 | 检查英文单词拼写及是否误用中文标点 |
| 找不到类 | 确认**文件名与类名完全一致** |
| 命令执行失败 | 检查编译与运行命令是否正确 |

### 10.3 相关概念

| 概念 | 说明 |
|------|------|
| **Bug** | 程序中的问题或漏洞 |
| **Debug** | 调试，查找并修复程序漏洞的过程 |

---

## 十一、JDK / JRE / JVM 的关系

```
JDK
 └── JRE
      └── JVM（Java 虚拟机，程序真正运行的地方）
      └── 核心类库（官方写好的 Java 类，可直接使用）
 └── 开发工具（javac、java 等）
```

| 名称 | 全称 | 说明 |
|------|------|------|
| **JVM** | Java Virtual Machine | Java 虚拟机，程序真正运行的地方 |
| **JRE** | Java Runtime Environment | Java 运行环境 = JVM + 核心类库 |
| **JDK** | Java Development Kit | Java 开发环境 = JRE + 开发工具 |

---

## 十二、跨平台原理

**核心理念：一次编译，到处运行**

```
HelloWorld.java
    ↓ javac 编译
HelloWorld.class（字节码文件）
    ↓ 不同 JVM 翻译
Windows / macOS / Linux（运行效果相同）
```

**实现原理：** JVM 在字节码文件与操作系统之间充当**翻译角色**，将相同的字节码翻译成各操作系统对应的机器码，从而实现跨平台。

---

## 十三、JDK 环境变量配置

> 建议**手动配置**环境变量，避免自动安装导致的路径问题。

**配置步骤：**

```
1. 在系统变量中新建变量：
   变量名：JAVA_HOME
   变量值：JDK 安装路径（如 C:\Program Files\Java\jdk-17）

2. 修改 Path 变量，新增：
   %JAVA_HOME%\bin
```

---

## 十四、IDEA 开发工具

### 14.1 下载与安装

- 从官网下载**专业版（Ultimate）**
- 安装后进行**破解**（学习使用）

### 14.2 代码管理层次结构

```
Project（项目）
 └── Module（模块）
      └── Package（包）
           └── Class（类）
```

### 14.3 常用配置

| 配置项 | 说明 |
|--------|------|
| 设置忽略大小写提示 | 代码补全时不区分大小写 |
| 修改字符集为 UTF-8 | 防止中文乱码 |

### 14.4 常用快捷键

| 快捷键 / 缩写 | 功能 |
|---------------|------|
| `sout` + 回车 | 生成 `System.out.println()` 打印语句 |
| `main` + 回车 | 生成 `main` 主方法 |
| `Ctrl + D` | 向下复制一行 |
| `Ctrl + Shift + ↑/↓` | 上下移动代码 |
| `Ctrl + X` | 删除当前行代码 |
| `Ctrl + Alt + L` | 格式化代码 |

---

*整理完毕 · Day17 · Vue 前端实战 + JavaSE 基础入门*
