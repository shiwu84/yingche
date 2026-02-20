---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, power]
source: https://wiki.archlinux.org/title/Systemd
aliases: [systemd 电源管理，关机，重启，休眠]
---

# Systemd 电源管理

> [!summary] 核心概念
> systemd 提供系统电源管理功能，包括关机、重启、睡眠、休眠等。

## 基本电源命令

### 关机并重启

```bash
systemctl reboot
```

### 关机并断电

```bash
systemctl poweroff
```

### 关机并挂起

```bash
systemctl halt
```

### 关闭操作系统（不断电）

```bash
systemctl shutdown
```

## 睡眠和休眠

### 挂起到内存（睡眠）

```bash
systemctl suspend
```

**特点：**
- 保持内存供电
- 唤醒快速
- 断电会丢失数据

### 休眠到磁盘

```bash
systemctl hibernate
```

**特点：**
- 内存内容写入磁盘
- 完全断电
- 唤醒较慢
- 需要 swap 空间 ≥ 内存大小

### 混合睡眠

```bash
systemctl hybrid-sleep
```

**特点：**
- 先挂起到内存
- 同时写入磁盘
- 结合了两者优点

### 先睡眠后休眠

```bash
systemctl suspend-then-hibernate
```

**特点：**
- 先睡眠指定时间
- 然后自动休眠
- 节省电量

## 软重启

### 用户空间重启

```bash
systemctl soft-reboot
```

**特点：**
- 仅重启用户空间
- 不经过内核初始化
- 跳过固件重新初始化
- 保持加密设备解锁状态

**适用场景：**
- 快速重启 systemd 和相关服务
- 切换 root 文件系统
- 容器环境

> ⚠️ **Warning**: 包更新涉及内核和 initramfs 后，不要使用软重启。

## 权限控制

### 本地用户会话

如果满足以下条件，无需 root 权限：
- 在本地 systemd-logind 用户会话中
- 没有其他活动会话

```bash
# 普通用户可以执行
systemctl reboot
systemctl poweroff
```

### 多用户情况

如果有其他用户活跃：
- 需要 root 密码
- 或使用 polkit 授权

### 远程会话

通过 SSH 等远程会话：
- 需要 root 权限
- 或使用 polkit 配置

## 定时关机

### 定时关机

```bash
# 10 分钟后关机
sudo shutdown +10

# 指定时间关机
sudo shutdown 23:00

# 取消关机
sudo shutdown -c
```

### 关机消息

```bash
sudo shutdown +5 "System will shutdown for maintenance"
```

## 配置电源管理

### logind.conf 配置

编辑 `/etc/systemd/logind.conf`：

```ini
[Login]
# 电源键行为
HandlePowerKey=poweroff

# 睡眠键行为
HandleSleepKey=suspend

# 合盖行为（笔记本）
HandleLidSwitch=suspend

# 合盖行为（外接显示器时）
HandleLidSwitchExternalPower=ignore

# 合盖行为（有用户登录时）
HandleLidSwitchDocked=ignore

# 空闲行为
IdleAction=ignore

# 空闲时间
IdleActionSec=30min
```

### 应用配置

```bash
sudo systemctl restart systemd-logind
```

## 睡眠配置

### sleep.conf 配置

编辑 `/etc/systemd/sleep.conf`：

```ini
[Sleep]
# 允许的睡眠模式
AllowSuspend=yes
AllowHibernation=yes
AllowHybridSleep=yes
AllowSuspendThenHibernate=yes

# 睡眠模式
SleepMode=suspend

# 休眠模式
HibernateMode=platform shutdown

# 休眠内存加密
HibernateState=mem
```

## 检查睡眠支持

### 检查支持的睡眠模式

```bash
cat /sys/power/state
```

**输出示例：**
```
freeze mem disk
```

- `freeze` - 浅睡眠
- `mem` - 挂起到内存（suspend）
- `disk` - 休眠到磁盘（hibernate）

### 检查休眠支持

```bash
# 检查 swap 大小
free -h

# 检查休眠镜像大小
cat /sys/power/image_size
```

### 测试睡眠

```bash
# 测试挂起
systemctl suspend

# 测试休眠
systemctl hibernate
```

## 常见问题

### 问题 1：无法休眠

**原因：** swap 空间不足

**解决：**
```bash
# 检查 swap 大小
free -h

# 增加 swap 或使用 swap 文件
```

### 问题 2：睡眠后立即唤醒

**原因：** 设备唤醒（如鼠标、网络）

**解决：**
```bash
# 查看唤醒源
cat /proc/acpi/wakeup

# 禁用特定唤醒源
echo LID > /proc/acpi/wakeup
```

### 问题 3：笔记本合盖不睡眠

**原因：** logind 配置问题

**解决：**
```ini
# /etc/systemd/logind.conf
HandleLidSwitch=suspend
```

## 电源事件

### 查看电源事件日志

```bash
journalctl -b | grep -E "sleep|hibernate|shutdown"
```

### 电源事件钩子

创建脚本在 `/lib/systemd/system-sleep/`：

```bash
#!/bin/bash

case $1/$2 in
  pre/*)
    echo "Going to $2..."
    ;;
  post/*)
    echo "Woke up from $2..."
    ;;
esac
```

## 电池管理

### 查看电池状态

```bash
upower -i /org/freedesktop/UPower/devices/battery_BAT0
```

### 查看电源信息

```bash
upower -d
```

## 最佳实践

### 1. 笔记本配置

```ini
# /etc/systemd/logind.conf
HandleLidSwitch=suspend
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

### 2. 服务器配置

```ini
# /etc/systemd/logind.conf
HandlePowerKey=ignore
HandleSleepKey=ignore
```

### 3. 桌面配置

```ini
# /etc/systemd/logind.conf
HandlePowerKey=interactive
HandleSleepKey=suspend
```

## 命令速查

| 操作 | 命令 |
|------|------|
| 重启 | `systemctl reboot` |
| 关机 | `systemctl poweroff` |
| 睡眠 | `systemctl suspend` |
| 休眠 | `systemctl hibernate` |
| 混合睡眠 | `systemctl hybrid-sleep` |
| 软重启 | `systemctl soft-reboot` |
| 定时关机 | `shutdown +10` |
| 取消关机 | `shutdown -c` |

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-服务管理]] - systemctl 命令
- [[Systemd-日志查看]] - 查看电源事件日志

## 外部资源

- [systemctl(1)](https://man.archlinux.org/man/systemctl.1)
- [systemd-logind.service(8)](https://man.archlinux.org/man/systemd-logind.service.8)
- [ArchWiki - Power management](https://wiki.archlinux.org/title/Power_management)
