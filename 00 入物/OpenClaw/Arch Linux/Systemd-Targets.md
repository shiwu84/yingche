---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, targets]
source: https://wiki.archlinux.org/title/Systemd
aliases: [systemd targets, 启动目标]
---

# Systemd Targets

> [!summary] 核心概念
> Target 是一组 units 的集合，用于定义系统状态和启动阶段。

## 什么是 Target

Target 是 systemd 的特殊 unit 类型，用于：
- 分组相关的 units
- 定义系统启动阶段
- 管理系统状态

**文件后缀：** `.target`

## 常用 Targets

### 系统状态 Targets

| Target | 说明 | 相当于 SysV |
|--------|------|-----------|
| `default.target` | 系统默认 target | - |
| `multi-user.target` | 多用户文本模式 | runlevel 3 |
| `graphical.target` | 图形界面模式 | runlevel 4 |
| `rescue.target` | 救援模式 | runlevel 1 |
| `emergency.target` | 紧急模式 | - |

### 启动阶段 Targets

| Target | 说明 |
|--------|------|
| `sysinit.target` | 系统初始化完成 |
| `basic.target` | 基本系统就绪 |
| `network.target` | 网络已配置 |
| `network-online.target` | 网络已连接 |
| `local-fs.target` | 本地文件系统已挂载 |
| `remote-fs.target` | 远程文件系统已挂载 |

### 特殊 Targets

| Target | 说明 |
|--------|------|
| `shutdown.target` | 关机 |
| `reboot.target` | 重启 |
| `poweroff.target` | 断电 |
| `hibernate.target` | 休眠 |
| `sleep.target` | 睡眠 |
| `suspend.target` | 挂起 |

## 查看 Targets

### 查看默认 Target

```bash
systemctl get-default
```

### 查看所有 Targets

```bash
systemctl list-units --type=target
```

### 查看 Target 依赖

```bash
systemctl list-dependencies multi-user.target
```

### 查看 Target 包含的 Units

```bash
systemctl show -p Wants multi-user.target
```

## 设置默认 Target

### 更改默认启动目标

```bash
# 设置为多用户文本模式
sudo systemctl set-default multi-user.target

# 设置为图形界面
sudo systemctl set-default graphical.target
```

这会创建符号链接：
```
/etc/systemd/system/default.target → /usr/lib/systemd/system/multi-user.target
```

### 临时更改 Target

```bash
# 重启到救援模式
sudo systemctl rescue

# 重启到紧急模式
sudo systemctl emergency
```

## 切换 Target

### 立即切换到 Target

```bash
# 切换到多用户模式
sudo systemctl isolate multi-user.target

# 切换到图形模式
sudo systemctl isolate graphical.target
```

> ⚠️ **Warning**: 这会停止不在新 target 中的所有服务。

### 救援模式

```bash
sudo systemctl rescue
```

进入单用户救援模式，用于系统维护。

### 紧急模式

```bash
sudo systemctl emergency
```

进入最基础的紧急 shell，用于系统修复。

## Target 依赖关系

### 典型启动流程

```
sysinit.target
    ↓
basic.target
    ↓
multi-user.target (或 graphical.target)
    ↓
default.target
```

### graphical.target 依赖

```ini
[Unit]
Description=Graphical Interface
Requires=multi-user.target
After=multi-user.target
Wants=display-manager.service
```

### multi-user.target 依赖

```ini
[Unit]
Description=Multi-User System
Requires=basic.target
After=basic.target
```

## 自定义 Target

### 创建 Target

```ini
# /etc/systemd/system/my-target.target
[Unit]
Description=My Custom Target
Requires=multi-user.target
After=multi-user.target

[Install]
WantedBy=default.target
```

### 启用自定义 Target

```bash
sudo systemctl enable my-target.target
```

## Target vs Runlevel

### SysV Runlevel

| Runlevel | 说明 |
|----------|------|
| 0 | 关机 |
| 1 | 救援模式 |
| 2 | 多用户（无网络） |
| 3 | 多用户（有网络） |
| 4 | 未使用 |
| 5 | 图形界面 |
| 6 | 重启 |

### systemd 兼容性

systemd 提供兼容命令：

```bash
# 查看 runlevel
runlevel

# 更改 runlevel
sudo telinit 3  # 相当于 multi-user.target
sudo telinit 5  # 相当于 graphical.target
```

### 映射关系

| Runlevel | systemd Target |
|----------|---------------|
| 0 | `poweroff.target` |
| 1 | `rescue.target` |
| 3 | `multi-user.target` |
| 5 | `graphical.target` |
| 6 | `reboot.target` |

## 常见场景

### 场景 1：服务器无需图形界面

```bash
sudo systemctl set-default multi-user.target
```

### 场景 2：桌面需要图形界面

```bash
sudo systemctl set-default graphical.target
```

### 场景 3：临时进入救援模式

```bash
sudo systemctl rescue
```

### 场景 4：调试启动问题

在 GRUB 内核参数中添加：
```
systemd.unit=rescue.target
```

## 调试 Targets

### 查看启动时间

```bash
systemd-analyze
```

### 查看各阶段时间

```bash
systemd-analyze blame
```

### 查看关键链

```bash
systemd-analyze critical-chain
```

### 生成依赖图

```bash
systemd-analyze dot | dot -Tsvg > boot.svg
```

## Target 文件示例

### multi-user.target

```ini
[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
After=basic.target
Conflict=rescue.target rescue.service
AllowIsolate=yes

[Install]
WantedBy=default.target
```

### graphical.target

```ini
[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
After=multi-user.target
AllowIsolate=yes

[Install]
WantedBy=default.target
```

## 最佳实践

### 1. 使用标准 Targets

```ini
# ✅ 推荐
WantedBy=multi-user.target

# ❌ 避免
WantedBy=custom-target.target
```

### 2. 理解依赖关系

```ini
# ✅ 推荐：明确依赖
Requires=multi-user.target
After=multi-user.target

# ❌ 避免：只指定一个
Requires=multi-user.target
```

### 3. 不要过度使用 Target

- 普通服务使用 `multi-user.target` 或 `graphical.target`
- 仅在需要定义新状态时创建自定义 target

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-依赖管理]] - 依赖关系配置
- [[Systemd-服务管理]] - systemctl 命令

## 外部资源

- [systemd.special(7)](https://man.archlinux.org/man/systemd.special.7)
- [ArchWiki - Target](https://wiki.archlinux.org/title/Systemd/Targets)
