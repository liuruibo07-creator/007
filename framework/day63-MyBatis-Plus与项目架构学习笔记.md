# Day 63｜MyBatis-Plus 与项目架构学习笔记

> **学习主题：** 乐观锁、逻辑删除配置、通用 Service、代码生成、项目架构、Nginx、ThreadLocal、DTO/VO、Hutool、链式查询与登录校验。

---

## 目录

- [一、MyBatis-Plus 乐观锁](#一mybatis-plus-乐观锁)
- [二、逻辑删除全局配置](#二逻辑删除全局配置)
- [三、通用 Service](#三通用-service)
- [四、代码生成](#四代码生成)
- [五、项目架构与扩展方式](#五项目架构与扩展方式)
- [六、Nginx 的作用](#六nginx-的作用)
- [七、ThreadLocal](#七threadlocal)
- [八、DTO 与 VO](#八dto-与-vo)
- [九、Web 对象的自动注入](#九web-对象的自动注入)
- [十、Hutool 常用工具](#十hutool-常用工具)
- [十一、Wrapper 与 ChainWrapper](#十一wrapper-与-chainwrapper)
- [十二、登录校验](#十二登录校验)
- [十三、知识点速记](#十三知识点速记)

---

## 一、MyBatis-Plus 乐观锁

### 1.1 核心思想

乐观锁本质上并没有对数据真正加锁，而是假设并发冲突较少，在更新数据时通过版本号判断数据是否已被其他操作修改。

### 1.2 实现步骤

1. 在数据库表中添加版本号字段 `version`，默认值设为 `1`；
2. 在实体类中添加 `version` 属性，并标注 `@Version`；
3. 注册 MyBatis-Plus 的乐观锁内部拦截器；
4. 执行更新操作时，实体对象必须携带查询到的旧版本号。

实体类示例：

```java
@Version
private Integer version;
```

拦截器配置示例：

```java
@Configuration
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}
```

### 1.3 更新原理

更新语句会同时检查旧版本号，并在更新成功后将版本号加一：

```sql
UPDATE tb_user
SET age = ?, version = version + 1
WHERE id = ? AND version = oldVersion;

 //查询  
User user = userMapper.selectById(1L);  
user.setAge(20);  
userMapper.updateById(user);
```

如果版本号不匹配，说明数据已被其他线程修改，本次更新不会命中记录。

> [!IMPORTANT]
> 乐观锁适合读多写少、冲突概率较低的场景。更新前应先查询数据以取得当前版本号，并检查更新操作影响的行数。

---

## 二、逻辑删除全局配置

MyBatis-Plus 的默认逻辑删除约定为：

| 配置项 | 默认约定 |
| --- | --- |
| 字段名 | `deleted` |
| 未删除 | `0` |
| 已删除 | `1` |

逻辑删除不会真正移除数据库记录，而是修改删除标识。之后执行常规查询时，MyBatis-Plus 会自动过滤已删除的数据。

实体类中的字段通常写为：

```java
@TableLogic
private Integer deleted;
```

---

## 三、通用 Service

MyBatis-Plus 除了提供 `BaseMapper`，还提供了通用 Service 层能力：

- `IService<T>`：通用 Service 接口；
- `ServiceImpl<M, T>`：通用 Service 实现类；
- `M` 表示 Mapper 类型，`T` 表示实体类型。

### 3.1 定义业务接口

```java
public interface UserService extends IService<User> {
}
```

### 3.2 定义业务实现类

```java
@Service
public class UserServiceImpl
        extends ServiceImpl<UserMapper, User>
        implements UserService {
}
```

完成继承后，Service 层可以直接使用通用的新增、删除、修改、查询和批量操作方法，也可以继续添加项目自己的业务方法。

---

## 四、代码生成

常见的代码生成方式有两种：

1. 使用 IDE 中的 MyBatis-X 插件生成；
2. 使用 MyBatis-Plus 官方代码生成器。

### 4.1 官方生成器依赖

```xml
<!-- MyBatis-Plus 代码生成器 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.17</version>
</dependency>

<!-- 模板引擎：Velocity、Freemarker、Beetl 三选一 -->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.3</version>
</dependency>
```

代码生成器通常用于根据数据库表快速生成以下结构：

```text
数据库表
├── Entity
├── Mapper
├── Service
├── ServiceImpl
└── Controller
```

> [!NOTE]
> 模板引擎为必选依赖，但只需选择一种，并按照生成器配置使用对应模板。

---

## 五、项目架构与扩展方式

导图中的点评项目架构可以概括为：

```text
手机、电脑等客户端
        │
        ▼
      Nginx
        │
        ▼
  Tomcat 应用服务器集群
        │
        ├── Redis 集群
        └── MySQL 集群
```

各层职责：

- **客户端**：发起 HTTP 请求；
- **Nginx**：提供静态资源、反向代理并进行负载均衡；
- **Tomcat 集群**：运行 Java Web 应用、处理业务请求；
- **Redis 集群**：承载缓存、验证码或会话相关数据；
- **MySQL 集群**：持久化业务数据。

### 5.1 水平扩展

通过增加服务器节点的数量提升系统承载能力，例如将一台 Tomcat 扩展为多台 Tomcat。

```text
水平扩展 = 增加机器数量
```

### 5.2 垂直扩展

通过升级单台服务器的 CPU、内存、磁盘等硬件提高性能。

```text
垂直扩展 = 提升单台机器性能
```

---

## 六、Nginx 的作用

### 6.1 静态资源服务器

Nginx 可以直接提供 HTML、CSS、JavaScript、图片等静态资源，减少应用服务器的压力。

### 6.2 反向代理

Nginx 接收客户端请求，再将请求转发给后方的 Tomcat 应用服务器。

| 类型 | 代理对象 | 常见例子 |
| --- | --- | --- |
| 正向代理 | 客户端 | VPN、客户端网络代理 |
| 反向代理 | 服务器 | Nginx 代理后方 Tomcat 集群 |

客户端通常知道正向代理的存在，但不需要知道反向代理后方具体由哪台服务器处理请求。

### 6.3 负载均衡

Nginx 可以将请求分配给多个 Tomcat 节点，常见策略如下：

| 策略 | 原理 | 特点 |
| --- | --- | --- |
| 轮询 | 按顺序依次分发请求 | 简单，适合节点性能相近的场景 |
| 加权 | 权重越大的节点获得越多请求 | 通常为高性能服务器设置更高权重 |
| IP Hash | 对客户端 IP 进行哈希，再对节点数量取模 | 同一客户端通常会命中同一服务器 |

---

## 七、ThreadLocal

### 7.1 核心原理

`ThreadLocal` 是线程局部变量工具。多个线程即使使用同一个 `ThreadLocal` 对象，也会在各自线程内部的 `ThreadLocalMap` 中保存独立的数据副本，从而实现线程之间的数据隔离。

```text
同一个 ThreadLocal 对象
        │
        ├── 线程 1 的 ThreadLocalMap：保存值 1
        └── 线程 2 的 ThreadLocalMap：保存值 2
```

因此，线程 1 修改自己的值时，不会直接改变线程 2 保存的值。

### 7.2 应用场景

项目中常用 `ThreadLocal` 在同一线程的调用链路中传递上下文数据，例如：

- 当前登录用户 ID；
- 当前用户信息；
- 请求范围内需要共享的数据。

```text
拦截器 → Controller → Service → Mapper
          同一请求处理线程
```

### 7.3 常用 API

| 方法 | 作用 |
| --- | --- |
| `set(T value)` | 将变量副本保存到当前线程的 `ThreadLocalMap` 中 |
| `get()` | 获取当前线程保存的变量副本 |
| `remove()` | 移除当前线程保存的变量副本 |

示例：

```java
private static final ThreadLocal<Long> USER_HOLDER = new ThreadLocal<>();

USER_HOLDER.set(userId);  // 保存当前用户 ID
Long id = USER_HOLDER.get();
USER_HOLDER.remove();     // 使用完成后清理
```

### 7.4 注意事项

在线程池环境中，线程会被重复使用。如果使用完成后没有调用 `remove()`，旧数据可能被后续请求误用，并带来内存泄漏风险。

因此，应在请求处理完成后及时清理，通常可以在拦截器的完成回调中执行：

```java
@Override
public void afterCompletion(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler,
        Exception ex) {
    USER_HOLDER.remove();
}
```

> [!WARNING]
> `set()` 与 `remove()` 最好成对出现。尤其在 Tomcat 线程池中，忘记清理不仅可能造成内存问题，也可能导致用户数据串用。

---

## 八、DTO 与 VO

### 8.1 DTO

DTO（Data Transfer Object，数据传输对象）常用于封装请求参数。具体字段应参考接口文档中的请求参数定义。

```text
客户端请求 → DTO → Controller
```

### 8.2 VO

VO（View Object，视图对象）常用于封装返回给客户端的响应数据。具体字段应参考接口文档中的响应数据定义。

```text
Controller → VO → 客户端响应
```

### 8.3 与实体类的区别

| 对象 | 主要用途 | 字段依据 |
| --- | --- | --- |
| Entity | 映射数据库表 | 数据库表结构 |
| DTO | 接收、传递请求数据 | 接口请求参数 |
| VO | 返回展示数据 | 接口响应结构 |

将 Entity、DTO、VO 分开，可以避免直接暴露数据库字段，也能让请求和响应结构更贴近接口需求。

---

## 九、Web 对象的自动注入

在 Spring MVC 控制器的方法参数中，可以直接声明 Java Web 对象，框架会自动传入当前请求对应的实例：

```java
@GetMapping("/example")
public void example(
        HttpServletRequest request,
        HttpServletResponse response,
        HttpSession session) {
    // 直接使用 request、response 和 session
}
```

常见对象包括：

- `HttpServletRequest`：读取请求信息；
- `HttpServletResponse`：操作响应信息；
- `HttpSession`：保存和读取会话数据。

---

## 十、Hutool 常用工具

### 10.1 生成随机数字字符串

```java
String code = RandomUtil.randomNumbers(6);
```

生成一个 6 位随机数字字符串，常用于验证码等场景。

### 10.2 生成随机字符串

```java
String value = RandomUtil.randomString(10);
```

生成一个长度为 10、包含字母和数字的随机字符串。

### 10.3 对象属性拷贝

```java
UserDTO dto = BeanUtil.copyProperties(user, UserDTO.class);
```

将 `user` 中可匹配的属性复制到新的 `UserDTO` 对象中。要顺利完成拷贝，对应属性通常需要名称一致且类型兼容。

---

## 十一、Wrapper 与 ChainWrapper

### 11.1 主要区别

| 类型 | 能否构造条件 | 能否直接执行数据库操作 |
| --- | --- | --- |
| `QueryWrapper` / `UpdateWrapper` | 可以 | 通常需要交给 Mapper 或 Service 执行 |
| `QueryChainWrapper` / `UpdateChainWrapper` | 可以 | 可以继续链式调用查询或更新方法 |

普通 Wrapper 主要负责拼接条件：

```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("age", 18);
List<User> users = userMapper.selectList(wrapper);
```

链式 Wrapper 可以构造条件后直接执行：

```java
List<User> users = userService.query()
        .eq("age", 18)
        .list();
```

在 Service 中可以通过以下方法获取链式构造器：

```java
userService.query();       // QueryChainWrapper
userService.lambdaQuery(); // LambdaQueryChainWrapper
```

Lambda 写法可以使用实体类属性的方法引用，减少手写数据库字段名：

```java
List<User> users = userService.lambdaQuery()
        .eq(User::getAge, 18)
        .list();
```

---

## 十二、登录校验

登录校验的基本实现思路是：

1. 定义拦截器；
2. 在请求进入 Controller 前检查用户是否已登录；
3. 校验通过后，可以将用户信息保存到 `ThreadLocal`；
4. 注册拦截器并配置需要拦截或排除的路径；
5. 请求结束后清理 `ThreadLocal` 中的数据。

```text
客户端请求
    │
    ▼
登录校验拦截器
    ├── 未登录：阻止请求并返回提示
    └── 已登录：保存用户上下文 → 放行请求
                                  │
                                  ▼
                           Controller / Service
                                  │
                                  ▼
                        请求结束后清理 ThreadLocal
```

注册拦截器的结构示例：

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/user/login");
    }
}
```

---

## 十三、知识点速记

> [!TIP]
> - **乐观锁：** 更新时通过版本号判断是否发生并发修改。
> - **逻辑删除：** 修改删除标识，不真正删除数据库记录。
> - **通用 Service：** `IService<T>` 配合 `ServiceImpl<M, T>`。
> - **水平扩展：** 增加服务器数量。
> - **垂直扩展：** 提升单台服务器性能。
> - **Nginx：** 静态资源服务、反向代理、负载均衡。
> - **ThreadLocal：** 保存当前线程独有的数据副本，用完必须清理。
> - **DTO：** 封装请求参数。
> - **VO：** 封装响应结果。
> - **ChainWrapper：** 构造条件后可以继续链式执行数据库操作。
> - **登录校验：** 拦截器负责校验，ThreadLocal 可保存当前用户上下文。

### 一句话串联

**MyBatis-Plus 负责简化数据访问，Nginx 与集群架构负责提升系统承载能力，拦截器配合 ThreadLocal 完成登录校验和用户上下文传递，DTO/VO 则负责隔离接口数据与数据库实体。**
