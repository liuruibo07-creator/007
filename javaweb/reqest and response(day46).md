# Servlet Request 与 Response 学习总结（Day 46）

> 本节核心：使用 `HttpServletRequest` 获取客户端请求信息，使用 `HttpServletResponse` 构造服务器响应，并掌握**请求转发、响应重定向、编码处理及常见数据响应方式**。

## 目录

- [一、Request 与 Response 概述](#一request-与-response-概述)
- [二、Request：获取请求报文](#二request获取请求报文)
- [三、获取请求参数](#三获取请求参数)
- [四、请求转发](#四请求转发)
- [五、请求参数乱码](#五请求参数乱码)
- [六、Response：设置响应报文](#六response设置响应报文)
- [七、响应重定向](#七响应重定向)
- [八、转发与重定向对比](#八转发与重定向对比)
- [九、路径书写规则](#九路径书写规则)
- [十、常见响应类型](#十常见响应类型)
- [十一、重点速记](#十一重点速记)

---

## 一、Request 与 Response 概述

在 Servlet 中，Tomcat 会把 HTTP 请求和响应封装为两个对象：

| 对象 | 作用 |
|---|---|
| `HttpServletRequest` | 获取请求行、请求头、请求参数和请求体等信息 |
| `HttpServletResponse` | 设置状态码、响应头和响应体，并向客户端返回数据 |

典型处理流程：

```text
浏览器发送请求
      ↓
Tomcat 创建 Request、Response 对象
      ↓
调用 Servlet 的 service()/doXxx() 方法
      ↓
Servlet 读取 Request，处理业务，写入 Response
      ↓
Tomcat 将响应返回浏览器
```
![](assets/reqest%20and%20response(day46)/file-20260802124517198.png)
---

## 二、Request：获取请求报文

### 1. 获取请求行信息

假设客户端访问：

```text
http://localhost:8080/web-demo/user?id=10
```

常用方法如下：

| 目标 | 方法 | 示例结果 |
|---|---|---|
| 请求方式 | `getMethod()` | `GET` |
| 上下文路径 | `getContextPath()` | `/web-demo` |
| 完整请求 URL | `getRequestURL()` | `http://localhost:8080/web-demo/user` |
| 请求 URI | `getRequestURI()` | `/web-demo/user` |
| 查询字符串 | `getQueryString()` | `id=10` |

示例：

```java
String method = request.getMethod();
String contextPath = request.getContextPath();
StringBuffer requestURL = request.getRequestURL();
String requestURI = request.getRequestURI();
String queryString = request.getQueryString();
```

> `URL` 包含协议、主机和端口；`URI` 不包含这些内容，只表示服务器中的资源路径。

### 2. 获取请求头

通过请求头名称获取对应值：

```java
String userAgent = request.getHeader("User-Agent");
String contentType = request.getHeader("Content-Type");
```

方法格式：

```java
request.getHeader("请求头名称");
```

### 3. 获取请求体

请求体可通过 I/O 流读取。

#### 字符流

适合读取文本、JSON 等字符数据：

```java
BufferedReader reader = request.getReader();
String line;
StringBuilder body = new StringBuilder();

while ((line = reader.readLine()) != null) {
    body.append(line);
}
```

#### 字节流

适合读取图片、文件等二进制数据：

```java
ServletInputStream inputStream = request.getInputStream();
```

| 方法 | 流类型 | 常见用途 |
|---|---|---|
| `getReader()` | 字符流（`BufferedReader`） | 文本、JSON |
| `getInputStream()` | 字节流（`ServletInputStream`） | 文件、图片、二进制数据 |

> 同一次请求中，不应同时使用 `getReader()` 和 `getInputStream()` 读取请求体。

---

## 三、获取请求参数

### 1. 获取单个参数值

```java
String username = request.getParameter("username");
```

对应请求数据：

```text
username=zhangsan
```

### 2. 获取同名的多个参数值

常用于复选框或一个参数对应多个值的场景：

```java
String[] hobbies = request.getParameterValues("hobby");
```

对应请求数据：

```text
hobby=java&hobby=music&hobby=sport
```

### 3. 适用的数据格式

参数 API 主要用于解析查询字符串和表单格式数据：

```text
key=value&key=value
```

例如：

- URL 查询参数
- `application/x-www-form-urlencoded` 表单数据

JSON 请求体不能直接通过 `getParameter()` 得到完整对象，应通过 `getReader()` 读取，再使用 JSON 工具转换。

---

## 四、请求转发

请求转发（Forward）是服务器在应用内部将当前请求交给另一个资源继续处理。

### 1. 基本写法

```java
request.getRequestDispatcher("/target")
       .forward(request, response);
```

转发路径是当前应用内部路径，通常不需要添加项目上下文路径。

### 2. 转发共享数据

转发前可以把数据存入 Request 域：

```java
request.setAttribute("message", "处理成功");
request.getRequestDispatcher("/target")
       .forward(request, response);
```

目标资源获取数据：

```java
Object message = request.getAttribute("message");
```

### 3. 转发特点

- 只能转发到当前 Web 应用内部的资源。
- 整个过程只有一次请求。
- 转发前后使用同一个 Request，可以共享请求域数据。
- 浏览器地址栏不会改变。
- 转发由服务器内部完成，浏览器不知道转发过程。

---

## 五、请求参数乱码

### 1. GET 请求

Tomcat 8 及以上版本通常已经使用 UTF-8 处理 URL 参数，一般不需要额外编写乱码处理代码。

### 2. POST 请求

对于使用请求体提交的表单参数，应在第一次读取参数之前设置请求字符编码：

```java
request.setCharacterEncoding("UTF-8");
String username = request.getParameter("username");
```

关键顺序：

```text
先设置字符编码 → 再读取请求参数
```

> Tomcat 10 对 UTF-8 的默认支持更完善；为兼容不同运行环境，处理传统表单 POST 请求时仍可明确设置编码。

---

## 六、Response：设置响应报文

### 1. 设置响应状态码

```java
response.setStatus(200);
```

更推荐使用状态码常量：

```java
response.setStatus(HttpServletResponse.SC_OK);
response.setStatus(HttpServletResponse.SC_NOT_FOUND);
```

### 2. 设置响应头

```java
response.setHeader("响应头名称", "响应头值");
```

示例：

```java
response.setHeader("Cache-Control", "no-cache");
```

### 3. 设置响应体

#### 字符流

适合输出文字、HTML、JSON：

```java
PrintWriter writer = response.getWriter();
writer.write("Hello Servlet");
```

#### 字节流

适合输出图片、文件等二进制内容：

```java
ServletOutputStream outputStream = response.getOutputStream();
```

| 方法 | 流类型 | 常见用途 |
|---|---|---|
| `getWriter()` | 字符流（`PrintWriter`） | 文本、HTML、JSON |
| `getOutputStream()` | 字节流（`ServletOutputStream`） | 图片、音频、文件下载 |

> 同一次响应中，不应同时使用 `getWriter()` 和 `getOutputStream()`。

---

## 七、响应重定向

响应重定向（Redirect）是服务器通知浏览器重新访问另一个地址。

### 1. 基本写法

重定向到当前应用中的资源：

```java
response.sendRedirect(
        request.getContextPath() + "/resp3"
);
```

也可以重定向到外部网站：

```java
response.sendRedirect("https://example.com");
```

### 2. 重定向特点

- 可以跳转到应用内部或外部资源。
- 浏览器会先收到重定向响应，再发送第二次请求。
- 两次请求的 Request 对象不同，不能通过 Request 域共享数据。
- 浏览器地址栏会变为新的访问地址。
- 浏览器执行第二次请求，因此内部资源路径通常要带上下文路径。

---

## 八、转发与重定向对比

| 对比项 | 请求转发 Forward | 响应重定向 Redirect |
|---|---|---|
| 执行者 | 服务器 | 浏览器 |
| 请求次数 | 1 次 | 2 次 |
| 地址栏 | 不变 | 改变 |
| 跳转范围 | 当前应用内部 | 可跳转到应用内部或外部 |
| Request 数据共享 | 可以 | 不可以 |
| 应用内部路径 | 通常不带上下文路径 | 通常需要带上下文路径 |
| 常用方法 | `forward(request, response)` | `sendRedirect(url)` |

记忆口诀：

> **转发是服务器内部接力，重定向是浏览器重新出发。**

---

## 九、路径书写规则

判断是否添加上下文路径，关键在于由谁发起下一步访问。

### 1. 浏览器行为：通常带上下文路径

重定向由浏览器重新发起请求：

```java
response.sendRedirect(
        request.getContextPath() + "/resp3"
);
```

浏览器最终访问：

```text
/项目上下文路径/资源路径
```

### 2. 服务器行为：通常不带上下文路径

请求转发由服务器在当前应用内部完成：

```java
request.getRequestDispatcher("/resp3")
       .forward(request, response);
```

### 3. 路径速记

```text
浏览器发请求 → 带项目上下文路径
服务器内转发 → 不带项目上下文路径
```

---

## 十、常见响应类型

### 1. 响应中文文本

应在获取输出流之前设置响应类型和字符编码：

```java
response.setContentType("text/plain;charset=UTF-8");
response.getWriter().write("你好，Servlet！");
```

返回 HTML：

```java
response.setContentType("text/html;charset=UTF-8");
response.getWriter().write("<h1>你好，Servlet！</h1>");
```

### 2. 响应 JSON

```java
response.setContentType("application/json;charset=UTF-8");
response.getWriter().write(
        "{\"code\":200,\"message\":\"成功\"}"
);
```

实际项目中通常使用 Jackson、Gson 等工具将 Java 对象序列化为 JSON，避免手动拼接字符串。

### 3. 响应图片

```java
response.setContentType("image/png");

try (InputStream in = getServletContext()
        .getResourceAsStream("/images/logo.png");
     ServletOutputStream out = response.getOutputStream()) {

    if (in == null) {
        response.sendError(404);
        return;
    }

    byte[] buffer = new byte[8192];
    int length;
    while ((length = in.read(buffer)) != -1) {
        out.write(buffer, 0, length);
    }
}
```

---

## 十一、重点速记

> [!TIP]
> `Request` 负责“读请求”，`Response` 负责“写响应”。

- `getMethod()`：获取请求方式。
- `getContextPath()`：获取项目上下文路径。
- `getRequestURL()`：获取完整 URL。
- `getRequestURI()`：获取不含协议、主机和端口的 URI。
- `getHeader()`：获取请求头。
- `getParameter()`：获取单个请求参数。
- `getParameterValues()`：获取同名的多个参数值。
- `getReader()` / `getInputStream()`：读取字符或字节请求体。
- `setStatus()` / `setHeader()`：设置状态码或响应头。
- `getWriter()` / `getOutputStream()`：输出字符或字节响应体。
- POST 表单读取中文前，先调用 `setCharacterEncoding("UTF-8")`。
- 转发：一次请求、地址不变、应用内部、可共享 Request 数据。
- 重定向：两次请求、地址改变、可访问外部、不能共享 Request 数据。
- 浏览器行为通常带上下文路径，服务器内部转发通常不带。
- 写中文、HTML 或 JSON 前，应先正确设置 `Content-Type` 和字符编码。

---

> 本文根据思维导图 **《day46》** 整理，并对方法名称、编码处理、路径规则及响应示例进行了结构化归纳。
