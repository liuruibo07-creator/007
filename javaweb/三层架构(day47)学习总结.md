# Java Web 学习总结（Day 47）

> 本节核心内容：**Response 数据响应、Jackson JSON 转换、Java Web 三层架构，以及 Cookie / Session 会话跟踪技术**。

## 目录

- [一、Response 响应数据](#一response-响应数据)
- [二、Jackson JSON 工具](#二jackson-json-工具)
- [三、Java Web 三层架构](#三java-web-三层架构)
- [四、会话跟踪技术](#四会话跟踪技术)
- [五、Cookie](#五cookie)
- [六、Session](#六session)
- [七、Cookie 与 Session 对比](#七cookie-与-session-对比)
- [八、Filter 与 Listener](#八filter-与-listener)
- [九、重点速记](#九重点速记)

---

## 一、Response 响应数据

`HttpServletResponse` 用于设置 HTTP 响应报文，并向客户端返回文本、JSON、图片或其他二进制数据。

### 1. 响应中文

在获取输出流之前设置响应内容类型和字符编码：

```java
response.setContentType("text/html;charset=UTF-8");
response.getWriter().write("<h1>你好，Java Web！</h1>");
```

若只返回普通文本，可以使用：

```java
response.setContentType("text/plain;charset=UTF-8");
response.getWriter().write("操作成功");
```

> `Content-Type` 同时告诉浏览器响应的数据格式和字符编码，可避免中文乱码。

### 2. 响应 JSON

返回 JSON 时，应设置正确的媒体类型：

```java
response.setContentType("application/json;charset=UTF-8");
```

Java 对象通常先通过 Jackson 等工具转换成 JSON 字符串，再写入响应体。

完整示例：

```java
Student student = new Student(1, "张三");

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(student);

response.setContentType("application/json;charset=UTF-8");
response.getWriter().write(json);
```
```java
//json转为java对象
Student student2=objectMapper.readValue(json,Student.class);  
System.out.println(student2);
//将json转换为集合  ，匿名内部类
List<Student> list1= objectMapper.readValue(json2,new TypeReference<List<Student>>(){});  
System.out.println(list1);

```

| 方法                               | 转换方向                   |
| -------------------------------- | ---------------------- |
| `writeValueAsString(object)`     | Java 对象 → JSON 字符串     |
| `readValue(json, Xxx.class)`     | JSON 字符串 → 单个 Java 对象  |
| `readValue(json, TypeReference)` | JSON 字符串 → 带泛型的集合或复杂对象 |
### 3. 响应字节数据

图片、音频、文件等二进制数据应通过字节输出流返回：

```java
response.setContentType("image/jpeg");
//设置这个后
response.setHeader("content-disposition","attachment;filename=ccf200a6755a7e565672a6b41d7f2c43.jpg");


```

设置 `Content-Disposition` 可以提示浏览器以附件形式下载：

```java
response.setContentType("image/jpeg");
//加这个，在浏览器只下载，不打开
response.setHeader(
        "Content-Disposition",
        "attachment; filename=ccf200a6755a7e565672a6b41d7f2c43.jpg"
);
//1.读取本地图片  
FileInputStream fis=new FileInputStream("C:\\Users\\liuru\\Desktop\\ccf200a6755a7e565672a6b41d7f2c43.jpg");  
//2.获取响应输出流  
ServletOutputStream outputStream=response.getOutputStream();  
//3.读取图片写入到响应输出流  
/*byte[] bytes=new byte[1024];  
int len=0;  
while((len=fis.read(bytes))!=-1){  
    outputStream.write(bytes,0,len);}  
fis.close();*/  
//4.简化代码  
IOUtils.copy(fis,outputStream);
```

| 响应内容 | Content-Type 示例 | 输出方式 |
|---|---|---|
| 纯文本 | `text/plain;charset=UTF-8` | `getWriter()` |
| HTML | `text/html;charset=UTF-8` | `getWriter()` |
| JSON | `application/json;charset=UTF-8` | `getWriter()` |
| JPEG 图片 | `image/jpeg` | `getOutputStream()` |
| 二进制文件 | `application/octet-stream` | `getOutputStream()` |

---

## 二、Jackson JSON 工具

Jackson 可以在 **Java 对象**与 **JSON 数据**之间进行转换。

### 1. 引入依赖

导图中的依赖配置如下：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.6</version>
</dependency>
```

> 实际项目中应根据项目依赖管理要求选用合适版本，并尽量统一管理版本号。

### 2. 创建 ObjectMapper

```java
ObjectMapper mapper = new ObjectMapper();
```

`ObjectMapper` 是 Jackson 最常用的核心对象。



### 3. 常用方法总结

| 方法                               | 转换方向                   |
| -------------------------------- | ---------------------- |
| `writeValueAsString(object)`     | Java 对象 → JSON 字符串     |
| `readValue(json, Xxx.class)`     | JSON 字符串 → 单个 Java 对象  |
| `readValue(json, TypeReference)` | JSON 字符串 → 带泛型的集合或复杂对象 |

---

## 三、Java Web 三层架构

三层架构通过职责拆分降低代码耦合，提高可维护性和可测试性。

```text
客户端请求
    ↓
表现层 Web/Controller
    ↓
业务层 Service
    ↓
持久层 DAO/Mapper
    ↓
数据库
```

### 1. 表现层

常用包名：`web` 或 `controller`

==主要职责==

1. 获取请求参数。
2. 调用业务层。
3. 将处理结果响应给客户端。

常见技术：Servlet、Spring MVC。

生活类比：**服务员**，负责接收顾客需求并返回结果。

### 2. 业务层

常用包名：`service`

主要职责：

- 完成项目的核心业务逻辑。
- 组织业务流程。
- 根据业务需要调用一次或多次持久层方法。

常见技术：Spring。

生活类比：**厨师**，根据需求完成核心加工过程。

### 3. 持久层

常用包名：`dao` 或 `mapper`

主要职责：对数据进行增、删、改、查操作。

常见技术：JDBC、MyBatis、MyBatis-Plus。

生活类比：**采购员**，负责与数据来源打交道。

### 4. 各层对比

| 层次 | 常用命名 | 核心职责 | 常见技术 | 类比 |
|---|---|---|---|---|
| 表现层 | Web / Controller | 接收参数、调用业务、响应结果 | Servlet、Spring MVC | 服务员 |
| 业务层 | Service | 处理核心业务逻辑 | Spring | 厨师 |
| 持久层 | DAO / Mapper | 操作数据库 | MyBatis、MyBatis-Plus | 采购员 |

### 5. 面向接口编程

层与层之间通常通过接口交互：

```text
Web → Service 接口 → Service 实现类
Service → DAO 接口 → DAO 实现类
```

这样可以：

- 降低层与层之间的耦合。
- 更方便替换实现。
- 便于单元测试和后期维护。

> 上层应调用下层，不建议表现层绕过业务层直接操作 DAO。

---

## 四、会话跟踪技术

### 1. 为什么需要会话跟踪

HTTP 是无状态协议，服务器默认无法直接知道多个请求是否来自同一次会话，也无法在这些请求之间自动共享数据。

会话跟踪技术用于在一次会话中的多次请求之间识别用户并共享数据。

典型场景：

- 保存登录状态。
- 记录购物车。
- 保存用户偏好。
- 保存验证码或临时流程数据。

### 2. 主要实现方案

| 技术 | 数据存放位置 | 类型 |
|---|---|---|
| Cookie | 客户端浏览器 | 客户端会话跟踪技术 |
| Session | 服务器 | 服务端会话跟踪技术 |

---

## 五、Cookie（客户端会话跟踪技术）

Cookie 是将少量数据存储在浏览器中的客户端会话跟踪技术。

### 1. 创建并发送 Cookie

```java
Cookie cookie = new Cookie("username", "zhangsan");
response.addCookie(cookie);
```

执行 `addCookie()` 后，服务器通过响应头将 Cookie 发送给浏览器。

### 2. 获取 Cookie

```java
Cookie[] cookies = request.getCookies();

if (cookies != null) {
    for (Cookie cookie : cookies) {
        if ("username".equals(cookie.getName())) {
            String username = cookie.getValue();
            System.out.println(username);
        }
    }
}
```
获取Cookie后，浏览器发送request请求携带cookie
### 3. Cookie 的底层原理

服务器向浏览器添加 Cookie，本质是返回响应头：

```http
Set-Cookie: username=zhangsan
```

浏览器再次访问符合条件的资源时，通过请求头自动携带：
==获取cookie,本质上就是获取请求头 cooki==

```http
Cookie: username=zhangsan
```

```text
第一次请求
浏览器 ──────────────> 服务器
浏览器 <── Set-Cookie ─ 服务器

后续请求
浏览器 ─── Cookie ───> 服务器
```

### 4. Cookie 存活时间

通过 `setMaxAge()` 设置，单位为秒：

```java
cookie.setMaxAge(60 * 60 * 24 * 7); // 保存 7 天
```

| Max-Age 值 | 含义 |
|---|---|
| 负数（默认） | 会话 Cookie，通常在浏览器会话结束后删除 |
| 正整数 | 持久化 Cookie，保存指定秒数 |
| `0` | 通知浏览器立即删除 Cookie |

删除示例：

```java
Cookie cookie = new Cookie("username", "");
cookie.setMaxAge(0);
response.addCookie(cookie);
```

> 删除时通常还要保证 Cookie 的 `Path` 等属性与创建时一致。

### 5. Cookie 存储中文

为了兼容 Cookie 值的字符限制，可使用 URL 编码保存中文。

写入前编码：

```java
String value = URLEncoder.encode("手机号码", UTF_8);
Cookie cookie = new Cookie("phone", value);
response.addCookie(cookie);
```

读取后解码：

```java
String value = URLDecoder.decode(
        cookie.getValue(),
        StandardCharsets.UTF_8
);
```

编码结果类似：

```text
%E6%89%8B%E6%9C%BA%E5%8F%B7%E7%A0%81
```

URL 编码会先按 UTF-8 将字符串转换成字节，再将不适合直接传输的字节表示为 `%` 加十六进制值。

### 6. Cookie 使用注意事项

- 不要直接保存密码等敏感信息。
- Cookie 容量有限，适合保存少量数据。
- 客户端可以查看或修改 Cookie，不能把它当作可信数据源。
- 实际项目中常结合 `HttpOnly`、`Secure`、`SameSite` 等安全属性。

---

## 六、Session

Session 是把共享数据存储在服务器中的服务端会话跟踪技术。

### 1. 获取 Session 对象

```java
HttpSession session = request.getSession();
```

若当前请求没有可用 Session，`getSession()` 通常会创建一个新 Session。

### 2. 存储数据

```java
session.setAttribute("username", "张三");
```

### 3. 获取数据

```java
Object username = session.getAttribute("username");
```

也可以在不需要时删除单个属性：

```java
session.removeAttribute("username");
```

### 4. Session 的底层原理

Session 数据存储在服务器端，但服务器需要识别浏览器对应的 Session。

通常流程如下：

1. 服务器创建 Session，并生成唯一的会话 ID。
2. 服务器通过名为 `JSESSIONID` 的 Cookie 将会话 ID 发给浏览器。
3. 浏览器后续请求自动携带 `JSESSIONID`。
4. 服务器根据该 ID 找到对应的 Session 数据。

```text
浏览器保存：JSESSIONID=会话ID
                     ↓
服务器根据会话ID定位：HttpSession 对象
```

> Session 通常依赖 Cookie 传递会话 ID，但服务器端保存的是会话数据本身。

### 5. Session 钝化与活化

部分 Servlet 容器在正常关闭或资源管理过程中，可能进行 Session 持久化：

- **钝化**：将内存中的 Session 数据序列化并保存到文件。
- **活化**：服务器重新启动后，读取持久化数据并恢复 Session。

是否执行以及具体行为取决于服务器和应用配置。需要持久化的对象通常应支持序列化。

### 6. Session 销毁

#### 超时自动销毁

Tomcat 的默认非活动超时时间通常为 30 分钟。这里的 30 分钟是指在此期间没有新的请求访问该 Session，而不是浏览器必须保持操作。

可在 `web.xml` 中配置，单位为分钟：

```xml
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

#### 手动销毁

```java
session.invalidate();
```

退出登录时通常应主动销毁 Session。

---

## 七、Cookie 与 Session 对比

| 对比项 | Cookie | Session |
|---|---|---|
| 数据存放位置 | 客户端浏览器 | 服务器 |
| 数据容量 | 较小 | 相对较大，但会占用服务器资源 |
| 安全性 | 较低，客户端可查看和修改 | 相对较高，数据保存在服务器 |
| 生命周期 | 由 `Max-Age` 等属性控制 | 由非活动超时或 `invalidate()` 控制 |
| 主要用途 | 用户偏好、会话标识等少量数据 | 登录用户、验证码、流程状态等服务端数据 |
| 二者关系 | 可保存 `JSESSIONID` | 通常依靠 `JSESSIONID` Cookie 定位 |

实际应用中的常见组合：

```text
Cookie 保存会话 ID
        ↓
Session 保存真实的用户会话数据
```

---

## 八、Filter 

导图中还列出了 Java Web 的另外两个重要组件：

### 1. Filter 的作用

Filter 可以拦截进入 Web 应用的请求和返回客户端的响应，适合抽取多个资源共同需要的功能。

常见应用场景：

- 登录与权限校验。
- 请求字符编码处理。
- 日志记录。
- 敏感词过滤。
- 响应数据统一处理。

```text
客户端
  ↓ 请求
Filter
  ↓
Servlet / 目标资源
  ↓ 响应
Filter
  ↓
客户端
```

### 2. 定义 Filter

1. 定义类并实现 `Filter` 接口。
2. 重写 `doFilter()` 方法。
3. 使用 `@WebFilter` 配置拦截路径。
4. 调用 `chain.doFilter()` 放行请求。

```java
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;

import java.io.IOException;
//*为拦截所有url
@WebFilter("/*")
public class EncodingFilter implements Filter {

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {

        // 放行前：处理请求
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");

        // 放行，进入下一个过滤器或目标资源
        chain.doFilter(request, response);

        // 放行后：处理响应
    }
}
```

> 若没有调用 `chain.doFilter(request, response)`，请求将被当前过滤器拦截，不会继续访问目标资源。

### 3. 过滤器链

同一个请求匹配多个 Filter 时，这些 Filter 会组成过滤器链。

假设执行顺序为 `Filter1 → Filter2 → Servlet`：

```text
Filter1 放行前代码
    ↓
Filter2 放行前代码
    ↓
Servlet 的 doGet()/doPost()
    ↓
Filter2 放行后代码
    ↓
Filter1 放行后代码
```

执行结构类似方法嵌套：

```text
Filter1 {
    Filter2 {
        Servlet
    }
}
```

### 4. 多个 Filter 的顺序

使用注解配置时，部分 Servlet 容器会按过滤器类名的自然顺序组织过滤器链，例如字符编码顺序通常表现为：

```text
0-9 → A-Z → a-z
```

其常见起始编码值为：

| 字符范围 | 起始编码值 |
|---|:---:|
| `0-9` | 48 |
| `A-Z` | 65 |
| `a-z` | 97 |

> 注解 Filter 的顺序不应作为跨容器的可靠业务规则。若业务严格依赖过滤器顺序，应使用项目框架提供的顺序机制，或采用可明确控制顺序的配置方式。


## 九、Listener 监听器

Listener 用于监听 Web 应用中对象的创建、销毁或属性变化。

本节重点是 `ServletContextListener`。

### 1. ServletContext

`ServletContext` 表示整个 Web 应用：

- 一个 Web 应用通常只有一个 `ServletContext` 对象。
- 应用启动时创建。
- 应用关闭时销毁。
- 可用于读取应用资源、获取上下文信息或保存应用级共享数据。

### 2. 定义 ServletContextListener

```java
import jakarta.servlet.ServletContextEvent;
import jakarta.servlet.ServletContextListener;
import jakarta.servlet.annotation.WebListener;

@WebListener
public class ApplicationListener
        implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent event) {
        // Web 应用启动时执行
        System.out.println("应用已启动");
    }

    @Override
    public void contextDestroyed(ServletContextEvent event) {
        // Web 应用正常关闭时执行
        System.out.println("应用已关闭");
    }
}
```

| 方法 | 执行时机 |
|---|---|
| `contextInitialized()` | Web 应用初始化完成时 |
| `contextDestroyed()` | Web 应用销毁时 |

常见用途：应用启动时加载配置、初始化资源，应用关闭时释放资源。

---

## 十、重点速记

> [!TIP]
> **Response 负责返回数据；三层架构负责拆分职责；Cookie 和 Session 负责跨请求识别用户、共享会话数据。**

- 中文响应：先设置 `Content-Type` 和 UTF-8，再获取输出流。
- JSON 响应：`application/json;charset=UTF-8`。
- `writeValueAsString()`：Java 对象转 JSON。
- `readValue()`：JSON 转 Java 对象。
- 表现层：获取参数 → 调用 Service → 响应结果。
- 业务层：处理核心业务，可调用多个 DAO 方法。
- 持久层：负责数据库增、删、改、查。
- 层与层之间应尽量面向接口编程。
- Cookie 数据存储在客户端，通过 `Set-Cookie` 和 `Cookie` 请求头传递。
- `setMaxAge(正数)`：持久化；负数：会话级；`0`：删除。
- Cookie 保存中文时，可使用 URL 编码与解码。
- Session 数据存储在服务器端，通常通过 `JSESSIONID` Cookie 识别会话。
- `setAttribute()`：向 Session 存数据。
- `getAttribute()`：从 Session 取数据。
- `invalidate()`：立即销毁 Session，常用于退出登录。
- Cookie 适合少量非敏感数据，Session 适合服务端会话状态。

---

> 本文根据思维导图 **《day47》** 整理，对原有知识点进行了结构化归纳，并补充了响应示例、三层职责对比以及 Cookie / Session 的完整使用流程。
