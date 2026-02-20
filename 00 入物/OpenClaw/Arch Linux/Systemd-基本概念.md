---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, basics]
source: https://wiki.archlinux.org/title/Systemd
aliases: [systemd 介绍，什么是 systemd]
---

# Systemd 基本概念

> [!summary] 核心概念
> Systemd 是 Linux 系统的基础构建块，提供系统和服务管理器，作为 PID 1 运行并启动系统的其余部分。

## 什么是 Systemd

Systemd 是一个 suite of basic building blocks，提供：

- **系统和服务管理器** - 作为 PID 1 运行，启动系统的其余部分
- **日志守护进程** - journald 记录系统日志
- **电源管理** - 控制系统关机、重启、休眠等
- **网络配置** - networkd 管理简单网络配置
- **时间同步** - timesyncd 同步系统时间
- **用户会话管理** - logind 管理登录会话

## 历史背景

> 💡 **Note**: systemd 支持 SysV 和 LSB init 脚本，作为 sysvinit 的替代品。

Arch Linux 迁移到 systemd 的详细解释见：[Arch 论坛帖子](https://bbs.archlinux.org/viewtopic.php?pid=1149530#p1149530)

## 什么是 Daemon

历史上，systemd 称为"service"的曾被称为 **daemon**（守护进程）：

- 任何作为"后台"进程运行的程序（没有终端或用户界面）
- 通常等待事件发生并提供服务

**示例：**
- Web 服务器 - 等待请求并提供页面
- SSH 服务器 - 等待用户登录
- 日志写入 - 将消息写入日志文件（syslog、metalog）
- 时间同步 - 保持系统时间准确（ntpd）

参考：[daemon(7)](https://man.archlinux.org/man/daemon.7)

## 核心特性

### 1. 激进的并行化能力

systemd 可以同时启动多个服务，显著加快启动速度。

### 2. Socket 和 D-Bus 激活

- **Socket 激活** - 服务在需要时自动启动
- **D-Bus 激活** - 通过 D-Bus 消息触发服务启动

### 3. 按需启动守护进程

服务仅在需要时启动，节省系统资源。

### 4. 控制组（cgroups）

使用 Linux 控制组跟踪进程：

- 跟踪进程树
- 资源限制
- 进程隔离

参考：[Control groups](https://wiki.archlinux.org/title/Control_groups)

### 5. 挂载点管理

- 维护挂载点（mount points）
- 自动挂载点（automount points）

### 6. 事务性依赖控制

复杂的基于依赖的服务控制逻辑。

## 其他组件

Systemd 还包括：

| 组件 | 功能 |
|------|------|
| hostnamectl | 控制主机名 |
| timedatectl | 控制日期和时间 |
| localectl | 控制本地化设置 |
| loginctl | 管理登录会话 |
| systemd-networkd | 简单网络配置 |
| systemd-resolved | 名称解析 |
| systemd-journald | 日志记录 |
| systemd-tmpfiles | 管理临时文件 |
| systemd-timers | 定时任务 |

## Units 是什么

**Unit** 是 systemd 管理的基本单元，类型包括：

- **Service** (`.service`) - 系统服务
- **Mount** (`.mount`) - 挂载点
- **Device** (`.device`) - 设备
- **Socket** (`.socket`) - 套接字
- **Target** (`.target`) - 启动目标
- **Timer** (`.timer`) - 定时任务

## Unit 文件位置

Systemd 从多个位置加载 unit 文件（优先级从低到高）：

1. `/usr/lib/systemd/system/` - 已安装包提供的 units
2. `/etc/systemd/system/` - 系统管理员安装的 units

> 💡 **Note**: 用户模式（user mode）的加载路径完全不同。

## 主要命令

管理 systemd 的主要命令是 **systemctl**：

```bash
# 查看系统状态
systemctl status

# 列出运行中的 units
systemctl list-units

# 查看服务状态
systemctl status sshd.service

# 启动服务
sudo systemctl start sshd.service

# 启用开机启动
sudo systemctl enable sshd.service
```

## 远程管理

可以使用 SSH 远程管理 systemd：

```bash
systemctl -H user@host status sshd.service
```

这会连接到远程机器的 systemd 实例。

## 图形界面

KDE Plasma 用户可以安装 systemdgenie：

```bash
sudo pacman -S systemdgenie
```

作为 systemctl 的图形前端。

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-服务管理]] - systemctl 命令详解
- [[Systemd-Unit 文件]] - Unit 文件结构

## 外部资源

- [systemd 官方网站](https://systemd.io/)
- [ArchWiki - Systemd](https://wiki.archlinux.org/title/Systemd)
- [systemd(1) man page](https://man.archlinux.org/man/systemd.1)
