# Day 58：Linux 项目部署与 Redis 入门

> 本笔记总结 Spring Boot 项目在 Linux 上的手动/自动部署流程，以及 Redis 的基础概念、安装方式与客户端连接方法。

## 一、Spring Boot 项目部署

### 1. 手动部署

#### 1.1 打包 Spring Boot 项目

在 `pom.xml` 中指定最终生成的文件名：

```xml
<finalName>app</finalName>
```

引入 Spring Boot Maven 打包插件：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

执行打包命令：

```bash
mvn clean package
```

打包完成后会得到 `app.jar`。

#### 1.2 上传并准备运行环境

1. 将 `app.jar` 上传到 Linux 服务器。
2. 在服务器上安装并配置 JDK、MySQL。
3. 前台启动项目：

```bash
java -jar app.jar
```

#### 1.3 后台启动项目

```bash
nohup java -Dspring.profiles.active=prod -jar app.jar > app.log 2>&1 &
```

参数说明：

| 参数 | 作用 |
| --- | --- |
| `nohup` | 忽略挂断信号，退出终端后服务仍可运行 |
| `-Dspring.profiles.active=prod` | 激活生产环境配置 |
| `> app.log` | 将标准输出写入日志文件 |
| `2>&1` | 将错误输出合并到标准输出 |
| `&` | 在后台运行进程 |

#### 1.4 停止服务

查询 Java 进程：

```bash
ps -ef | grep java
# 或
jps
```

停止指定进程：

```bash
kill -15 <PID>  # 优先使用：允许程序正常退出
kill -9 <PID>   # 无法正常退出时再强制终止
```

> [!WARNING]
> `kill -9` 不会给程序清理资源的机会，因此不建议作为首选方式。

---

### 2. 自动部署

自动部署的核心思路是：通过 Shell 脚本串联“停止旧服务、更新代码、重新打包、启动新服务”等操作。

#### 2.1 安装 Git

```bash
yum install -y git
```

#### 2.2 安装并配置 Maven

Maven 可以像 JDK 一样采用二进制包方式安装。安装后需要完成：

- 配置 `MAVEN_HOME`；
- 将 Maven 的 `bin` 目录加入 `PATH`；
- 配置本地仓库；
- 按需配置 Maven 私服。

#### 2.3 推送代码

将本地开发环境中的代码提交并推送到远程 Git 仓库，供服务器获取。

#### 2.4 编写部署脚本

脚本应依次完成以下任务：

1. 停止原有服务，优先执行 `kill -15 <PID>`，必要时再执行 `kill -9 <PID>`。
2. 获取项目代码：
   - 首次部署：`git clone <仓库地址>`；
   - 后续更新：`git pull`。
3. 重新打包：

```bash
mvn clean package -DskipTests
```

4. 后台启动：

```bash
nohup java -Dspring.profiles.active=prod -jar app.jar > app.log 2>&1 &
```

#### 2.5 执行部署脚本

授予脚本执行权限：

```bash
chmod 755 bootStart.sh
```

`755` 权限说明：

| 数字 | 对象 | 权限 |
| --- | --- | --- |
| `7` | 文件所有者 | 读、写、执行 |
| `5` | 同组用户 | 读、执行 |
| `5` | 其他用户 | 读、执行 |

执行脚本：

```bash
./bootStart.sh
```

### 3. 部署流程速记

```text
提交代码 → 服务器拉取代码 → Maven 打包 → 停止旧服务 → 后台启动新服务 → 查看日志
```

---

## 二、Redis 入门

### 1. Redis 概述

Redis 是一种基于内存的键值型非关系型数据库（NoSQL）。

### 2. Redis 安装

#### 2.1 Windows 安装

1. 下载并解压 Redis。
2. 使用配置文件启动服务：

```bash
redis-server redis.windows.conf
```

#### 2.2 Linux 安装

- 可以采用源码安装，需要先编译再安装；
- 源码通常建议放在 `/usr/local/src` 目录；
- 默认可执行文件安装到 `/usr/local/bin`；
- `/usr/local/bin` 通常已经加入 `PATH`，因此可以直接执行 Redis 命令。

### 3. Redis 客户端

#### 3.1 命令行客户端

使用 `redis-cli` 连接 Redis：

```bash
redis-cli -h <主机地址> -p <端口号>
```

常用参数：

| 参数 | 含义 | 默认值 |
| --- | --- | --- |
| `-h` | Redis 服务器 IP 地址 | `127.0.0.1` |
| `-p` | Redis 服务端口 | `6379` |
| `-a` | 连接密码 | 无 |

为了避免密码直接出现在命令历史中，建议先连接，再进行认证：

```bash
redis-cli -h 127.0.0.1 -p 6379
AUTH <密码>
```

#### 3.2 图形化客户端

可以使用 RESP 等图形化客户端管理 Redis。

---

## 三、核心命令清单

```bash
# Maven 打包
mvn clean package

# 后台启动 Spring Boot 项目
nohup java -Dspring.profiles.active=prod -jar app.jar > app.log 2>&1 &

# 查询 Java 进程
ps -ef | grep java
jps

# 停止进程
kill -15 <PID>

# Git 安装与代码更新
yum install -y git
git clone <仓库地址>
git pull

# 授予并执行部署脚本
chmod 755 bootStart.sh
./bootStart.sh

# 连接 Redis
redis-cli -h 127.0.0.1 -p 6379
```

> [!TIP]
> 排查部署问题时，可优先检查 Java 进程、端口占用、生产环境配置以及 `app.log` 中的错误信息。
