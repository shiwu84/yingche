---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, systemctl]
source: https://wiki.archlinux.org/title/Systemd
aliases: [systemctl 命令，systemd 服务管理]
---

# Systemd 服务管理

> [!summary] 核心概念
> systemctl 是管理 systemd 的主要命令，用于检查系统状态和管理服务。

## 基本用法

systemctl 用于：
- 检查系统状态
- 管理系统和服务
- 启动、停止、重启服务
- 启用、禁用开机启动

参考：[systemctl(1)](https://man.archlinux.org/man/systemctl.1)

## 分析系统状态

### 显示系统状态

```bash
systemctl status
```

显示系统整体状态，包括：
- 运行时间
- 启动时间
- 运行的 units 数量
- 失败的 units

### 列出运行中的 Units

```bash
systemctl
# 或
systemctl list-units
```

### 列出失败的 Units

```bash
systemctl --failed
```

快速查看哪些服务启动失败。

### 列出已安装的 Unit 文件

```bash
systemctl list-unit-files
```

显示所有可用的 unit 文件及其状态。

### 查看进程状态

```bash
systemctl status PID
```

查看特定 PID 的进程状态，包括 cgroup、内存和父进程。

## 检查 Unit 状态

### 查看 Unit 状态

```bash
systemctl status unit_name
```

**示例：**
```bash
systemctl status sshd.service
systemctl status NetworkManager
```

显示：
- 是否正在运行
- 是否启用
- 最近的日志
- 进程树

### 查看帮助手册

```bash
systemctl help unit_name
```

显示 unit 相关的手册页（如果支持）。

### 检查是否启用

```bash
systemctl is-enabled unit_name
```

返回：
- `enabled` - 已启用
- `disabled` - 已禁用
- `static` - 静态（不能被启用）
- `masked` - 已屏蔽

## 启动、停止、重启服务

### 启动服务

```bash
sudo systemctl start unit_name
```

**示例：**
```bash
sudo systemctl start sshd.service
```

### 停止服务

```bash
sudo systemctl stop unit_name
```

### 重启服务

```bash
sudo systemctl restart unit_name
```

### 重新加载配置

```bash
sudo systemctl reload unit_name
```

不中断服务，重新加载配置文件。

### 重新加载 systemd 配置

```bash
sudo systemctl daemon-reload
```

扫描新的或更改的 unit 文件。

> 💡 **Tip**: 修改 unit 文件后必须运行此命令。

## 启用开机启动

### 启用服务

```bash
sudo systemctl enable unit_name
```

创建符号链接，使服务在启动时自动运行。

### 立即启用并启动

```bash
sudo systemctl enable --now unit_name
```

同时启用和启动服务。

### 禁用服务

```bash
sudo systemctl disable unit_name
```

移除符号链接，服务不再自动启动。

### 禁用并停止

```bash
sudo systemctl disable --now unit_name
```

同时禁用和停止服务。

### 重新启用

```bash
sudo systemctl reenable unit_name
```

先禁用再启用，适用于 Install 部分更改后。

## 屏蔽服务

### 屏蔽服务

```bash
sudo systemctl mask unit_name
```

使服务无法启动（手动或作为依赖）。

> ⚠️ **Warning**: 屏蔽很危险，会阻止所有启动方式。

### 取消屏蔽

```bash
sudo systemctl unmask unit_name
```

### 查看屏蔽的服务

```bash
systemctl list-unit-files --state=masked
```

## Unit 命名规则

### 完整名称

通常需要指定完整名称（包括后缀）：
```bash
systemctl status sshd.socket
```

### 简写规则

**1. 省略 .service**
```bash
systemctl start sshd
# 等同于
systemctl start sshd.service
```

**2. 挂载点自动转换**
```bash
systemctl status /home
# 等同于
systemctl status home.mount
```

**3. 设备自动转换**
```bash
systemctl status /dev/sda2
# 等同于
systemctl status dev-sda2.device
```

## 模板 Units

### 实例化模板

模板 unit 包含 `@` 符号：
```
name@.service
```

使用时指定实例名：
```bash
systemctl start name@instance.service
```

**示例：**
```bash
# 模板文件：getty@.service
# 启动 tty1 的 getty
systemctl start getty@tty1.service
```

### 模板参数

在 unit 文件中，`%i` 会被实例名替换。

## 系统 Units vs 用户 Units

### 系统 Units（默认）

```bash
sudo systemctl start sshd.service
```

管理整个系统的服务，需要 root 权限。

### 用户 Units

```bash
systemctl --user start my-service.service
```

管理当前用户的服务，不需要 root 权限。

参考：[systemctl --user](https://wiki.archlinux.org/title/Systemctl_--user)

## 实用技巧

### 1. 使用 -H 远程管理

```bash
systemctl -H user@host status sshd.service
```

通过 SSH 管理远程 systemd。

### 2. 同时操作多个 Units

```bash
sudo systemctl start nginx php-fpm
```

### 3. 查找包的 Units

```bash
pacman -Qql package_name | grep -E '\.service|\.socket'
```

查看包提供了哪些 service 或 socket。

### 4. 使用 --now 开关

```bash
# 启用并立即启动
sudo systemctl enable --now sshd.service

# 禁用并立即停止
sudo systemctl disable --now bluetooth.service
```

## 常用命令速查

| 操作 | 命令 |
|------|------|
| 查看状态 | `systemctl status unit` |
| 启动服务 | `sudo systemctl start unit` |
| 停止服务 | `sudo systemctl stop unit` |
| 重启服务 | `sudo systemctl restart unit` |
| 重载配置 | `sudo systemctl reload unit` |
| 启用开机启动 | `sudo systemctl enable unit` |
| 禁用开机启动 | `sudo systemctl disable unit` |
| 屏蔽服务 | `sudo systemctl mask unit` |
| 列出失败服务 | `systemctl --failed` |
| 重新加载 systemd | `sudo systemctl daemon-reload` |

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-基本概念]] - systemd 架构
- [[Systemd-Unit 文件]] - 编写 unit 文件
- [[Systemd-电源管理]] - 电源控制命令
