---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, timers]
source: https://wiki.archlinux.org/title/Systemd
aliases: [systemd 定时任务，systemd timer, cron 替代]
---

# Systemd 定时任务

> [!summary] 核心概念
> systemd-timers 是 cron 的现代替代品，用于定时执行任务。

## 什么是 systemd-timers

systemd-timers 是 systemd 的 timer unit，用于：
- 定时激活其他 units
- 替代传统的 cron
- 提供精确的时间控制
- 记录执行历史

## Timer Unit 结构

### 基本结构

```ini
[Unit]
Description=My Daily Task

[Timer]
OnCalendar=daily
Unit=my-task.service

[Install]
WantedBy=timers.target
```

### 关联的 Service

```ini
[Unit]
Description=My Daily Task

[Service]
Type=oneshot
ExecStart=/usr/local/bin/my-daily-task.sh
```

## 时间配置

### OnCalendar 日历时间

**语法：**
```
OnCalendar=YEAR-MONTH-DAY HOUR:MINUTE:SECOND
```

**常用格式：**

| 格式 | 说明 | 示例 |
|------|------|------|
| `*-*-* HH:MM:SS` | 每天指定时间 | `*-*-* 09:00:00` |
| `*-*-* HH:MM` | 每天指定时间（无秒） | `*-*-* 09:00` |
| `DOW` | 每周某天 | `Mon`, `Fri` |
| `daily` | 每天午夜 | `daily` |
| `weekly` | 每周一午夜 | `weekly` |
| `monthly` | 每月 1 日午夜 | `monthly` |
| `yearly` | 每年 1 月 1 日 | `yearly` |

### 常用日历表达式

```ini
# 每天早上 9 点
OnCalendar=*-*-* 09:00:00

# 每周一 9 点
OnCalendar=Mon *-*-* 09:00:00

# 工作日 9 点（周一到周五）
OnCalendar=Mon..Fri *-*-* 09:00:00

# 每月 1 号和 15 号
OnCalendar=*-*-01,15 00:00:00

# 每小时
OnCalendar=hourly

# 每 15 分钟
OnCalendar=*:0/15
```

### 相对时间

```ini
# 启动后 5 分钟
OnBootSec=5min

# 空闲 10 分钟后
OnUnitActiveSec=10min

# 上次执行后 1 小时
OnUnitInactiveSec=1h
```

## 完整示例

### 示例 1：每日备份

**Timer Unit：**
```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

**Service Unit：**
```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/daily-backup.sh
```

### 示例 2：每小时检查

```ini
# /etc/systemd/system/check.timer
[Unit]
Description=Hourly Check Timer

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/check.service
[Unit]
Description=Hourly Check

[Service]
Type=oneshot
ExecStart=/usr/local/bin/hourly-check.sh
```

### 示例 3：工作日任务

```ini
# /etc/systemd/system/workday.timer
[Unit]
Description=Workday Timer

[Timer]
OnCalendar=Mon..Fri *-*-* 09:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

## 管理 Timers

### 列出所有 Timers

```bash
systemctl list-timers
```

显示：
- NEXT - 下次触发时间
- LEFT - 距离下次触发的时间
- LAST - 上次触发时间
- PASSED - 距离上次触发的时间
- UNIT - timer unit 名称
- ACTIVATES - 激活的 service
- ENABLED - 是否启用

### 列出所有 Timers（包括未激活的）

```bash
systemctl list-timers --all
```

### 启动 Timer

```bash
sudo systemctl start backup.timer
```

### 停止 Timer

```bash
sudo systemctl stop backup.timer
```

### 启用 Timer（开机启动）

```bash
sudo systemctl enable backup.timer
```

### 禁用 Timer

```bash
sudo systemctl disable backup.timer
```

### 查看 Timer 状态

```bash
systemctl status backup.timer
```

### 查看下次触发时间

```bash
systemctl show backup.timer -p NextElapseUSecRealtime
```

## Persistent 选项

### 什么是 Persistent

```ini
[Timer]
Persistent=true
```

**作用：**
- 如果系统关机时错过了定时任务
- 下次启动时会执行错过的任务

**适用场景：**
- 每日备份
- 日志轮转
- 数据同步

**不适用场景：**
- 每小时检查（可能会在启动时执行多次）

## 与其他 Timers 的区别

### OnBootSec vs OnCalendar

```ini
# 启动后 5 分钟执行一次
OnBootSec=5min

# 每天早上 9 点执行
OnCalendar=*-*-* 09:00:00
```

### OnUnitActiveSec vs OnUnitInactiveSec

```ini
# 上次启动后 1 小时
OnUnitActiveSec=1h

# 上次执行完成后 1 小时
OnUnitInactiveSec=1h
```

## 随机延迟

### RandomizedDelaySec

```ini
[Timer]
OnCalendar=daily
RandomizedDelaySec=1h
```

**作用：** 在指定时间后随机延迟执行，避免所有任务同时运行。

**适用场景：**
- 多台服务器同时更新
- 避免网络拥塞

## 精度控制

### AccuracySec

```ini
[Timer]
OnCalendar=daily
AccuracySec=1h
```

**作用：** 允许 systemd 将多个 timer 合并执行，节省电量。

**默认值：** 1 分钟

**设置为 1us：** 精确执行

## 调试 Timers

### 查看 Timer 触发历史

```bash
journalctl -u backup.timer
```

### 查看 Service 执行日志

```bash
journalctl -u backup.service
```

### 手动触发 Timer

```bash
sudo systemctl start backup.timer
```

### 测试 Service

```bash
sudo systemctl start backup.service
```

## Timers vs Cron

### Timers 优势

| 特性 | systemd-timers | cron |
|------|---------------|------|
| 日志记录 | ✅ journalctl | ❌ 需要配置 |
| 依赖管理 | ✅ systemd 依赖 | ❌ 无 |
| 错过执行 | ✅ Persistent | ❌ 需要脚本 |
| 时间精度 | ✅ 微秒级 | ⚠️ 分钟级 |
| 状态查看 | ✅ systemctl | ❌ 需要日志 |

### Cron 优势

| 特性 | systemd-timers | cron |
|------|---------------|------|
| 语法简洁 | ❌ 较复杂 | ✅ 简单 |
| 用户级别 | ⚠️ 需要配置 | ✅ 原生支持 |
| 普及度 | ⚠️ 较新 | ✅ 广泛使用 |

## 迁移 Cron 任务

### Cron 格式

```cron
# m h dom mon dow command
0 2 * * * /usr/local/bin/daily-backup.sh
```

### 转换为 systemd-timer

**Timer：**
```ini
[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
```

**Service：**
```ini
[Service]
Type=oneshot
ExecStart=/usr/local/bin/daily-backup.sh
```

## 最佳实践

### 1. 使用 Persistent

```ini
# ✅ 推荐：确保任务执行
Persistent=true

# ❌ 避免：可能错过任务
Persistent=false
```

### 2. 合理的随机延迟

```ini
# ✅ 推荐：避免同时执行
RandomizedDelaySec=5min

# ❌ 避免：延迟过长
RandomizedDelaySec=24h
```

### 3. 清晰的命名

```ini
# ✅ 推荐：清晰命名
backup.timer → backup.service

# ❌ 避免：混淆命名
daily-task.timer → nightly-job.service
```

## 常见问题

### Timer 不触发

**检查：**
1. Timer 是否启用：`systemctl list-timers`
2. 时间表达式是否正确
3. Service 是否有错误

### Service 执行失败

**检查：**
```bash
journalctl -u service_name -n 50
```

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-日志查看]] - 查看执行日志
- [[Systemd-服务类型]] - oneshot 类型

## 外部资源

- [systemd.timer(5)](https://man.archlinux.org/man/systemd.timer.5)
- [ArchWiki - systemd/timers](https://wiki.archlinux.org/title/Systemd/Timers)
