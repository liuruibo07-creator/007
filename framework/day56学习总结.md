# 自定义 Starter、Spring Boot 打包与 Linux 入门（Day 56）

> 核心内容：**自定义 Spring Boot Starter 的模块结构与自动配置注册、可执行 JAR 打包，以及 Linux、CentOS、VMware 和远程连接基础**。

## 目录

- [第一部分：自定义 Spring Boot Starter](#第一部分自定义-spring-boot-starter)
- [一、自定义 Starter 的作用](#一自定义-starter-的作用)
- [二、双模块结构](#二双模块结构)
- [三、编写自动配置模块](#三编写自动配置模块)
- [四、编写 Starter 模块](#四编写-starter-模块)
- [五、使用自定义 Starter](#五使用自定义-starter)
- [第二部分：Spring Boot 打包](#第二部分spring-boot-打包)
- [六、生成可执行 JAR](#六生成可执行-jar)
- [七、运行和外部配置](#七运行和外部配置)
- [第三部分：Linux 入门](#第三部分linux-入门)
- [八、Linux 概述](#八linux-概述)
- [九、内核与发行版](#九内核与发行版)
- [十、VMware 安装 CentOS](#十vmware-安装-centos)
- [十一、远程连接 Linux](#十一远程连接-linux)
- [十二、重点速记](#十二重点速记)

---

# 第一部分：自定义 Spring Boot Starter

## 一、自定义 Starter 的作用

自定义 Starter 用于把某项通用功能的依赖、配置属性和 Bean 注册逻辑封装起来，让其他项目只需引入一个依赖并填写少量配置即可使用。

典型场景：

- 公司内部统一日志、鉴权或审计组件。
- 短信、对象存储、支付等第三方客户端封装。
- Redis、消息队列等基础设施的统一配置。
- 多个项目共享的工具和业务组件。

```text
业务项目引入 Starter
        ↓
获得功能依赖
        ↓
Spring Boot 发现自动配置
        ↓
条件满足时注册 Bean
        ↓
业务代码直接注入使用
```

---

## 二、双模块结构

一个较规范的自定义 Starter 通常拆分为两个 Maven 模块：

```text
xxx-spring-boot-parent
├─ xxx-spring-boot-autoconfigure
└─ xxx-spring-boot-starter
```

### 1. autoconfigure 模块

```text
xxx-spring-boot-autoconfigure
```

主要负责：

- 定义配置属性类。
- 定义自动配置类。
- 注册功能所需的 Bean。
- 使用条件注解控制是否生效。
- 在规定文件中声明自动配置类。

### 2. starter 模块

```text
xxx-spring-boot-starter
```

主要负责：

- 聚合功能依赖。
- 引入 `xxx-spring-boot-autoconfigure`。
- 为使用者提供单一、清晰的依赖入口。

### 3. 两个模块的关系

```text
业务项目
   ↓ 依赖
xxx-spring-boot-starter
   ↓ 传递依赖
xxx-spring-boot-autoconfigure
   ↓
自动配置 + Bean
```

> 简单组件也可以只使用一个模块，但双模块结构能将“依赖入口”和“自动配置实现”分离，职责更加明确。

---

## 三、编写自动配置模块

下面以 `hello` 功能为例。

### 1. 项目结构

```text
hello-spring-boot-autoconfigure/
├─ pom.xml
└─ src/main/
   ├─ java/com/example/hello/
   │  ├─ HelloProperties.java
   │  ├─ HelloService.java
   │  └─ HelloAutoConfiguration.java
   └─ resources/
      └─ META-INF/spring/
         └─ org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

> IDEA 可能把 `META-INF/spring` 压缩显示成 `META-INF.spring`，磁盘上的实际结构仍是两级目录。

### 2. 配置属性类

```java
@ConfigurationProperties(prefix = "hello")
public class HelloProperties {
    private String prefix = "Hello";
    private String suffix = "!";

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

### 3. 功能类

```java
public class HelloService {

    private final HelloProperties properties;

    public HelloService(HelloProperties properties) {
        this.properties = properties;
    }

    public String sayHello(String name) {
        return properties.getPrefix()
                + ", " + name
                + properties.getSuffix();
    }
}
```

### 4. 自动配置类

```java
@AutoConfiguration
@EnableConfigurationProperties(HelloProperties.class)
@ConditionalOnClass(HelloService.class)
@ConditionalOnProperty(
        prefix = "hello",
        name = "enabled",
        havingValue = "true",
        matchIfMissing = true
)
public class HelloAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public HelloService helloService(
            HelloProperties properties
    ) {
        return new HelloService(properties);
    }
}
```

关键注解：

| 注解 | 作用 |
|---|---|
| `@AutoConfiguration` | 声明 Spring Boot 自动配置类 |
| `@EnableConfigurationProperties` | 启用配置属性绑定 |
| `@ConditionalOnClass` | 指定类存在时才生效 |
| `@ConditionalOnProperty` | 配置属性满足条件时才生效 |
| `@ConditionalOnMissingBean` | 用户未自定义同类 Bean 时提供默认实现 |

### 5. 注册自动配置类

Spring Boot 3 使用文件：

```text
src/main/resources/META-INF/spring/
org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

文件内容是一行一个自动配置类全限定名：

```text
com.example.hello.HelloAutoConfiguration
```

多个配置类：

```text
com.example.hello.HelloAutoConfiguration
com.example.hello.HelloWebAutoConfiguration
```

> 文件名和路径必须准确；导图配图展示的正是 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`。

### 6. 配置提示（可选）

可添加配置处理器，使 IDE 为自定义属性提供提示：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

---

## 四、编写 Starter 模块

Starter 本身通常不包含复杂 Java 代码，主要通过 `pom.xml` 聚合依赖。

```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>hello-spring-boot-autoconfigure</artifactId>
        <version>${project.version}</version>
    </dependency>

    <!-- 功能运行所需的其他依赖 -->
</dependencies>
```

推荐命名：

```text
第三方 Starter：功能名-spring-boot-starter
官方 Starter：spring-boot-starter-功能名
```

例如自定义项目可命名为：

```text
hello-spring-boot-starter
```

---

## 五、使用自定义 Starter

### 1. 引入依赖

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>hello-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

### 2. 添加配置

```yaml
hello:
  enabled: true
  prefix: Welcome
  suffix: "!"
```

### 3. 注入使用

```java
@RestController
public class HelloController {

    private final HelloService helloService;

    public HelloController(HelloService helloService) {
        this.helloService = helloService;
    }

    @GetMapping("/hello")
    public String hello(String name) {
        return helloService.sayHello(name);
    }
}
```

### 4. 构建与安装

开发阶段先把自定义 Starter 安装到本地 Maven 仓库：

```bash
mvn clean install
```

团队共享时通常发布到公司的 Maven 私服。

---

# 第二部分：Spring Boot 打包

## 六、生成可执行 JAR

### 1. 配置打包插件

```xml
<build>
    <!-- 设置最终 JAR 名称 -->
    <finalName>app</finalName>

    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

`spring-boot-maven-plugin` 的 `repackage` 能将普通 JAR 重新打包为包含应用类、依赖库和启动信息的可执行 JAR。

使用 `spring-boot-starter-parent` 时，插件执行通常已有合适的默认配置。若未使用父工程或对应插件管理，可能需要显式配置执行目标。

### 2. 执行打包

```bash
mvn clean package
```

跳过测试：

```bash
mvn clean package -DskipTests
```

构建产物：

```text
target/app.jar
```

### 3. 打包前检查

- 项目测试是否通过。
- Java 版本是否与服务器一致。
- 生产环境敏感配置是否已外置。
- 数据库、上传目录、日志目录是否适合服务器环境。

---

## 七、运行和外部配置

### 1. 运行 JAR

```bash
java -jar app.jar
```

指定环境：

```bash
java -jar app.jar --spring.profiles.active=prod
```

覆盖端口：

```bash
java -jar app.jar --server.port=8081
```

设置 JVM 参数时，应放在 `-jar` 前：

```bash
java -Xms256m -Xmx512m -jar app.jar
```

### 2. 使用外部配置

可以将配置文件放在 JAR 同级的 `config` 目录：

```text
deploy/
├─ app.jar
└─ config/
   ├─ application.yml
   └─ application-prod.yml
```

运行：

```bash
java -jar app.jar --spring.profiles.active=prod
```

外部配置便于在不重新打包的情况下修改环境参数，并避免把生产密码写入代码包。

### 3. 基础诊断

```bash
java -version
java -jar app.jar
```

若启动失败，优先检查：

- JDK 版本。
- 端口是否被占用。
- 配置文件位置和激活环境。
- 数据库和其他外部服务是否可访问。
- 日志中的首个根因异常。

---

# 第三部分：Linux 入门

## 八、Linux 概述

Linux 是一种开源、类 Unix 操作系统，在服务器、云计算、容器和嵌入式设备中广泛使用。

课程中把 Linux 简化理解为“服务器端操作系统”，但它也可运行在个人电脑、移动设备和嵌入式环境中。

Java Web 项目常见部署结构：

```text
开发电脑
   ↓ 打包、上传
Linux 服务器
├─ JDK
├─ Spring Boot JAR
├─ 数据库或数据库客户端
├─ 日志目录
└─ 运维脚本
```

---

## 九、内核与发行版

### 1. Linux 内核

Linux 内核是操作系统核心，负责：

- CPU 调度。
- 内存管理。
- 文件系统。
- 网络通信。
- 驱动硬件设备。
- 进程与权限管理。

### 2. Linux 发行版

发行版通常由 Linux 内核、系统工具、软件包管理器和桌面/服务组件组成。

```text
Linux 内核 + GNU 工具 + 包管理器 + 默认软件 = Linux 发行版
```

导图课程使用的是：

```text
CentOS 7
```

> CentOS 7 适合作为旧课程环境复现，但已属于老旧发行版。真实新项目应根据公司基础设施和维护周期选择仍受支持的发行版。

---

## 十、VMware 安装 CentOS

VMware 是虚拟化软件，可以在一台物理电脑上模拟一套虚拟硬件，再在虚拟硬件中安装 Linux。

```text
物理电脑
└─ VMware
   └─ 虚拟机硬件
      └─ CentOS 7
```

### 1. 基本步骤

1. 安装 VMware。
2. 准备 CentOS ISO 镜像。
3. 创建新虚拟机。
4. 分配 CPU、内存和磁盘。
5. 选择网络模式。
6. 挂载 ISO 并安装 CentOS。
7. 设置用户名、密码、时区和磁盘分区。
8. 启动系统并查看 IP 地址。

### 2. 网络模式

| 模式 | 说明 |
|---|---|
| NAT | 虚拟机通过宿主机访问外网，学习环境常用 |
| 桥接 | 虚拟机像独立设备一样接入局域网 |
| 仅主机 | 主要与宿主机和同网段虚拟机通信 |

### 3. 查看 IP

```bash
ip addr
```

找到网卡的 IPv4 地址，例如：

```text
192.168.10.100
```

### 4. 验证网络

```bash
ping -c 4 127.0.0.1
ping -c 4 网关地址
ping -c 4 外部地址
```

---

## 十一、远程连接 Linux

FinalShell 是支持 SSH 和文件传输的远程连接工具。也可使用系统自带 OpenSSH、PuTTY 等工具。

### 1. SSH 连接信息

需要准备：

| 配置 | 示例 |
|---|---|
| 主机地址 | `192.168.10.100` |
| SSH 端口 | `22` |
| 用户名 | `root` 或普通用户 |
| 认证 | 密码或 SSH 密钥 |

命令行连接：

```bash
ssh username@192.168.10.100
```

指定非默认端口：

```bash
ssh -p 2222 username@192.168.10.100
```

### 2. 连接失败排查

```text
虚拟机是否启动
    ↓
IP 地址是否正确
    ↓
宿主机能否 ping 通虚拟机
    ↓
SSH 服务是否运行
    ↓
22 端口是否被防火墙放行
    ↓
用户名、密码或密钥是否正确
```

### 3. 基础命令

```bash
pwd                 # 当前目录
ls -la              # 查看文件
cd /path            # 切换目录
mkdir app            # 创建目录
cp source target    # 复制
mv source target    # 移动或重命名
tail -f app.log     # 实时查看日志
ps -ef               # 查看进程
```

> 日常部署建议使用普通用户和必要权限，避免长期直接使用 root 操作应用。

---

## 十二、重点速记

> [!TIP]
> **autoconfigure 模块负责自动配置，starter 模块负责聚合依赖；Spring Boot 插件生成可执行 JAR，Linux 是常见的服务器运行环境。**

- 自定义 Starter 常拆为 `autoconfigure` 和 `starter` 两个模块。
- 自动配置类使用 `@AutoConfiguration`。
- 配置属性类使用 `@ConfigurationProperties`。
- `@ConditionalOnMissingBean` 允许用户自定义 Bean 覆盖默认实现。
- Spring Boot 3 在 `AutoConfiguration.imports` 中登记自动配置类。
- 文件路径为 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`。
- Starter 模块必须依赖 autoconfigure 模块。
- 开发阶段使用 `mvn clean install` 安装到本地仓库。
- `spring-boot-maven-plugin` 用于生成可执行 JAR。
- `<finalName>app</finalName>` 将产物命名为 `app.jar`。
- 使用 `java -jar app.jar` 运行应用。
- JVM 参数放在 `-jar` 之前，应用参数放在 JAR 文件名之后。
- 外部配置便于切换环境和保护生产配置。
- Linux 内核负责硬件和系统资源管理，发行版在内核之上提供完整系统。
- VMware 模拟硬件，CentOS 安装在虚拟硬件中。
- FinalShell 等工具通过 SSH 远程连接 Linux。
- 远程连接需要确认 IP、端口、SSH 服务、网络和认证信息。

---

> 本文根据思维导图 **《day56》** 整理，并将自动配置文件配图转换为标准目录结构，同时补充了构建、运行和 Linux 连接检查清单。
