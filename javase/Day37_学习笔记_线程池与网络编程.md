# 📘 Day 37 学习笔记：线程池 与 网络编程基础

> 涵盖内容：线程池、并发与并行、线程生命周期、网络编程三要素（IP/端口/协议）、UDP 编程

---

## 目录

- [一、线程池](#一线程池)
- [二、并发和并行](#二并发和并行)
- [三、线程生命周期](#三线程生命周期)
- [四、网络编程概述](#四网络编程概述)
- [五、网络编程三要素](#五网络编程三要素)
- [六、UDP 编程](#六udp-编程)
- [七、知识速查表](#七知识速查表)

---

## 一、线程池

### 1.1 什么是线程池

> **装线程的池子**，采用**池化思想**，对线程进行**复用**，避免频繁创建、销毁线程带来的性能开销。

### 1.2 创建方式

| 方式 | 说明 | 是否推荐 |
|---|---|---|
| **方式一** | 使用 `ThreadPoolExecutor` 创建（`ExecutorService` 接口） | ✅ **推荐** |
| **方式二** | 使用 `Executors`（线程池工具类）创建 | ⚠️ 不推荐（底层参数不透明，易导致 OOM） |

### 1.3 ThreadPoolExecutor 的七大核心参数

| 参数 | 含义 |
|---|---|
| `corePoolSize` | **核心**线程数 |
| `maximumPoolSize` | **最大**线程数 |
| `keepAliveTime` | 临时线程**空闲等待时间**（数值） |
| `unit` | 临时线程空闲等待时间（**单位**） |
| `workQueue` | **任务队列** |
| `threadFactory` | 创建线程的**工厂** |
| `handler` | **拒绝策略** |

```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    corePoolSize,      // 核心线程数
    maximumPoolSize,   // 最大线程数
    keepAliveTime,     // 空闲等待时间数值
    unit,              // 空闲等待时间单位
    workQueue,         // 任务队列
    threadFactory,     // 线程工厂
    handler            // 拒绝策略
);
```

### 1.4 提交任务的两种方式

| 方法 | 任务类型 | 是否有返回值 |
|---|---|---|
| `void execute(Runnable command)` | `Runnable` 任务 | ❌ 无 |
| `Future<T> submit(Callable<T> task)` | `Callable` 任务 | ✅ 有（通过 `Future` 获取） |

### 1.5 线程池的工作原理

```
任务提交后的执行流程
├── 1️⃣ 任务数 ≤ 核心线程数        → 由核心线程直接执行
├── 2️⃣ 任务数 > 核心线程数        → 进入任务队列排队
├── 3️⃣ 任务队列也满了              → 创建临时线程执行任务
└── 4️⃣ 临时线程也满 + 队列也满     → 触发拒绝策略
```

> 💡 执行顺序：**核心线程 → 任务队列 → 临时线程 → 拒绝策略**，是"能省则省、能排就排、实在不行才扩容、扩无可扩才拒绝"的思路。

### 1.6 拒绝策略

| 策略 | 行为 |
|---|---|
| `AbortPolicy` | **默认策略**：丢弃任务，并**抛出异常** |
| `DiscardPolicy` | 丢弃任务，**不抛异常** |
| `DiscardOldestPolicy` | 丢弃队列中**等待最久**的任务，再尝试提交新任务 |
| `CallerRunsPolicy` | 由**调用者所在的线程**来执行该任务 |

### 1.7 Executors 工具类创建线程池

> Executors 创建的各类线程池，**底层都是基于 `ThreadPoolExecutor`** 实现的。

以 `newScheduledThreadPool`（定时任务线程池）为例：

| 方法 | 作用 | 类比 |
|---|---|---|
| `schedule(Runnable command, long delay, TimeUnit unit)` | 延迟 `delay` 时间后执行一次任务 | 类似 JS 的 `setTimeout()` |
| `scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)` | 延迟 `initialDelay` 后执行第一次，之后每隔 `period` 重复执行 | 类似 JS 的 `setInterval()` |

---

## 二、并发和并行

| 概念 | 定义 |
|---|---|
| **并发** | 多个线程在**单核 CPU** 上**交替**执行 |
| **并行** | 多个线程在**多核 CPU** 上**同时**执行 |

```
并发 vs 并行
├── 并发：单核 CPU，看似同时，实则快速切换（交替）
└── 并行：多核 CPU，真正意义上的同时执行
```

---

## 三、线程生命周期

线程生命周期是指：**线程从创建到销毁的整个过程**，其中包含 **6 种状态**之间的相互转换。

> 📌 该部分笔记原文仅列出要点框架（线程 6 种状态及其转换关系），具体状态图建议结合 `Thread.State` 枚举（NEW / RUNNABLE / BLOCKED / WAITING / TIMED_WAITING / TERMINATED）进一步学习。

---

## 四、网络编程概述

### 4.1 什么是网络编程

> 用来实现**计算机网络中两台设备之间进行数据通信**的编程方式。

### 4.2 两种通信架构

| 架构 | 全称 | 说明 |
|---|---|---|
| **C/S** | Client / Server（客户端/服务器） | 需要单独开发客户端程序 |
| **B/S** | Browser / Server（浏览器/服务器） | 直接使用浏览器作为客户端 |

---

## 五、网络编程三要素

```
网络编程三要素
├── IP    → 计算机在网络中的唯一标识
├── 端口  → 应用程序在计算机中的唯一标识
└── 协议  → 通信双方都必须遵守的规则
```

### 5.1 IP

| 版本 | 长度 | 表示方式 |
|---|---|---|
| **IPv4** | 4 个字节 | **点分十进制**（如 `192.168.1.1`） |
| **IPv6** | 16 个字节 | **冒分十六进制** |

**`InetAddress` 类（表示 IP 地址）常用方法：**

| 方法 | 作用 |
|---|---|
| `static InetAddress getLocalHost()` | 获取**本地** IP |
| `static InetAddress getByName(String ip/域名)` | 获取**指定** IP |
| `String getHostName()` | 获取主机名 / 域名 |
| `String getHostAddress()` | 获取 IP 地址 |
| `boolean isReachable(int 毫秒值)` | 判断本地与指定 IP 是否**互通** |

> 🏠 **本机 IP** 有两种常见写法：`127.0.0.1` 或 `localhost`。

### 5.2 端口

| 项目 | 内容 |
|---|---|
| 取值范围 | `0` ~ `65535` |
| 唯一性要求 | 同一台机器上，**两个应用程序端口不能相同** |

### 5.3 协议

**TCP/IP 四层模型：**

| 层级 | 对应协议 |
|---|---|
| 应用层 | `HTTP` |
| 传输层 | `TCP` / `UDP` |
| 网络层 | `IP` |
| 网络接口层 | （物理层相关） |

**UDP vs TCP 对比：**

| 协议 | 特点 |
|---|---|
| **UDP** | **无连接**，**不可靠**传输 |
| **TCP** | **面向连接**，**可靠**传输 |

**TCP 建立与断开连接：**

| 过程 | 次数 | 作用 |
|---|---|---|
| **三次握手** | 3 次 | 确保通信双方**收发能力**都正常 |
| **四次挥手** | 4 次 | 确保通信双方**收发完成**，并**正常断开**连接 |

---

## 六、UDP 编程

### 6.1 客户端

| 步骤 | 方法 / 代码 |
|---|---|
| 创建 | `new DatagramSocket()` |
| 封装数据包 | `new DatagramPacket(bytes, bytes.length, InetAddress.getLocalHost(), 10010)` |
| 发送 | `send(datagramPacket)` |
| 关闭 | `close()` |

> `DatagramPacket` 四个参数含义：**① 要发送内容的字节数组 ② 发送数组的长度 ③ 服务端 IP ④ 服务端端口**

```java
// UDP 客户端示例
DatagramSocket socket = new DatagramSocket();
byte[] bytes = "hello".getBytes();
DatagramPacket packet = new DatagramPacket(
    bytes, bytes.length, InetAddress.getLocalHost(), 10010
);
socket.send(packet);
socket.close();
```

### 6.2 服务端

| 步骤 | 方法 / 代码 |
|---|---|
| 创建 | `new DatagramSocket(10010)`（绑定端口） |
| 封装数据包（用于接收） | `new DatagramPacket(buf, buf.length)` |
| 接收 | `receive(datagramPacket)` |
| 关闭 | `close()` |

> 服务端 `DatagramPacket` 两个参数含义：**① 用来接收数据的字节数组 ② 用多大容量来接收数据**

```java
// UDP 服务端示例
DatagramSocket socket = new DatagramSocket(10010);
byte[] buf = new byte[1024];
DatagramPacket packet = new DatagramPacket(buf, buf.length);
socket.receive(packet);
socket.close();
```

### 6.3 客户端 vs 服务端对比

| 对比项 | 客户端 | 服务端 |
|---|---|---|
| 创建 Socket | `new DatagramSocket()`（不绑定端口） | `new DatagramSocket(端口)`（绑定固定端口） |
| 核心动作 | `send()` | `receive()` |
| DatagramPacket 用途 | 携带发送内容 + 目标地址端口 | 携带接收缓冲区 |

> 💡 **如何多开客户端**：多个客户端各自 `new DatagramSocket()`（系统自动分配不同的临时端口），都可以向同一个服务端端口发送数据。

---

## 七、知识速查表

| 主题 | 核心内容 | 一句话总结 |
|---|---|---|
| 线程池 | `ThreadPoolExecutor` 七参数 | 核心线程→队列→临时线程→拒绝策略，四级处理流程 |
| 拒绝策略 | 4种策略 | 默认 AbortPolicy 会抛异常，其余三种各有取舍 |
| 并发/并行 | 单核交替 vs 多核同时 | 并发是"看起来同时"，并行是"真正同时" |
| 网络编程三要素 | IP / 端口 / 协议 | IP找设备，端口找应用，协议定规则 |
| IP | IPv4/IPv6 | IPv4是4字节点分十进制，IPv6是16字节冒分十六进制 |
| 协议 | TCP vs UDP | TCP面向连接可靠（三次握手/四次挥手），UDP无连接不可靠 |
| UDP编程 | DatagramSocket/DatagramPacket | 客户端send，服务端receive，数据包是核心载体 |

---

## 🌳 整体知识树

```
Day37
├── 线程池
│   ├── 概念：池化思想，复用线程
│   ├── 创建方式：ThreadPoolExecutor（推荐）/ Executors
│   ├── 七大参数：core/max/keepAliveTime/unit/queue/factory/handler
│   ├── 提交任务：execute(Runnable) / submit(Callable)→Future
│   ├── 工作原理：核心线程→任务队列→临时线程→拒绝策略
│   └── 拒绝策略：Abort(默认,抛异常) / Discard / DiscardOldest / CallerRuns
│
├── 并发和并行
│   ├── 并发：单核CPU交替执行
│   └── 并行：多核CPU同时执行
│
├── 线程生命周期
│   └── 创建→销毁，6种状态相互转换
│
├── 网络编程概述
│   ├── 定义：设备间数据通信
│   └── 架构：C/S（客户端服务器）/ B/S（浏览器服务器）
│
├── 网络编程三要素
│   ├── IP：唯一标识设备（IPv4/IPv6，InetAddress类）
│   ├── 端口：唯一标识应用（0-65535，不可重复）
│   └── 协议：通信规则
│       ├── TCP/IP四层模型：应用层/传输层/网络层/网络接口层
│       ├── UDP：无连接，不可靠
│       └── TCP：面向连接，可靠（三次握手/四次挥手）
│
└── UDP编程
    ├── 客户端：DatagramSocket() → send() → close()
    └── 服务端：DatagramSocket(端口) → receive() → close()
```
