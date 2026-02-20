---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, moc]
source: https://wiki.archlinux.org/title/Systemd
---

# Systemd MOC

> [!summary] 系统和服务管理器知识地图
> Systemd 是 Linux 系统的基础构建块，提供系统和服务管理器，作为 PID 1 运行并启动系统的其余部分。

## 核心概念

- [[Systemd-基本概念]] - 什么是 systemd、units、架构概述
- [[Systemd-服务管理]] - systemctl 命令详解，管理服务和系统状态
- [[Systemd-Unit 文件]] - 编写和编辑 unit 文件，语法和结构
- [[Systemd-服务类型]] - simple、forking、oneshot、notify、dbus、idle
- [[Systemd-依赖管理]] - Requires、Wants、Before、After 依赖关系
- [[Systemd-Targets]] - 启动目标和启动顺序
- [[Systemd-定时任务]] - systemd-timers 替代 cron
- [[Systemd-日志查看]] - journalctl 日志管理
- [[Systemd-电源管理]] - reboot、suspend、hibernate 电源控制
- [[Systemd-故障排除]] - 常见问题和解决方案

## 外部资源

- [ArchWiki - Systemd](https://wiki.archlinux.org/title/Systemd)
- [systemd 官方网站](https://systemd.io/)
- [systemctl(1) man page](https://man.archlinux.org/man/systemctl.1)
- [systemd.unit(5) man page](https://man.archlinux.org/man/systemd.unit.5)
- [systemd.service(5) man page](https://man.archlinux.org/man/systemd.service.5)
