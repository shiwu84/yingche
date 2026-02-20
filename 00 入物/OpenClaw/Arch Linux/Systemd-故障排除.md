---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, troubleshooting]
source: https://wiki.archlinux.org/title/Systemd
aliases: [systemd 故障排除，systemd 常见问题]
---

# Systemd 故障排除

> [!summary] 核心概念
> 诊断和解决 systemd 相关问题的方法和工具。

## 诊断工具

### systemd-analyze

**查看启动时间：**
```bash
systemd-analyze
```

**查看各服务启动时间：**
```bash
systemd-analyze blame
```

**查看关键链：**
```bash
systemd-analyze critical-chain
```

**生成依赖图：**
```bash
systemd-analyze dot | dot -Tsvg > boot.svg
```

**验证 unit 文件：**
```bash
systemd-analyze verify /etc/systemd/system/my-service.service
```

### systemctl

**查看失败的服务：**
```bash
systemctl --failed
```

**查看系统状态：**
```bash
systemctl status
```

**查看特定服务：**
```bash
systemctl status service_name
```

### journalctl

**查看错误日志：**
```bash
journalctl -p err -b
```

**查看特定服务日志：**
```bash
journalctl -u service_name -b
```

**实时跟踪日志：**
```bash
journalctl -u service_name -f
```

## 常见问题

### 问题 1：服务启动失败

**症状：**
```bash
systemctl --failed
```

显示服务失败。

**诊断步骤：**

1. **查看服务状态**
```bash
systemctl status service_name
```

2. **查看详细日志**
```bash
journalctl -u service_name -b -n 50
```

3. **检查依赖**
```bash
systemctl list-dependencies service_name
```

4. **验证 unit 文件**
```bash
systemd-analyze verify /etc/systemd/system/service_name.service
```

**常见原因：**
- 配置文件错误
- 依赖服务未启动
- 权限问题
- 文件路径错误

### 问题 2：服务无法启用

**症状：**
```bash
sudo systemctl enable service_name
# 报错：Unit not found
```

**原因：**
- Unit 文件不存在
- Unit 文件路径错误
- 需要 daemon-reload

**解决：**
```bash
# 1. 检查文件是否存在
ls -l /etc/systemd/system/service_name.service

# 2. 重新加载配置
sudo systemctl daemon-reload

# 3. 再次启用
sudo systemctl enable service_name
```

### 问题 3：循环依赖

**症状：**
```
Circular dependency detected
```

**诊断：**
```bash
systemctl list-dependencies service_name
```

**解决：**
- 检查 `Requires=` 和 `Wants=`
- 检查 `Before=` 和 `After=`
- 移除不必要的依赖

### 问题 4：服务启动慢

**诊断：**
```bash
# 查看启动时间
systemd-analyze blame | grep service_name

# 查看关键链
systemd-analyze critical-chain service_name
```

**优化：**
- 检查依赖是否必要
- 使用 `Type=notify` 精确控制
- 优化服务启动脚本

### 问题 5：Timer 不触发

**症状：** Timer 到时间但未执行。

**诊断：**
```bash
# 查看 timer 状态
systemctl status timer_name

# 查看下次触发时间
systemctl list-timers

# 查看 timer 日志
journalctl -u timer_name
```

**常见原因：**
- Timer 未启用
- 时间表达式错误
- Service 有错误

**解决：**
```bash
# 启用 timer
sudo systemctl enable --now timer_name

# 检查时间表达式
# 验证格式：*-*-* HH:MM:SS
```

### 问题 6：依赖服务未就绪

**症状：** 服务启动时依赖的服务还没准备好。

**解决：**
```ini
# 添加 After= 显式指定顺序
[Unit]
After=network-online.target
Wants=network-online.target
```

### 问题 7：服务反复重启

**症状：** 服务启动后立即退出，然后 systemd 不断重启它。

**诊断：**
```bash
journalctl -u service_name -f
```

**常见原因：**
- 配置错误导致立即退出
- 缺少依赖
- 权限问题

**解决：**
1. 查看日志找出退出原因
2. 修复配置或依赖
3. 调整重启策略：
```ini
[Service]
Restart=on-failure
RestartSec=10s
```

## 调试技巧

### 1. 临时修改服务

```bash
# 创建覆盖配置
sudo systemctl edit service_name

# 添加调试选项
[Service]
ExecStart=
ExecStart=/usr/bin/my-service --debug --verbose
```

### 2. 手动运行服务命令

```bash
# 以相同用户运行
sudo -u service_user /usr/bin/my-service
```

### 3. 检查环境变量

```bash
# 查看服务的环境变量
systemctl show service_name -p Environment
```

### 4. 使用 strace 调试

```bash
# 跟踪系统调用
strace -f /usr/bin/my-service
```

### 5. 检查文件权限

```bash
# 检查可执行文件
ls -l /usr/bin/my-service

# 检查配置文件
ls -l /etc/my-service.conf
```

## 恢复模式

### 救援模式

```bash
sudo systemctl rescue
```

进入单用户模式，用于系统维护。

### 紧急模式

```bash
sudo systemctl emergency
```

进入最基础的 shell，用于系统修复。

### 从 GRUB 进入救援模式

在 GRUB 内核参数中添加：
```
systemd.unit=rescue.target
```

或紧急模式：
```
systemd.unit=emergency.target
```

## 重置和清理

### 重置失败状态

```bash
sudo systemctl reset-failed
```

清除失败服务列表。

### 清理日志

```bash
# 清理旧日志
sudo journalctl --vacuum-time=1week

# 清理特定服务日志
sudo journalctl --rotate
sudo journalctl --vacuum-files=1
```

### 重新加载所有配置

```bash
sudo systemctl daemon-reload
sudo systemctl daemon-reexec
```

## 性能优化

### 禁用不需要的服务

```bash
# 查看启用的服务
systemctl list-unit-files --state=enabled

# 禁用不需要的服务
sudo systemctl disable bluetooth.service
```

### 并行启动优化

```bash
# 查看启动瓶颈
systemd-analyze blame

# 优化依赖关系
# 移除不必要的 After= 和 Requires=
```

## 日志分析

### 查找特定错误

```bash
# 查找包含关键词的错误
journalctl -p err | grep "keyword"

# 查找特定时间段的错误
journalctl --since "1 hour ago" -p err
```

### 统计错误数量

```bash
journalctl -p err -b | wc -l
```

### 导出日志分析

```bash
# 导出为 JSON
journalctl -o json > logs.json

# 导出为文本
journalctl > logs.txt
```

## 实用脚本

### 检查所有失败服务

```bash
#!/bin/bash
echo "=== Failed Units ==="
systemctl --failed --no-legend | awk '{print $1}'

echo ""
echo "=== Recent Errors ==="
journalctl -p err -b --no-pager -n 20
```

### 服务健康检查

```bash
#!/bin/bash
SERVICE=$1

echo "=== Status ==="
systemctl is-active $SERVICE

echo ""
echo "=== Enabled ==="
systemctl is-enabled $SERVICE

echo ""
echo "=== Recent Logs ==="
journalctl -u $SERVICE -n 10 --no-pager
```

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-日志查看]] - 查看日志
- [[Systemd-服务管理]] - systemctl 命令

## 外部资源

- [systemd-analyze(1)](https://man.archlinux.org/man/systemd-analyze.1)
- [ArchWiki - Systemd troubleshooting](https://wiki.archlinux.org/title/Systemd#Troubleshooting)
- [freedesktop.org - systemd documentation](https://www.freedesktop.org/wiki/Software/systemd/)
