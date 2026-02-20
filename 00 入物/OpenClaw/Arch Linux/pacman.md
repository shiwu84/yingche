---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, package-manager, pacman, essentials]
source: https://wiki.archlinux.org/title/Pacman
aliases: [包管理器，pacman 教程，Arch 包管理]
---

# Pacman

> [!summary] 概述
> Pacman 是 Arch Linux 的包管理器，结合了简单的二进制包格式和易用的 Arch 构建系统。它是 Arch Linux 的核心特色之一，用于管理软件包的安装、更新和移除。

## 简介

Pacman 是 Arch Linux 的主要 distinguishing feature 之一，用 [C](https://wiki.archlinux.org/title/C) 语言编写，使用 [bsdtar(1)](https://man.archlinux.org/man/bsdtar.1) tar 格式进行打包。

**核心特点：**
- 保持系统更新，与主服务器同步包列表
- 服务器/客户端模型，允许通过简单命令下载/安装包
- 自动处理所有依赖关系
- 支持官方仓库和用户自建包

> 💡 **Tip**: `pacman` 包包含 `makepkg` 和 `vercmp(8)` 等工具。其他实用工具如 `pactree` 和 `checkupdates` 在 `pacman-contrib` 包中。

## 基本用法

### 安装软件包

**安装单个或多个包（包括依赖）：**
```bash
sudo pacman -S package_name1 package_name2 ...
```

**使用正则表达式安装：**
```bash
sudo pacman -S $(pacman -Ssq package_regex)
```

**从特定仓库安装：**
```bash
sudo pacman -S extra/package_name
```

**使用大括号展开安装多个相关包：**
```bash
sudo pacman -S plasma-{desktop,mediacenter,nm}
```

> ⚠️ **Warning**: 避免只刷新包列表而不升级系统。不要运行 `pacman -Sy package_name`，应该运行 `pacman -Syu package_name`，否则可能导致依赖问题。

#### 虚拟包

虚拟包是一个特殊包，本身不存在，但由一个或多个其他包提供。例如 `dbus-units`。虚拟包不能直接安装，只能安装提供它的包。

### 安装包组

**安装整个包组（如 gnome）：**
```bash
sudo pacman -S gnome
```

**查看包组包含的包：**
```bash
pacman -Sg gnome
```

**选择安装（交互式）：**
```
Enter a selection (default=all): 1-10 15
```

**排除某些包：**
```
Enter a selection (default=all): ^5-8 ^2
```

> 💡 **Tip**: 访问 https://archlinux.org/groups/ 查看可用的包组。

### 移除软件包

**移除单个包，保留依赖：**
```bash
sudo pacman -R package_name
```

**移除包及其不再需要的依赖：**
```bash
sudo pacman -Rs package_name
```

**移除包、依赖和依赖该包的其他包（递归）：**
```bash
# ⚠️ 谨慎使用，可能移除大量包
sudo pacman -Rsc package_name
```

**强制移除（不检查依赖）：**
```bash
sudo pacman -Rdd package_name
```

**移除包但不保存配置文件：**
```bash
sudo pacman -Rn package_name
```

> 💡 **Note**: Pacman 不会移除应用程序在用户主目录创建的配置文件（如 "dotfiles"）。

### 升级系统

**完整系统升级（推荐）：**
```bash
sudo pacman -Syu
```

> ⚠️ **Warning**: Arch 仅支持完整系统升级。部分升级不受支持，可能导致系统不稳定。

## 查询包数据库

### 搜索包

**在包名称和描述中搜索：**
```bash
pacman -Ss string1 string2 ...
```

**仅匹配包名（避免过多结果）：**
```bash
pacman -Ss '^vim-'
```

**搜索已安装的包：**
```bash
pacman -Qs string1 string2 ...
```

**在远程包中搜索文件名：**
```bash
pacman -F string1 string2 ...
```

### 查看包信息

**查看远程包的详细信息：**
```bash
pacman -Si package_name
```

**查看本地已安装包的详细信息：**
```bash
pacman -Qi package_name
```

**查看包安装的文件列表：**
```bash
pacman -Ql package_name
```

**验证已安装文件的完整性：**
```bash
pacman -Qk package_name
# 更彻底的检查
pacman -Qii package_name
```

**查询文件属于哪个包：**
```bash
pacman -Qo /path/to/file_name
```

**列出不再需要的依赖（孤儿包）：**
```bash
pacman -Qdt
```

**列出显式安装的包：**
```bash
pacman -Qet
```

### 依赖树

**查看包的依赖树：**
```bash
pactree package_name
```

**查看反向依赖树：**
```bash
pactree -r package_name
```

## 清理包缓存

Pacman 将下载的包存储在 `/var/cache/pacman/pkg/`，不会自动删除旧版本。

**使用 paccache 保留最近 3 个版本：**
```bash
sudo paccache -r
```

**仅保留 1 个版本：**
```bash
sudo paccache -rk1
```

**仅清理已卸载包的缓存：**
```bash
sudo paccache -ruk0
```

**使用 pacman 清理（更激进）：**
```bash
# 移除未安装包的缓存
sudo pacman -Sc

# 移除所有缓存（不推荐）
sudo pacman -Scc
```

> 💡 **Tip**: 启用 `paccache.timer` 每周自动清理：
> ```bash
> sudo systemctl enable --now paccache.timer
> ```

## 其他实用命令

**下载包但不安装：**
```bash
sudo pacman -Sw package_name
```

**安装本地包（如 AUR 下载的包）：**
```bash
sudo pacman -U /path/to/package/package_name-version.pkg.tar.zst
```

**安装远程包（URL）：**
```bash
sudo pacman -U http://www.example.com/repo/example.pkg.tar.zst
```

**预览操作（dry run）：**
```bash
pacman -Sp package_name
```

## 安装原因

Pacman 将已安装包分为两类：

- **显式安装**：直接通过 `pacman -S` 或 `pacman -U` 安装的包
- **依赖**：因其他包需要而自动安装的包

**查看显式安装的包：**
```bash
pacman -Qe
```

**查看依赖包：**
```bash
pacman -Qd
```

**将包标记为依赖：**
```bash
sudo pacman -D --asdeps package_name
```

**将包标记为显式安装：**
```bash
sudo pacman -D --asexplicit package_name
```

> 💡 **Tip**: 安装可选依赖时使用 `--asdeps`，这样在清理孤儿包时会被自动移除。

## 配置

Pacman 配置文件位于 `/etc/pacman.conf`。

### 常用配置选项

**显示版本对比信息：**
```bash
# 在 /etc/pacman.conf 中取消注释
VerbosePkgLists
```

**并行下载（默认 5）：**
```bash
# 在 /etc/pacman.conf 的 [options] 部分
ParallelDownloads = 5
```

**跳过特定包的升级：**
```bash
# 在 /etc/pacman.conf 的 [options] 部分
IgnorePkg = linux
# 多个包
IgnorePkg = linux linux-firmware
```

**跳过包组升级：**
```bash
IgnoreGroup = gnome
```

**跳过特定文件升级：**
```bash
NoUpgrade = path/to/file
NoUpgrade = path/to/file1 path/to/file2
```

**跳过文件安装：**
```bash
NoExtract = usr/share/bash-completion/completions/*
```

### 仓库和镜像

仓库配置在 `/etc/pacman.d/mirrorlist`。镜像站顺序很重要，排在前面的优先级更高。

**包缓存目录：**
```bash
# 默认：/var/cache/pacman/pkg/
# 可在 pacman.conf 中修改 CacheDir
```

> ⚠️ **Warning**: 不要将 `/var/cache/pacman/pkg/` symlink 到其他位置，会导致 pacman 行为异常。

### 包签名

默认配置 `SigLevel = Required DatabaseOptional` 启用包签名验证。详见 [[Pacman-key]]。

## 故障排除

### "conflicting files" 错误

**错误信息：**
```
error: could not prepare transaction
error: failed to commit transaction (conflicting files)
package: /path/to/file exists in filesystem
```

**解决方案：**

1. 检查文件属于哪个包：
```bash
pacman -Qo /path/to/file
```

2. 如果不属于任何包，重命名该文件后重试升级

3. 如果属于其他包，提交 bug 报告

4. 强制覆盖（谨慎使用）：
```bash
sudo pacman -S --overwrite '*' package_name
```

### "invalid or corrupted package" 错误

**原因：** 部分下载的包或过期的 archlinux-keyring

**解决方案：**

1. 删除部分下载的包：
```bash
find /var/cache/pacman/pkg/ -iname "*.part" -delete
```

2. 更新密钥环：
```bash
sudo pacman -Sy archlinux-keyring
sudo pacman -Su
```

### "unable to lock database" 错误

**原因：** pacman 进程未正常退出，数据库锁未释放

**解决方案：**
```bash
sudo rm /var/lib/pacman/db.lck
```

## 相关资源

- [Pacman/Tips and tricks](https://wiki.archlinux.org/title/Pacman/Tips_and_tricks) - 更多实用技巧
- [Pacman/Package signing](https://wiki.archlinux.org/title/Pacman/Package_signing) - 包签名
- [Pacman/Pacnew and Pacsave](https://wiki.archlinux.org/title/Pacman/Pacnew_and_Pacsave) - 配置文件处理
- [Arch User Repository](https://wiki.archlinux.org/title/Arch_User_Repository) - AUR 使用
- [System maintenance](https://wiki.archlinux.org/title/System_maintenance) - 系统维护
- [Pacman 官方文档](https://pacman.archlinux.page/)
- [pacman(8) man page](https://man.archlinux.org/man/pacman.8)

## 常用命令速查表

| 操作 | 命令 |
|------|------|
| 安装包 | `sudo pacman -S package` |
| 移除包 | `sudo pacman -R package` |
| 移除包和依赖 | `sudo pacman -Rs package` |
| 升级系统 | `sudo pacman -Syu` |
| 搜索包 | `pacman -Ss keyword` |
| 查看包信息 | `pacman -Qi package` |
| 列出文件 | `pacman -Ql package` |
| 清理缓存 | `sudo paccache -r` |
| 查看孤儿包 | `pacman -Qdt` |
| 文件属于哪个包 | `pacman -Qo /path/to/file` |

---

_本文基于 ArchWiki 内容整理，遵循 CC BY-SA 3.0 许可。最后同步：2026-02-20_
