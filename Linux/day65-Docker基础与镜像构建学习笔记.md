# Day 65｜Docker 基础与镜像构建学习笔记

> **学习主题：** Docker 核心概念、安装准备、容器运行、常用命令、数据卷、本地目录挂载、Dockerfile 与自定义镜像。

---

## 目录

- [一、Docker 概述](#一docker-概述)
- [二、镜像、容器与仓库](#二镜像容器与仓库)
- [三、Docker 安装要点](#三docker-安装要点)
- [四、入门案例：运行 MySQL 容器](#四入门案例运行-mysql-容器)
- [五、Docker 常用命令](#五docker-常用命令)
- [六、数据卷](#六数据卷)
- [七、本地目录挂载](#七本地目录挂载)
- [八、自定义镜像](#八自定义镜像)
- [九、Dockerfile 常用指令](#九dockerfile-常用指令)
- [十、完整构建示例](#十完整构建示例)
- [十一、知识点速记](#十一知识点速记)

---

## 一、Docker 概述

Docker 是一种容器化工具，主要用于打包、分发和部署软件。它可以把应用程序及其运行依赖封装成镜像，再使用镜像快速创建环境一致的容器。

```text
应用程序 + 运行依赖
          │
          ▼
        镜像
          │ docker run
          ▼
        容器
```

Docker 的主要价值：

- 减少“开发环境能运行，生产环境不能运行”的问题；
- 简化软件安装和部署流程；
- 快速创建、停止、删除应用实例；
- 便于在不同 Linux 环境之间迁移应用。

---

## 二、镜像、容器与仓库

### 2.1 镜像（Image）

镜像是将软件及其运行所需依赖、函数库打包后的结果，是一种**静态、只读的模板**。

可以类比 Java 中的类：

```text
Docker 镜像 ≈ Java 类
```

镜像通常使用以下格式标识：

```text
镜像名称:版本标签
```

例如：

```text
mysql:8
nginx:latest
```

如果省略标签，Docker 默认使用 `latest`：

```bash
docker pull nginx
# 等价于 docker pull nginx:latest
```

> [!NOTE]
> `latest` 只是默认标签名称，并不能在所有镜像仓库中绝对保证它指向软件的最新版本。生产环境建议明确指定版本标签。

### 2.2 容器（Container）

容器是镜像运行后创建的实例，是一个**动态概念**。

可以类比 Java 中的对象：

```text
Docker 镜像 ≈ Java 类
Docker 容器 ≈ Java 对象
```

同一个镜像可以创建多个相互隔离的容器：

```text
mysql:8 镜像
├── MySQL 容器 1
├── MySQL 容器 2
└── MySQL 容器 3
```

容器共享宿主机的 Linux 内核，不需要像传统虚拟机一样为每个实例运行一套完整操作系统。

### 2.3 镜像仓库（Registry）

镜像仓库用于存储和分发 Docker 镜像。

| 类型 | 说明 |
| --- | --- |
| 中央仓库 | 公共镜像仓库，例如 Docker Hub |
| 私有仓库 | 公司或团队自行搭建的镜像仓库 |
| 本地镜像库 | 当前主机已经下载或构建的镜像集合 |

常见流转过程：

```text
远程仓库 ──docker pull──▶ 本地镜像库
远程仓库 ◀─docker push── 本地镜像库
本地镜像 ──docker run──▶ 容器
```

---

## 三、Docker 安装要点

Docker 的具体安装步骤应以对应 Linux 发行版的安装文档为准。

通过 `yum` 安装 Docker 时，需要注意：

1. 先添加 Docker 软件源仓库；
2. 再通过软件源下载并安装 Docker；
3. 根据网络环境配置镜像加速器或公司私有仓库，以提升镜像下载速度。

```text
配置 yum 软件源 → 安装 Docker → 启动 Docker → 配置镜像加速/私服
```

---

## 四、入门案例：运行 MySQL 容器

典型命令结构如下：

```bash
docker run -d \
  --name mysql \
  -p 3306:3306 \
  mysql:8
```

各部分含义：

| 参数 | 作用 |
| --- | --- |
| `docker run` | 创建并运行一个新容器 |
| `-d` | 让容器在后台运行 |
| `--name mysql` | 将容器命名为 `mysql`，名称不能与现有容器重复 |
| `-p 3306:3306` | 将宿主机端口映射到容器端口 |
| `mysql:8` | 使用名称为 `mysql`、标签为 `8` 的镜像 |

### 4.1 端口映射

端口映射格式：

```text
-p 宿主机端口:容器端口
```

因此：

```bash
-p 3306:3306
```

表示访问宿主机的 `3306` 端口时，请求会被转发到容器内部的 `3306` 端口。

```text
客户端 → 宿主机:3306 → MySQL 容器:3306
```

> [!IMPORTANT]
> 实际运行官方 MySQL 镜像时，通常还需要通过 `-e` 设置 root 密码等必要环境变量；此处重点是理解 `docker run` 的基本参数。

---

## 五、Docker 常用命令

Docker 最常用的命令可以分为镜像操作和容器操作。

### 5.1 镜像命令

| 命令              | 作用                 | 示例                               |
| --------------- | ------------------ | -------------------------------- |
| `docker pull`   | 从远程仓库拉取镜像          | `docker pull nginx:latest`       |
| `docker push`   | 将本地镜像推送到远程仓库       | `docker push user/app:1.0`       |
| `docker images` | 查看本地镜像             | `docker images`                  |
| `docker rmi`    | 删除本地镜像             | `docker rmi nginx:latest`        |
| `docker build`  | 根据 Dockerfile 构建镜像 | `docker build -t app:1.0 .`      |
| `docker save`   | 将镜像导出为归档文件         | `docker save -o app.tar app:1.0` |
| `docker load`   | 从归档文件导入镜像          | `docker load -i app.tar`         |
```
docker save -o uuu.tar nginx:latest
docker load -i  uuu.tar
```
镜像导入导出流程：

```text
本地镜像 ──docker save──▶ app.tar
本地镜像 ◀─docker load── app.tar
```

### 5.2 容器命令

| 命令 | 作用 | 示例 |
| --- | --- | --- |
| `docker run` | 创建并运行容器 | `docker run -d nginx` |
| `docker ps` | 查看正在运行的容器 | `docker ps` |
| `docker ps -a` | 查看所有容器，包括已停止容器 | `docker ps -a` |
| `docker stop` | 停止运行中的容器 | `docker stop nginx` |
| `docker start` | 启动已停止的容器 | `docker start nginx` |
| `docker rm` | 删除已停止的容器 | `docker rm nginx` |
| `docker rm -f` | 强制删除容器 | `docker rm -f nginx` |
| `docker logs` | 查看容器日志 | `docker logs nginx` |
| `docker logs -f` | 持续查看实时日志 | `docker logs -f nginx` |
| `docker exec` | 在运行中的容器内执行命令 | `docker exec -it nginx /bin/bash` |
```
docker run -d --name nginx -p 80:80 nginx:latest
docker exec -it nginx /bin/bash 
```
### 5.3 容器生命周期

```text
镜像
 │ docker run
 ▼
运行中的容器 ──docker stop──▶ 已停止容器
       ▲                         │
       └──────docker start───────┘
                                 │ docker rm
                                 ▼
                               删除
```

### 5.4 进入容器

```bash
docker exec -it 容器名称 /bin/bash
```

- `exec`：在正在运行的容器中执行命令；
- `-i`：保持标准输入打开；
- `-t`：分配一个伪终端；
- `/bin/bash`：启动 Bash Shell。

如果镜像中没有 Bash，可以根据镜像环境尝试 `/bin/sh`。

---

## 六、数据卷

### 6.1 概念

数据卷（Volume）可以理解为由 Docker 管理的虚拟目录。它将宿主机上的实际目录与容器内目录映射，实现双向绑定。

```text
宿主机中的数据卷目录 ⇄ 容器内目录
```

容器内对挂载目录的修改会反映到宿主机，宿主机对该目录的修改也能被容器看到。

数据卷的主要价值：

- 实现数据持久化，避免删除容器时同时丢失业务数据；
- 方便宿主机与容器之间共享文件；
- 让多个容器共享同一份数据；
- 将重要数据与容器生命周期解耦。

### 6.2 数据卷命令

| 命令 | 作用 |
| --- | --- |
| `docker volume create 数据卷名称` | 创建数据卷 |
| `docker volume ls` | 查看数据卷列表 |
| `docker volume inspect 数据卷名称` | 查看数据卷详情和宿主机实际路径 |
| `docker volume rm 数据卷名称` | 删除指定数据卷 |
| `docker volume prune` | 删除所有未被使用的本地数据卷 |

> [!WARNING]
> `docker volume rm` 和 `docker volume prune` 会删除持久化数据。执行前应确认数据卷确实不再需要，并检查是否已经备份。

### 6.3 挂载数据卷

语法：

```text
-v 数据卷名称:容器内目录
```
```
docker run -d --name nginx -p 80:80
```
示例：

```bash
docker run -d \
  --name nginx \
  -v html-data:/usr/share/nginx/html \
  nginx:latest
```

如果 `html-data` 数据卷不存在，Docker 会自动创建它，然后再完成挂载。

---

## 七、本地目录挂载

除了 Docker 管理的数据卷，也可以将宿主机上的指定目录直接挂载到容器内部。这种方式通常称为 Bind Mount。

语法：

```text
-v 本地目录:容器内目录
```

示例：

```bash
docker run -d \
  --name nginx \
  -v /opt/nginx/html:/usr/share/nginx/html \
  nginx:latest
```

### 7.1 如何区分数据卷和本地目录

| `-v` 左侧写法 | Docker 的识别方式 |
| --- | --- |
| 以 `/` 开头 | 宿主机绝对路径 |
| 以 `./` 开头 | 宿主机相对路径 |
| 不以 `/` 或 `./` 开头 | Docker 数据卷名称 |

对比：

```bash
-v html-data:/usr/share/nginx/html
# html-data 是 Docker 数据卷

-v /opt/html:/usr/share/nginx/html
# /opt/html 是宿主机本地目录
```

### 7.2 数据卷与本地目录挂载对比

| 对比项 | 数据卷 | 本地目录挂载 |
| --- | --- | --- |
| 管理方 | Docker 管理 | 用户自行管理 |
| 宿主机路径 | Docker 自动决定 | 用户明确指定 |
| 典型用途 | 持久化业务数据 | 挂载配置文件、源码或明确目录 |
| 查看路径 | `docker volume inspect` | 挂载命令中直接可见 |

---

## 八、自定义镜像

### 8.1 镜像的分层结构

Docker 镜像采用分层结构。构建自己的应用镜像时，通常不需要从完全空白的 Linux 环境开始，而是选择一个合适的基础镜像，再逐层添加环境、依赖和应用程序。

```text
应用启动配置层
应用程序层
JRE / 运行环境层
基础 Linux 镜像层
```

合理复用基础层，可以减少重复构建和镜像传输的成本。

### 8.2 Dockerfile

Dockerfile 是一个文本文件，通过一系列指令描述镜像的构建过程。

```text
Dockerfile ──docker build──▶ 自定义镜像 ──docker run──▶ 容器
```

### 8.3 构建命令

```bash
docker build -t 镜像名称:镜像版本 Dockerfile所在目录
```

例如：

```bash
docker build -t myapp:1.0 .
```

- `-t myapp:1.0`：为镜像指定名称和标签；
- `.`：当前目录是构建上下文，Docker 会在此查找 Dockerfile 和构建所需文件。

---

## 九、Dockerfile 常用指令

| 指令 | 作用 | 示例 |
| --- | --- | --- |
| `FROM` | 指定基础镜像 | `FROM centos:7` |
| `ENV` | 设置环境变量，供后续指令或容器使用 | `ENV KEY=value` |
| `COPY` | 将构建上下文中的文件复制到镜像目录 | `COPY jre11.tar.gz /tmp/` |
| `RUN` | 构建镜像时执行 Linux Shell 命令 | `RUN tar -zxvf /tmp/jre11.tar.gz` |
| `EXPOSE` | 声明容器运行时使用的端口 | `EXPOSE 8080` |
| `ENTRYPOINT` | 指定容器启动时执行的命令 | `ENTRYPOINT ["java", "-jar", "app.jar"]` |

### 9.1 `FROM`

指定构建所基于的基础镜像，通常是 Dockerfile 的第一条指令：

```dockerfile
FROM centos:7
```

### 9.2 `ENV`

设置环境变量：

```dockerfile
ENV JAVA_HOME=/opt/jre11
ENV PATH=${JAVA_HOME}/bin:${PATH}
```

> [!NOTE]
> 导图示例中的 `EXPORTS path=...` 不是正确的 Dockerfile 指令。持久设置镜像环境变量应使用 `ENV`；Linux Shell 临时设置变量使用 `export`。

### 9.3 `COPY`

把本地构建上下文中的文件复制到镜像内部：

```dockerfile
COPY jre11.tar.gz /tmp/
```

### 9.4 `RUN`

在构建镜像的过程中执行命令，常用于安装软件或处理文件：

```dockerfile
RUN tar -zxvf /tmp/jre11.tar.gz -C /opt
```

`RUN` 发生在 `docker build` 阶段，不是每次启动容器时执行。

### 9.5 `EXPOSE`

声明容器中的应用预计监听哪个端口：

```dockerfile
EXPOSE 8080
```

`EXPOSE` 主要用于说明镜像的端口信息，不等同于宿主机端口映射。真正映射端口仍需在运行时使用 `-p`：

```bash
docker run -p 8080:8080 myapp:1.0
```

### 9.6 `ENTRYPOINT`

指定容器启动时执行的应用命令：

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

容器启动时会运行该 Java 应用。

---

## 十、完整构建示例

下面是一个用于部署 Spring Boot JAR 包的简化 Dockerfile：

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY app.jar /app/app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

构建镜像：

```bash
docker build -t demo-app:1.0 .
```

运行容器：

```bash
docker run -d \
  --name demo-app \
  -p 8080:8080 \
  demo-app:1.0
```

查看运行状态和日志：

```bash
docker ps
docker logs -f demo-app
```

---

## 十一、知识点速记

> [!TIP]
> - **镜像：** 软件及其依赖组成的静态只读模板，类似 Java 类。
> - **容器：** 镜像运行后的动态实例，类似 Java 对象。
> - **镜像仓库：** 存储和分发镜像，包括中央仓库、私服和本地镜像库。
> - **`docker run`：** 创建并运行容器。
> - **`-d`：** 后台运行容器。
> - **`--name`：** 指定不重复的容器名称。
> - **`-p 主机端口:容器端口`：** 完成端口映射。
> - **数据卷：** Docker 管理的持久化目录，可与容器目录双向绑定。
> - **Bind Mount：** 将明确的宿主机目录挂载到容器。
> - **Dockerfile：** 使用指令描述镜像构建过程。
> - **`docker build -t 名称:标签 .`：** 根据当前目录的 Dockerfile 构建镜像。

### 一句话串联

**Dockerfile 描述如何构建镜像，镜像通过 `docker run` 创建容器，端口映射让外部访问容器服务，数据卷或本地目录挂载则让重要数据独立于容器生命周期持久保存。**
