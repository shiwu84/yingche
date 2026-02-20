---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, service-type]
source: https://wiki.archlinux.org/title/Systemd
aliases: [systemd 服务类型，Type=simple, Type=forking]
---

# Systemd 服务类型

> [!summary] 核心概念
> Type= 参数定义服务的启动方式，选择合适的类型对服务管理至关重要。

## Type=simple（默认）

### 特点

- systemd 认为服务立即启动完成
- 进程**不能 fork**
- 主进程保持前台运行

### 适用场景

- 现代守护进程
- 不 fork 的应用程序
- 容器化应用

### 配置示例

```ini
[Service]
Type=simple
ExecStart=/usr/bin/my-service
```

### 注意事项

> ⚠️ **Warning**: 如果其他服务需要按顺序启动，不要使用此类型（除非是 socket 激活）。

### 典型应用

- Node.js 应用
- Python Flask/Django
- Go 程序
- Rust 服务

## Type=forking

### 特点

- systemd 认为服务在**fork 且父进程退出**后启动完成
- 传统 Unix daemon 的标准行为
- 需要指定 `PIDFile=`

### 适用场景

- 传统 Unix 守护进程
- 自行 fork 的应用程序
- 老式服务

### 配置示例

```ini
[Service]
Type=forking
PIDFile=/var/run/my-service.pid
ExecStart=/usr/bin/my-service start
ExecStop=/usr/bin/my-service stop
```

### 注意事项

- **必须**指定 `PIDFile=`，让 systemd 跟踪主进程
- 除非知道不必要，否则传统 daemon 使用此类型

### 典型应用

- Apache httpd
- Nginx（旧版本）
- 传统 init 脚本迁移的服务

## Type=oneshot

### 特点

- 适用于执行**单次任务**后退出的脚本
- 通常配合 `RemainAfterExit=yes`
- 不保持运行状态

### 适用场景

- 初始化脚本
- 数据库迁移
- 系统配置脚本
- 一次性任务

### 配置示例

```ini
[Service]
Type=oneshot
ExecStart=/usr/local/bin/setup-database.sh
RemainAfterExit=yes
```

### RemainAfterExit

| 值 | 行为 |
|------|------|
| `yes` | 进程退出后仍认为服务 active |
| `no`（默认） | 进程退出后认为服务 inactive |

**适用场景：**
- 改变系统状态的脚本（如挂载分区）
- 需要被其他服务依赖的初始化脚本

### 典型应用

- 数据库初始化
- 文件系统挂载
- 网络配置
- 系统清理脚本

## Type=notify

### 特点

- 与 `Type=simple` 相同
- 守护进程会在就绪时**发送信号**给 systemd
- 使用 libsystemd-daemon.so

### 适用场景

- 需要精确控制启动时机的服务
- 复杂的初始化过程
- 需要通知 systemd 就绪状态的服务

### 配置示例

```ini
[Service]
Type=notify
ExecStart=/usr/bin/my-service
# 服务代码中需要调用 sd_notify()
```

### 代码示例（C 语言）

```c
#include <systemd/sd-daemon.h>

// 初始化完成后
sd_notify(0, "READY=1");
```

### 典型应用

- 需要复杂初始化的服务
- 数据库服务（如 PostgreSQL）
- 消息队列（如 RabbitMQ）

## Type=dbus

### 特点

- 服务在指定 **DBus 名称出现**时被认为就绪
- 自动等待 DBus 注册完成

### 适用场景

- 提供 DBus 接口的服务
- 需要 DBus 激活的服务

### 配置示例

```ini
[Service]
Type=dbus
BusName=org.example.MyService
ExecStart=/usr/bin/my-dbus-service
```

### 典型应用

- DBus 服务
- 桌面环境组件
- 系统托盘应用

## Type=idle

### 特点

- 延迟执行，直到**所有作业分发完成**
- 如果延迟超过 5 秒，无论如何都会启动
- 行为类似 `Type=simple`

### 适用场景

- 避免控制台输出混乱
- 不重要的后台服务
- 不希望干扰启动输出的服务

### 配置示例

```ini
[Service]
Type=idle
ExecStart=/usr/bin/background-task
```

### 注意事项

> ⚠️ **Warning**: 永远不要用于服务排序，仅用于帮助控制台输出可读性。

## 类型对比

| Type | Fork | 就绪信号 | 适用场景 |
|------|------|---------|---------|
| `simple` | ❌ | 立即 | 现代服务 |
| `forking` | ✅ | 父进程退出 | 传统 daemon |
| `oneshot` | - | 进程退出 | 一次性任务 |
| `notify` | ❌ | sd_notify() | 复杂初始化 |
| `dbus` | - | DBus 名称 | DBus 服务 |
| `idle` | ❌ | 作业完成后 | 后台任务 |

## 选择指南

### 快速决策树

```
服务是否 fork？
├── 是 → Type=forking（指定 PIDFile）
└── 否 → 服务是否需要精确控制？
    ├── 是，有复杂初始化 → Type=notify
    ├── 是，提供 DBus → Type=dbus
    ├── 否，一次性任务 → Type=oneshot
    └── 否，普通服务 → Type=simple（默认）
```

### 推荐实践

1. **新服务** - 使用 `Type=simple`，不要 fork
2. **传统 daemon** - 使用 `Type=forking` + `PIDFile=`
3. **脚本** - 使用 `Type=oneshot` + `RemainAfterExit=yes`
4. **复杂服务** - 使用 `Type=notify` 精确控制

## 调试服务类型

### 查看服务状态

```bash
systemctl status service_name
```

### 检查启动时间

```bash
systemd-analyze blame | grep service_name
```

### 查看日志

```bash
journalctl -u service_name -f
```

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-Unit 文件]] - Unit 文件配置
- [[Systemd-服务管理]] - systemctl 命令

## 外部资源

- [systemd.service(5) § OPTIONS](https://man.archlinux.org/man/systemd.service.5#OPTIONS)
- [systemd-analyze(1)](https://man.archlinux.org/man/systemd-analyze.1)
- [Simple vs Oneshot](https://trstringer.com/simple-vs-oneshot-systemd-service/)
