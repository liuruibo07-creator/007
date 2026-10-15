# NTP 时间同步与 Chrony

> 来源：`05-系统运维-NTP服务.xmind`｜重点：时间一致性、NTP 层级、Chrony 配置与验证。


## 本篇目录

- [一、为什么需要时间同步](#一为什么需要时间同步)
- [二、NTP 与层级结构](#二ntp-与层级结构)
- [三、Chrony 的组件与服务](#三chrony-的组件与服务)
- [四、配置重点](#四配置重点)
- [五、如何确认同步成功](#五如何确认同步成功)
- [六、面试速记](#六面试速记)

---

## 一、为什么需要时间同步

服务器时间偏差会影响日志关联、任务调度、认证以及分布式协作。企业通常让内部服务器向统一时间源同步，内网时间源再向可信上游同步。

```bash
date
date +"%Y-%m-%d %H:%M:%S"
timedatectl
```

时间同步校准的是系统时钟，时区决定显示方式；时区设置正确不代表时钟已同步。

## 二、NTP 与层级结构

NTP（Network Time Protocol）用于网络时间同步，常用 UDP 123 端口。Stratum 表示距参考时钟的逻辑层数，不能直接等同于精度排名。

| 层级 | 含义 |
| --- | --- |
| Stratum 0 | GPS、北斗等参考时钟设备，通常不是直接提供 NTP 的普通服务器 |
| Stratum 1 | 直接连接参考时钟的 NTP 服务器 |
| Stratum 2 | 从 Stratum 1 同步的服务器 |
| Stratum 3 及以下 | 从更上层服务器同步，逐层增加层级 |

分层有助于分散负载并组织同步关系。原图中“二层供公众、三层供企业”是教学示例，实际部署并无这样的固定角色限制。

## 三、Chrony 的组件与服务

Chrony 是 NTP 的一种实现；`chronyd` 是后台服务，`chronyc` 是管理客户端。原图以 CentOS 系列环境为例，常见配置文件为 `/etc/chrony.conf`。

```bash
dnf install -y chrony
systemctl enable --now chronyd
systemctl status chronyd
```

## 四、配置重点

原图重点为时间源、允许的客户端网段和日志。以下为补充的配置示例，需替换上游名称：

```conf
# 向上游同步；替换为企业实际可达的时间源
server ntp.example.internal iburst

# 仅当本机需要向该内网提供 NTP 服务时添加
allow 192.168.88.0/24

# 按需启用日志
logdir /var/log/chrony
log tracking measurements statistics
```

`iburst` 有助于初始同步。客户端不需要为了向上游同步而配置 `allow`；作为 NTP 服务端时，还需允许客户端访问 UDP 123。配置语义参考 [Chrony 官方配置手册](https://chrony-project.org/doc/4.6/chrony.conf.html)。

```bash
systemctl restart chronyd
chronyc sources -v
chronyc tracking
```

## 五、如何确认同步成功

| 检查项 | 关注点 |
| --- | --- |
| `systemctl status chronyd` | 服务是否正常运行 |
| `chronyc sources -v` | 上游是否可达，是否存在被选中的时间源 |
| `chronyc tracking` | 参考源、层级、系统偏差、闰秒状态等 |
| `journalctl -u chronyd` | 配置、解析、访问及同步异常 |

排查顺序：时间源配置 → DNS/网络连通 → UDP 123 访问控制 → 服务状态 → 同步结果。服务已启动与时钟已同步是两个不同状态。

## 六、面试速记

- **NTP 与 Chrony 什么关系？**NTP 是协议，Chrony 是实现软件。
- **chronyd 与 chronyc 有何区别？**前者提供后台服务，后者用于查询和管理。
- **怎样判断真的同步？**结合 sources、tracking 和服务日志判断。
- **内网统一对时怎么做？**配置内部时间源，让业务节点向其同步，内部时间源连接可信上游。

---

[返回总目录](README.md)
