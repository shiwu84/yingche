---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, journal, logging]
source: https://wiki.archlinux.org/title/Systemd
aliases: [journalctl, systemd 日志，系统日志]
---

# Systemd 日志查看

> [!summary] 核心概念
> journald 是 systemd 的日志守护进程，journalctl 用于查看和管理日志。

## 什么是 journald

journald 是 systemd 的日志组件：
- 收集系统和服务日志
- 结构化存储（JSON 格式）
- 支持多种输出格式
- 自动日志轮转

## journalctl 基本用法

### 查看所有日志

```bash
journalctl
```

### 查看实时日志

```bash
journalctl -f
```

类似 `tail -f`，实时跟踪日志。

### 查看最近 N 行

```bash
journalctl -n 100
```

### 查看特定服务的日志

```bash
journalctl -u service_name
```

**示例：**
```bash
journalctl -u sshd.service
journalctl -u nginx.service
```

### 查看多个服务的日志

```bash
journalctl -u service1 -u service2
```

## 时间过滤

### 从今天开始

```bash
journalctl --since today
```

### 从指定时间开始

```bash
journalctl --since "2024-01-01 09:00:00"
```

### 时间范围

```bash
journalctl --since "2024-01-01" --until "2024-01-02"
```

### 相对时间

```bash
# 最近 1 小时
journalctl --since "1 hour ago"

# 最近 30 分钟
journalctl --since "30 min ago"

# 最近 2 天
journalctl --since "2 days ago"
```

### 快速时间别名

| 别名 | 说明 |
|------|------|
| `today` | 今天 00:00 |
| `yesterday` | 昨天 00:00 |
| `this-week` | 本周开始 |
| `this-month` | 本月开始 |
| `this-year` | 今年开始 |

## 优先级过滤

### 查看特定优先级

```bash
# 查看错误及以上
journalctl -p err

# 查看警告及以上
journalctl -p warning

# 查看通知及以上
journalctl -p notice
```

### 优先级级别

| 级别 | 值 | 说明 |
|------|-----|------|
| emerg | 0 | 系统不可用 |
| alert | 1 | 需要立即行动 |
| crit | 2 | 严重错误 |
| err | 3 | 错误 |
| warning | 4 | 警告 |
| notice | 5 | 正常但重要 |
| info | 6 | 信息 |
| debug | 7 | 调试 |

### 查看错误日志

```bash
journalctl -p err -b
```

查看本次启动以来的错误。

## 启动相关

### 查看本次启动的日志

```bash
journalctl -b
```

### 查看上次启动的日志

```bash
journalctl -b -1
```

### 查看特定启动的日志

```bash
# 列出所有启动
journalctl --list-boots

# 查看第 2 次启动的日志
journalctl -b -2
```

### 查看启动时间

```bash
journalctl --list-boots
```

显示：
```
-2 abc123... Mon 2024-01-01 09:00:00—Mon 2024-01-01 18:00:00
-1 def456... Tue 2024-01-02 08:30:00—Tue 2024-01-02 20:00:00
 0 789abc... Wed 2024-01-03 09:15:00—n/a
```

## 输出格式

### 简短格式（默认）

```bash
journalctl -o short
```

### 详细格式

```bash
journalctl -o verbose
```

显示所有字段。

### JSON 格式

```bash
journalctl -o json
```

适用于程序处理。

### JSON 流格式

```bash
journalctl -o json-pretty
```

格式化的 JSON。

### 导出格式

```bash
# 导出为二进制（可导入）
journalctl -o export > backup.journal

# 导入
journalctl --file=backup.journal
```

## 字段过滤

### 查看可用字段

```bash
journalctl -F _SYSTEMD_UNIT
```

### 按字段过滤

```bash
# 按用户 ID
journalctl _UID=1000

# 按进程 ID
journalctl _PID=1234

# 按可执行文件
journalctl _EXE=/usr/bin/nginx

# 按主机名
journalctl _HOSTNAME=myserver
```

### 组合过滤

```bash
journalctl _SYSTEMD_UNIT=nginx.service _UID=0
```

## 常用字段

| 字段 | 说明 |
|------|------|
| `_SYSTEMD_UNIT` | Unit 名称 |
| `_PID` | 进程 ID |
| `_UID` | 用户 ID |
| `_EXE` | 可执行文件路径 |
| `_CMDLINE` | 命令行 |
| `_HOSTNAME` | 主机名 |
| `MESSAGE` | 日志消息 |
| `PRIORITY` | 优先级 |
| `SYSLOG_FACILITY` | Syslog 设备 |

## 日志管理

### 查看日志大小

```bash
journalctl --disk-usage
```

### 清理日志

```bash
# 保留最近 1 秒的日志
sudo journalctl --vacuum-time=1s

# 保留最近 100MB
sudo journalctl --vacuum-size=100M

# 保留最近 1000 行
sudo journalctl --vacuum-files=1000
```

### 配置日志大小限制

编辑 `/etc/systemd/journald.conf`：

```ini
[Journal]
# 最大占用空间
SystemMaxUse=1G

# 保留的最小空间
SystemKeepFree=5G

# 单个日志文件最大大小
SystemMaxFileSize=100M

# 保留的日志时间
MaxRetentionSec=1month
```

应用配置：
```bash
sudo systemctl restart systemd-journald
```

## 永久存储

### 启用永久日志

默认 journald 使用易失性存储（/run/log/journal）。

**创建持久存储目录：**
```bash
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo systemctl restart systemd-journald
```

### 验证

```bash
journalctl --list-boots
```

如果有历史记录，说明永久存储已启用。

## 实用技巧

### 1. 查看内核消息

```bash
journalctl -k
```

### 2. 查看用户会话日志

```bash
journalctl --user
```

### 3. 查看特定用户的所有日志

```bash
journalctl _UID=1000
```

### 4. 搜索关键词

```bash
journalctl | grep "error"
```

### 5. 查看日志统计

```bash
journalctl --header
```

### 6. 无颜色输出

```bash
journalctl --no-pager -n 100
```

适用于脚本和重定向。

### 7. 显示行数

```bash
journalctl --lines=50
```

## 常见问题

### 问题 1：日志丢失

**原因：** 使用了易失性存储

**解决：** 启用永久存储

### 问题 2：日志占用过大

**解决：**
```bash
# 清理旧日志
sudo journalctl --vacuum-time=1week

# 限制大小
sudo journalctl --vacuum-size=500M
```

### 问题 3：找不到旧日志

**检查：**
```bash
# 查看可用启动记录
journalctl --list-boots

# 检查日志目录
ls -lh /var/log/journal/
```

## 配置选项

### journald.conf

```ini
[Journal]
# 存储方式：auto, persistent, volatile, none
Storage=auto

# 压缩方式：xz, gzip, lz4, zstd
Compress=yes

# 最大占用空间
SystemMaxUse=3G

# 保留的最小空间
SystemKeepFree=5G

# 单个文件最大大小
SystemMaxFileSize=500M

# 日志保留时间
MaxRetentionSec=1month

# 转发到 syslog
ForwardToSyslog=no

# 转发到控制台
ForwardToConsole=no

# 转发到墙壁
ForwardToWall=no
```

## 最佳实践

### 1. 启用永久存储

```bash
# ✅ 推荐：永久存储
sudo mkdir -p /var/log/journal

# ❌ 避免：易失性存储（重启丢失）
# /run/log/journal
```

### 2. 定期清理

```bash
# 创建定时任务
sudo systemctl edit systemd-journald-clean.timer
```

### 3. 合理设置大小限制

```ini
# ✅ 推荐：根据磁盘大小设置
SystemMaxUse=1G

# ❌ 避免：不限制
# SystemMaxUse=
```

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-定时任务]] - 查看 timer 日志
- [[Systemd-故障排除]] - 查看错误日志

## 外部资源

- [journalctl(1)](https://man.archlinux.org/man/journalctl.1)
- [systemd-journald.service(8)](https://man.archlinux.org/man/systemd-journald.service.8)
- [ArchWiki - Journal](https://wiki.archlinux.org/title/Systemd/Journal)
