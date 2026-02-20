---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, pacman, atomic, config]
source: https://wiki.archlinux.org/title/Pacman
aliases: [pacman.conf, pacman 配置]
---

# Pacman 配置文件

> [!summary] 核心概念
> Pacman 配置文件 `/etc/pacman.conf` 控制包管理器行为，包括仓库、镜像、下载选项等。

## 配置文件位置

主配置文件：
```
/etc/pacman.conf
```

镜像列表：
```
/etc/pacman.d/mirrorlist
```

Hooks 目录：
```
/etc/pacman.d/hooks/
```

## 一般选项

在 `[options]` 部分配置：

### VerbosePkgLists

显示版本对比信息：

```bash
VerbosePkgLists
```

升级时显示：
```
Package (6)            Old Version  New Version  Net Change  Download Size
extra/libmariadbclient 10.1.9-4     10.1.10-1    0.03 MiB    4.35 MiB
```

### ParallelDownloads

并行下载包数量：

```bash
ParallelDownloads = 5
```

默认 5，设为 0 或不设置则顺序下载。

### CacheDir

包缓存目录：

```bash
CacheDir = /var/cache/pacman/pkg/
```

> 💡 **Note**: 保留尾部斜杠。

### SigLevel

包签名验证级别：

```bash
SigLevel = Required DatabaseOptional
```

全局启用包签名验证。

## 仓库配置

每个 `[repository]` 部分定义一个仓库：

```bash
[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

[community]
Include = /etc/pacman.d/mirrorlist
```

> 💡 **Note**: 仓库顺序很重要，排在前面的优先级更高。

### 启用测试仓库

```bash
[core-testing]
Include = /etc/pacman.d/mirrorlist
```

## 跳过升级

### IgnorePkg

跳过特定包升级：

```bash
IgnorePkg = linux
IgnorePkg = linux linux-firmware
```

支持 glob 模式。

> ⚠️ **Warning**: 跳过包需谨慎，部分升级不受支持。

### IgnoreGroup

跳过包组升级：

```bash
IgnoreGroup = gnome
```

### NoUpgrade

某些文件永远不被覆盖：

```bash
NoUpgrade = path/to/file
NoUpgrade = path/to/file1 path/to/file2
```

> 💡 **Note**: 路径指包归档中的文件，不要包含前导斜杠。

### NoExtract

跳过文件安装：

```bash
NoExtract = usr/share/bash-completion/completions/*
```

后续规则可覆盖前面的，用 `!` 否定：
```bash
NoExtract = usr/share/bash-completion/*
NoExtract = !usr/share/bash-completion/completions/vim
```

## Include 选项

共享配置：

```bash
Include = /path/to/common/settings
```

多个配置文件可包含相同的设置文件。

## Hooks

Pacman 在事务前后执行 hooks：

```bash
HookDir = /etc/pacman.d/hooks
```

默认目录：`/usr/share/libalpm/hooks/`

Hook 文件必须以 `.hook` 结尾。

### 示例 Hooks

**systemd-sysusers.hook** - 自动创建系统用户

**systemd-tmpfiles.hook** - 自动创建临时文件

参考：[alpm-hooks(5)](https://man.archlinux.org/man/alpm-hooks.5)

## 常用配置示例

```bash
[options]
# 显示版本对比
VerbosePkgLists

# 并行下载 5 个包
ParallelDownloads = 5

# 缓存目录
CacheDir = /var/cache/pacman/pkg/

# 包签名验证
SigLevel = Required DatabaseOptional

# 跳过内核升级（谨慎使用）
# IgnorePkg = linux

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

[community]
Include = /etc/pacman.d/mirrorlist

# [community-testing]
# Include = /etc/pacman.d/mirrorlist
```

## 镜像配置

编辑 `/etc/pacman.d/mirrorlist`：

```bash
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch
```

> 💡 **Tip**: 将最快的镜像放在最前面。

## 维护多个配置文件

主配置：
```bash
# /etc/pacman.conf
Include = /etc/pacman.d/settings
```

测试配置：
```bash
# /etc/pacman-testing.conf
Include = /etc/pacman.d/settings
[core-testing]
Include = /etc/pacman.d/mirrorlist
```

共享设置：
```bash
# /etc/pacman.d/settings
ParallelDownloads = 5
VerbosePkgLists
```

## 相关链接

- [[Pacman-MOC]] - 返回 Pacman 知识地图
- [[Pacman-升级系统]] - 系统升级
- [[Pacman-清理缓存]] - 缓存管理
