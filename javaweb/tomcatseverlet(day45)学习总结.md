# Java Web 学习总结（Day 45）

> 本节核心内容：**HTTP 协议、Tomcat Web 服务器、Servlet 基础**。  
> 学习主线：浏览器发送 HTTP 请求 → Tomcat 接收请求 → Servlet 处理请求 → 服务器返回 HTTP 响应。

## 目录

- [一、HTTP 协议](#一http-协议)
- [二、Tomcat 服务器](#二tomcat-服务器)
- [三、Servlet 基础](#三servlet-基础)
- [四、Servlet 执行流程](#四servlet-执行流程)
- [五、Servlet 生命周期](#五servlet-生命周期)
- [六、Servlet 体系结构](#六servlet-体系结构)
- [七、Servlet URL 匹配](#七servlet-url-匹配)
- [八、开发环境补充配置](#八开发环境补充配置)
- [九、重点速记](#九重点速记)

---

## 一、HTTP 协议

### 1. HTTP 概述

HTTP（HyperText Transfer Protocol，超文本传输协议）是应用层协议，通常基于 TCP 实现。

主要特点：

- **请求—响应模型**：一次请求对应一次响应。
- **无状态**：默认情况下，不同请求彼此独立，无法直接共享数据。
- **可靠传输**：HTTP 基于 TCP 时，由 TCP 提供可靠的数据传输能力。

HTTP 通信过程：

```text
客户端（浏览器） ── HTTP 请求 ──> Web 服务器
客户端（浏览器） <── HTTP 响应 ── Web 服务器
```

### 2. HTTP 请求报文

HTTP 请求报文主要由四部分组成：

```text
请求行
请求头
空行
请求体
```

#### 2.1 请求行

基本格式：

```text
请求方法 请求路径 HTTP协议版本
```

示例：

```http
GET /users?id=1 HTTP/1.1
```

常见请求方法：

| 请求方法 | 常见用途 |
|---|---|
| `GET` | 查询资源 |
| `POST` | 新增资源、提交数据 |
| `PUT` | 更新资源 |
| `DELETE` | 删除资源 |

#### 2.2 请求头

请求头以 `名称: 值` 的形式传递附加信息：

```http
Content-Type: application/json
```

常见请求数据格式：

| 数据格式 | Content-Type | 说明 |
|---|---|---|
| 普通表单 | `application/x-www-form-urlencoded` | 数据形如 `key=value&key=value` |
| JSON | `application/json` | 用 JSON 表示结构化数据 |
| 文件表单 | `multipart/form-data` | 常用于文件上传 |

#### 2.3 JSON 格式

JSON 的基本规则：

- 键必须使用双引号包裹。
- 数值、布尔值直接书写。
- 字符串使用双引号包裹。
- 数组使用 `[]`。
- 对象使用 `{}`。
- 空值写作 `null`。

示例：

```json
{
  "id": 1,
  "name": "张三",
  "enabled": true,
  "skills": ["Java", "Maven"],
  "address": {
    "city": "北京"
  },
  "remark": null
}
```

#### 2.4 请求体与请求参数

- `GET`、`DELETE` 的参数通常放在 URL 查询字符串中。
- `POST`、`PUT` 的参数通常放在请求体中，也可以通过 URL 传递查询参数。
- 请求体的数据格式由 `Content-Type` 描述。

> 严格来说，HTTP 规范并未绝对禁止某些请求方法携带请求体；实际开发应遵循接口约定和服务器框架的处理方式。

### 3. HTTP 响应报文

HTTP 响应报文主要由四部分组成：

```text
状态行
响应头
空行
响应体
```

#### 3.1 状态行

基本格式：

```text
HTTP协议版本 状态码 状态描述
```

示例：

```http
HTTP/1.1 200 OK
```

常见状态码：

| 状态码 | 含义 |
|:---:|---|
| `200` | 请求成功 |
| `302` | 临时重定向到其他资源 |
| `304` | 资源未修改，可继续使用浏览器缓存 |
| `401` | 未认证或认证失败 |
| `403` | 已识别请求，但权限不足 |
| `404` | 请求的资源不存在 |
| `405` | 当前资源不支持该请求方法 |
| `500` | 服务器内部错误 |

#### 3.2 响应头与响应体

响应头同样采用 `名称: 值` 的形式。

`Content-Type` 用于描述响应体格式，例如：

```http
Content-Type: application/json
Content-Type: text/html
```

响应体中存放服务器实际返回的数据，如 JSON、HTML、文本或文件内容。

---

## 二、Tomcat 服务器

### 1. Tomcat 概述

Tomcat 是支持 Servlet 规范的 Web 服务器，同时也是一个 **Servlet 容器**。

它的主要职责包括：

- 接收客户端 HTTP 请求。
- 根据请求路径找到对应的 Web 项目和 Servlet。
- 创建、管理 Servlet 对象并调用其方法。
- 将处理结果封装成 HTTP 响应并返回客户端。

### 2. 启动与关闭

在 Windows 中，可进入 Tomcat 的 `bin` 目录：

```text
启动：startup.bat
关闭：在运行窗口按 Ctrl + C，或使用 shutdown.bat
```

### 3. 修改端口

Tomcat 默认端口通常为 `8080`，配置文件位置：

```text
Tomcat安装目录/conf/server.xml
```

HTTP 的默认端口是 `80`；访问 `8080` 端口时需要在 URL 中明确写出端口号：

```text
http://localhost:8080
```

### 4. 部署 Web 项目

传统 Java Web 项目通常打成 `war` 包，然后将其放入：

```text
Tomcat安装目录/webapps
```

Tomcat 会部署该应用。

#### 上下文路径

上下文路径（Context Path）用于区分同一个 Tomcat 中部署的不同 Web 应用，通常与应用目录名或 WAR 包名对应。

例如：

```text
WAR 包：servlet-demo.war
上下文路径：/servlet-demo
```

`ROOT` 是特殊应用名，表示根应用，访问时不需要额外的上下文路径。

### 5. IDEA 创建 Maven Web 项目

创建项目时可选择以 `webapp` 结尾的 Maven Archetype，它会帮助生成 `webapp` 目录。

同时应将项目打包方式设置为：

```xml
<packaging>war</packaging>
```

### 6. IDEA 集成本地 Tomcat

基本步骤：

1. 打开 **Run/Debug Configurations**。
2. 新建 **Tomcat Server → Local**。
3. 在 **Application server** 中配置本地 Tomcat 安装目录。
4. 切换到 **Deployment** 页面。
5. 点击 `+`，添加项目的 `war exploded` Artifact。
6. 设置 **Application context**（上下文路径）。
7. 保存配置并启动 Tomcat。

> 可通过 Tomcat Server 的配置模板统一设置后续运行配置。

---

## 三、Servlet 基础

### 1. Servlet 是什么

Servlet 是运行在 Web 服务器中的 Java 程序，主要用于处理 HTTP 请求和响应。

Servlet 具有以下特点：

- 只能在 Servlet 容器中运行。
- 由 Web 服务器创建和管理。
- 负责接收请求、处理业务并生成响应。
- 是 Java Web 三大组件之一。

### 2. Servlet 快速入门

#### 第一步：引入 Servlet API

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>
```

`scope` 使用 `provided`，表示编译、测试时需要该依赖，但部署运行时由 Tomcat 提供，因此不打入 WAR 包。

#### 第二步：定义 Servlet

入门阶段可以定义类实现 `Servlet` 接口，并重写接口中的方法。

实际开发中更常见的方式是继承 `HttpServlet`：

```java
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/demo1")
public class DemoServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {
        response.getWriter().write("Hello Servlet");
    }
}
```

#### 第三步：配置访问路径

使用 `@WebServlet` 指定 `urlPatterns`：

```java
@WebServlet("/demo1")
```

URL Pattern 必须以 `/` 开头。

---

## 四、Servlet 执行流程

以如下地址为例：

```text
http://localhost:8080/servlet-demo/demo1
```

地址可拆分为：

| URL 部分 | 作用 |
|---|---|
| `http://` | 使用 HTTP 协议 |
| `localhost` / `127.0.0.1` | 定位本机 |
| `8080` | 定位 Tomcat 服务 |
| `/servlet-demo` | 定位 Tomcat 中的 Web 项目 |
| `/demo1` | 定位项目中的 Servlet |

完整执行过程：

1. 浏览器向 `localhost:8080` 发送 HTTP 请求。
2. Tomcat 根据 `/servlet-demo` 找到对应 Web 应用。
3. 容器根据 `/demo1` 找到匹配的 Servlet。
4. Tomcat 创建或获取 Servlet 对象。
5. Tomcat 调用 Servlet 的 `service()` 方法处理请求。
6. `HttpServlet` 根据请求方法继续调用 `doGet()`、`doPost()` 等方法。
7. Tomcat 将处理结果封装成 HTTP 响应并返回浏览器。

> Servlet 对象由 Web 服务器创建，Servlet 方法也由 Web 服务器调用。

---

## 五、Servlet 生命周期

Servlet 生命周期由容器管理，可分为四个阶段：

| 阶段 | 方法/动作 | 调用次数 |
|---|---|:---:|
| 创建对象 | 容器实例化 Servlet | 通常一次 |
| 初始化 | `init()` | 一次 |
| 提供服务 | `service()` | 多次 |
| 销毁 | `destroy()` | 一次 |

### 1. 创建对象

默认情况下，Servlet 通常在**第一次被访问时**创建。

通过设置 `loadOnStartup` 为 `0` 或正整数，可以让服务器启动时就创建 Servlet：

```java
@WebServlet(urlPatterns = "/demo1", loadOnStartup = 1)
```

默认值为 `-1`，表示按需加载。

### 2. 初始化

Servlet 创建后，容器调用 `init()` 完成初始化，该方法只调用一次。

### 3. 对外提供服务

每次收到匹配的请求时，容器调用 `service()`，因此该方法会被调用多次。

### 4. 销毁

服务器正常关闭或应用被卸载时，容器调用 `destroy()`，该方法通常只调用一次。

> `service()` 可能被多个请求并发调用，开发时应避免把请求专属数据放在 Servlet 实例变量中。

---

## 六、Servlet 体系结构

```text
Servlet 接口
    └─ GenericServlet 抽象类
         └─ HttpServlet 抽象类
              └─ 自定义 Servlet
```

### 1. Servlet 接口

定义 Servlet 生命周期及服务能力，包含 `init()`、`service()`、`destroy()` 等方法。

### 2. GenericServlet

实现了 Servlet 接口的大部分通用方法，开发者主要关注 `service()`。

### 3. HttpServlet

专门处理 HTTP 请求。

其内部服务流程可以理解为：

1. 将通用请求、响应对象转换为 HTTP 类型。
2. 根据请求方法分发到对应的 `doXxx()` 方法：

   - `GET` → `doGet()`
   - `POST` → `doPost()`
   - `PUT` → `doPut()`
   - `DELETE` → `doDelete()`

因此，后续开发 Servlet 时通常：

```text
继承 HttpServlet → 重写对应的 doXxx() 方法
```

---

## 七、Servlet URL 匹配

`urlPatterns` 是数组，因此一个 Servlet 可以配置多个访问路径：

```java
@WebServlet(urlPatterns = {"/demo1", "/demo2"})
```

常见匹配方式：

| 匹配方式 | 示例 |
|---|---|
| 精确匹配 | `/demo1` |
| 目录匹配 | `/user/*` |
| 扩展名匹配 | `*.do` |
| 默认匹配 | `/` |

通常应优先使用含义明确的精确路径。精确匹配的优先级高于默认匹配。

---

## 八、开发环境补充配置

### 1. 设置 Maven 使用 JDK 17

可以在 Maven 的 `conf/settings.xml` 中，将以下配置加入 `<profiles>`：

```xml
<profile>
    <id>jdk17</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>17</jdk>
    </activation>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <maven.compiler.compilerVersion>17</maven.compiler.compilerVersion>
        <encoding>UTF-8</encoding>
    </properties>
</profile>
```

项目级开发中，也可以将编译版本直接配置在项目的 `pom.xml` 中，使项目配置更容易共享。

### 2. IDEA 自动下载 Maven 源码

进入：

```text
Settings → Build, Execution, Deployment
         → Build Tools → Maven → Importing
```

勾选：

- **Sources**
- **Documentation**

IDEA 会在导入 Maven 项目时尝试下载依赖源码和文档，便于查看源码与注释。

### 3. Maven 镜像地址

配置 Maven 镜像时应优先使用 HTTPS 地址，例如：

```xml
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>https://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

---

## 九、重点速记

> [!TIP]
> **请求路径决定“找谁处理”，请求方法决定“调用哪个处理方法”。**

- HTTP 是应用层、请求—响应式、无状态协议。
- 请求报文：请求行、请求头、空行、请求体。
- 响应报文：状态行、响应头、空行、响应体。
- `Content-Type` 用于说明请求体或响应体的数据格式。
- Tomcat 既是 Web 服务器，也是 Servlet 容器。
- Web 项目通常打成 WAR 包并部署到 Tomcat。
- Servlet 对象由容器创建，方法由容器调用。
- Servlet 生命周期：创建 → `init()` → `service()` → `destroy()`。
- `init()`、`destroy()` 通常各调用一次，`service()` 会调用多次。
- 实际开发通常继承 `HttpServlet`，重写 `doGet()`、`doPost()` 等方法。
- `@WebServlet` 的路径必须以 `/` 开头。
- Servlet API 使用 `provided` 范围，因为运行时由 Tomcat 提供。

---

> 本文根据思维导图 **《day45》** 整理，对原有知识点进行了结构化归纳，并将截图中的配置和执行流程转换为文字步骤。
