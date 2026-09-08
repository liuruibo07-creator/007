# Day 66 学习笔记：Docker 部署与 TypeScript 入门

> 本篇整理自 `day66.xmind`，主要涵盖 **Docker 自定义网络、前后端项目部署、Docker Compose** 以及 **TypeScript 基础用法**。

---

## 📚 目录

- [一、Docker 自定义网络](day66%20Docker%20部署与%20TypeScript%20入门.md#一docker-自定义网络)
- [二、部署后端项目](day66%20Docker%20部署与%20TypeScript%20入门.md#二部署后端项目)
- [三、部署前端项目](day66%20Docker%20部署与%20TypeScript%20入门.md#三部署前端项目)
- [四、Docker Compose](day66%20Docker%20部署与%20TypeScript%20入门.md#四docker-compose)
- [五、TypeScript 简介](day66%20Docker%20部署与%20TypeScript%20入门.md#五typescript-简介)
- [六、TypeScript 的编写、编译与运行](day66%20Docker%20部署与%20TypeScript%20入门.md#六typescript-的编写编译与运行)
- [七、TypeScript 配置文件](day66%20Docker%20部署与%20TypeScript%20入门.md#七typescript-配置文件)
- [八、TypeScript 自动编译](day66%20Docker%20部署与%20TypeScript%20入门.md#八typescript-自动编译)
- [九、TypeScript 类型声明](day66%20Docker%20部署与%20TypeScript%20入门.md#九typescript-类型声明)
- [十、知识点速查](day66%20Docker%20部署与%20TypeScript%20入门.md#十知识点速查)

---

## 一、Docker 自定义网络

### 1.1 作用

容器加入同一个自定义网络后，可以直接通过 **容器名** 相互访问，不必依赖容易变化的容器 IP 地址。

> 💡 例如：后端容器可以使用数据库容器名作为访问地址，Nginx 也可以通过后端容器名转发请求。

### 1.2 常用命令

| 功能         | 命令                                        |
| ---------- | ----------------------------------------- |
| 创建网络       | `docker network create <网络名称>`            |
| 查看网络列表     | `docker network ls`                       |
| 查看网络详情     | `docker network inspect <网络名称>`           |
| 删除指定网络     | `docker network rm <网络名称>`                |
| 删除所有未使用的网络 | `docker network prune`                    |
| 将已有容器加入网络  | `docker network connect <网络名称> <容器名称>`    |
| 将容器移出网络    | `docker network disconnect <网络名称> <容器名称>` |
```
docker network create sysk
docker network ls
docker network inspect sysk
docker network rm sysk
docker network prune
docker network connect sysyk nginx


```

### 1.3 创建容器时加入网络

在 `docker run` 命令中使用 `--network`：

```bash
docker run --network <网络名称> <镜像名称>
```

示例：

```bash
docker run -d --name app --network sysk app:1.0
```

---

## 二、部署后端项目

### 2.1 上传项目资料

将以下文件上传到服务器的项目目录中：

- 后端项目的 JAR 包
- `Dockerfile`

### 2.2 构建镜像

```bash
docker build -t hmall:1.0 /root/hmall
```

参数说明：

- `-t hmall:1.0`：设置镜像名称和版本标签。
- `/root/hmall`：指定构建上下文，即 `Dockerfile` 和相关文件所在目录。

### 2.3 创建并运行容器

```bash
docker run -d \
  --name hmall \
  -p 8080:8080 \
  --network sysk \
  hmall:1.0
```

| 参数 | 说明 |
| --- | --- |
| `-d` | 在后台运行容器 |
| `--name hmall` | 将容器命名为 `hmall` |
| `-p 8080:8080` | 将宿主机的 8080 端口映射到容器的 8080 端口 |
| `--network sysk` | 将容器加入 `sysk` 网络 |
| `hmall:1.0` | 使用指定镜像创建容器 |

---

## 三、部署前端项目

### 3.1 上传项目资料

需要准备并上传：

- Nginx 相关目录和文件
- `nginx.conf` 配置文件
- 两个已经完成打包的前端项目

### 3.2 创建并运行 Nginx 容器

```bash
docker run -d \
  --name nginx \
  -p 18080:18080 \
  -p 18081:18081 \
  -v /root/nginx/html:/usr/share/nginx/html \
  -v /root/nginx/nginx.conf:/etc/nginx/nginx.conf \
  --network sysk \
  nginx
```

重点说明：

- `-p`：将两个前端服务端口暴露到宿主机。
- 第一个 `-v`：挂载前端静态资源目录。
- 第二个 `-v`：挂载 Nginx 配置文件。
- `--network sysk`：使 Nginx 能通过容器名访问同一网络中的后端服务。

> 📝 原思维导图中的命令在 `--network sysk` 后结束；上例补充了常用的官方镜像名 `nginx`，实际使用时应以项目要求的镜像为准。

---

## 四、Docker Compose

### 4.1 作用

Docker Compose 用于 **批量定义、创建和管理多个容器**，适合部署由前端、后端、数据库等多个服务组成的应用。

### 4.2 `docker-compose.yml` 文件

`docker-compose.yml` 中集中定义多个容器（服务）的配置信息，例如：

- 使用的镜像或构建目录
- 容器名称
- 端口映射
- 数据卷挂载
- 网络配置
- 环境变量
- 服务之间的依赖关系

### 4.3 命令格式

```bash
docker compose [选项] <命令> [命令选项]
```

### 4.4 常用选项

| 选项 | 作用 |
| --- | --- |
| `-f <文件>` | 指定 Compose 文件的路径和名称 |
| `-p <名称>` | 指定项目名称 |
| `-d` | 后台运行，通常与 `up` 搭配使用 |

### 4.5 常用命令

| 命令 | 作用 |
| --- | --- |
| `docker compose up -d` | 创建并在后台启动所有服务容器 |
| `docker compose down` | 停止并移除相关容器和网络 |
| `docker compose ps` | 列出 Compose 项目中的容器 |
| `docker compose logs` | 查看服务日志 |
| `docker compose stop` | 停止服务容器，但不删除 |
| `docker compose start` | 启动已经存在的服务容器 |
| `docker compose restart` | 重启服务容器 |
| `docker compose top` | 查看容器中运行的进程 |
| `docker compose exec <服务名> <命令>` | 在正在运行的服务容器中执行命令 |

> ⚠️ `down` 会移除由 Compose 创建的容器和网络；若同时添加卷相关参数，还可能影响数据卷，执行前应确认数据是否需要保留。

---

## 五、TypeScript 简介

TypeScript（TS）是 JavaScript（JS）的超集：

- 包含 JavaScript 的全部功能。
- 在 JavaScript 基础上增加了类型注解等能力。
- 浏览器和 Node.js 默认不能直接执行 `.ts` 文件。
- TypeScript 代码需要先编译为 JavaScript，再交给浏览器或 Node.js 执行。
- 编译完成后，类型注解等 TypeScript 扩展语法会被擦除，最终生成普通 JavaScript。

```text
TypeScript 源码（.ts） → tsc 编译 → JavaScript 文件（.js） → 浏览器或 Node.js 执行
```

---

## 六、TypeScript 的编写、编译与运行

### 6.1 文件扩展名

TypeScript 文件使用 `.ts` 作为扩展名，例如：

```text
demo.ts
```

### 6.2 安装 TypeScript 编译器

全局安装 TypeScript 5：

```bash
npm install -g typescript@5
```

安装完成后，可使用 `tsc` 命令。

### 6.3 编译 TypeScript

```bash
tsc demo.ts
```

编译成功后会生成对应的 JavaScript 文件，例如 `demo.js`。

### 6.4 运行 JavaScript

```bash
node demo.js
```

---

## 七、TypeScript 配置文件

### 7.1 生成配置文件

在项目根目录执行：

```bash
tsc --init
```

该命令会生成默认的 `tsconfig.json`。

### 7.2 主要修改项

1. 启用 `rootDir` 和 `outDir`，并在项目根目录创建 `src`、`dist` 目录。
2. 将 `demo.ts` 移动到 `src` 目录中。
3. 将 `module` 设置为 `es6`。
4. 将 `target` 设置为 `es6`。
5. 注释 `declaration`、`declarationMap`、`sourceMap`，保证只生成 JavaScript 文件。
6. 添加 `"noEmitOnError": true`，使 TypeScript 代码存在错误时不生成 JavaScript。

配置示例：

```json
{
  "compilerOptions": {
    "rootDir": "./src",
    "outDir": "./dist",
    "module": "es6",
    "target": "es6",
    "noEmitOnError": true
  }
}
```

推荐的目录结构：

```text
项目根目录/
├── src/
│   └── demo.ts
├── dist/
│   └── demo.js
└── tsconfig.json
```

---

## 八、TypeScript 自动编译

使用以下命令监听文件变化并自动编译：

```bash
tsc --watch
```

开启后，只要修改并保存 TypeScript 文件，编译器就会根据 `tsconfig.json` 自动重新编译。

> 💡 自动编译适合开发阶段；终端需要保持运行，按 `Ctrl + C` 可停止监听。

---

## 九、TypeScript 类型声明

### 9.1 变量类型声明

先声明，后赋值：

```typescript
let a: number;
a = 10;
```

声明并赋值：

```typescript
let b: number = 20;
```

其中，冒号后的 `number` 表示变量只能保存数字类型的数据。

### 9.2 函数类型声明

函数的形参和返回值都可以声明数据类型：

```typescript
function sum(a: number, b: number): number {
  return a + b;
}
```

说明：

- `a: number`、`b: number`：两个形参都必须是数字。
- 括号后的 `: number`：函数返回值必须是数字。

---

## 十、知识点速查

| 场景 | 核心命令或配置 |
| --- | --- |
| 创建 Docker 网络 | `docker network create sysk` |
| 容器加入网络 | `--network sysk` |
| 构建后端镜像 | `docker build -t hmall:1.0 /root/hmall` |
| 后台启动 Compose 服务 | `docker compose up -d` |
| 停止并移除 Compose 服务 | `docker compose down` |
| 安装 TypeScript 5 | `npm install -g typescript@5` |
| 初始化 TS 配置 | `tsc --init` |
| 编译单个 TS 文件 | `tsc demo.ts` |
| 开启自动编译 | `tsc --watch` |
| TS 源码目录 | `rootDir: "./src"` |
| JS 输出目录 | `outDir: "./dist"` |
| 编译错误时不输出 JS | `noEmitOnError: true` |

---

> ✅ **本节重点：** 使用 Docker 自定义网络实现容器间通信，通过 Docker Compose 统一管理多容器应用；掌握 TypeScript 从编写、配置、编译到运行的基本流程，并学会为变量、函数参数和返回值添加类型声明。
