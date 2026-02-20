---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, pacman, atomic, upgrade]
source: https://wiki.archlinux.org/title/Pacman
aliases: [系统升级，更新系统，pacman 升级]
---

# Pacman 升级系统

> [!summary] 核心概念
> Arch Linux 仅支持完整系统升级，使用 `pacman -Syu` 同步仓库数据库并升级所有包。

## 完整系统升级

同步仓库数据库并升级所有包：

```bash
sudo pacman -Syu
```

这会：
1. 同步所有启用的仓库数据库
2. 升级所有已安装的包
3. 排除不在配置仓库中的"本地"包

> ⚠️ **Warning**: Arch 仅支持完整系统升级。部分升级不受支持，可能导致系统不稳定。

## 为什么不能部分升级

> ⚠️ **Warning**: 不要运行 `pacman -Sy package_name` 来安装单个包。这会导致部分升级，可能破坏依赖关系。

**正确做法：**
```bash
# ❌ 错误：部分升级
sudo pacman -Sy firefox

# ✅ 正确：完整系统升级
sudo pacman -Syu
```

参考：[System maintenance#Partial upgrades are unsupported](https://wiki.archlinux.org/title/System_maintenance#Partial_upgrades_are_unsupported)

## 查看升级详情

启用详细包列表（在 `/etc/pacman.conf` 中取消注释）：

```bash
VerbosePkgLists
```

升级时会显示：
```
Package (6)            Old Version  New Version  Net Change  Download Size
extra/libmariadbclient 10.1.9-4     10.1.10-1    0.03 MiB    4.35 MiB
extra/libpng           1.6.19-1     1.6.20-1     0.00 MiB    0.23 MiB
```

## 跳过特定包升级

在 `/etc/pacman.conf` 的 `[options]` 部分：

```bash
IgnorePkg = linux
IgnorePkg = linux linux-firmware
```

支持 glob 模式。

> ⚠️ **Warning**: 跳过包升级需谨慎，部分升级不受支持。

临时跳过（命令行）：
```bash
sudo pacman -Syu --ignore linux
```

## 跳过包组升级

```bash
IgnoreGroup = gnome
```

## 跳过文件升级

某些文件永远不被覆盖：

```bash
NoUpgrade = path/to/file
NoUpgrade = path/to/file1 path/to/file2
```

> 💡 **Note**: 路径指包归档中的文件，不要包含前导斜杠。

## 跳过文件安装

不安装特定文件或目录：

```bash
NoExtract = usr/share/bash-completion/completions/*
```

## 并行下载

在 `/etc/pacman.conf` 中设置：

```bash
ParallelDownloads = 5
```

默认 5 个包同时下载。设为 0 或不设置则顺序下载。

## 升级流程

Pacman 升级包的工作流程：

1. 初始化事务（获取数据库锁）
2. 选择要添加/移除的包
3. 准备事务（检查依赖）
4. 提交事务：
   - 下载包
   - 执行 PreTransaction hooks
   - 移除要替换/冲突的包
   - 执行包的 pre_upgrade 脚本
   - 删除旧版本文件（保留配置文件）
   - 解压新版本包
   - 执行包的 post_upgrade 脚本
   - 执行 PostTransaction hooks
5. 释放事务锁

## 相关链接

- [[Pacman-MOC]] - 返回 Pacman 知识地图
- [[Pacman-安装软件包]] - 安装新包
- [[Pacman-配置文件]] - pacman.conf 配置
