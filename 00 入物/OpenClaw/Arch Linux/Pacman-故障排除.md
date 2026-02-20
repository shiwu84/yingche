---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, pacman, atomic, troubleshooting]
source: https://wiki.archlinux.org/title/Pacman
aliases: [pacman 错误，常见问题]
---

# Pacman 故障排除

> [!summary] 核心概念
> 解决 Pacman 常见错误，包括文件冲突、损坏包、数据库锁等问题。

## 文件冲突错误

### 错误信息

```
error: could not prepare transaction
error: failed to commit transaction (conflicting files)
package: /path/to/file exists in filesystem
Errors occurred, no packages were upgraded.
```

### 原因

Pacman 检测到文件冲突，不会覆盖现有文件。

### 解决方案

**1. 检查文件归属**

```bash
pacman -Qo /path/to/file
```

**2. 根据结果处理**

- **文件属于其他包** → 提交 bug 报告
- **文件不属于任何包** → 重命名或删除该文件后重试

**3. 强制覆盖（谨慎使用）**

```bash
sudo pacman -S --overwrite '*' package_name
```

> ⚠️ **Warning**: 仅在确定安全的情况下使用。

## 损坏包错误

### 错误信息

```
error: could not commit transaction
error: failed to commit transaction (invalid or corrupted package)
```

### 原因

- 部分下载的包（.part 文件）
- archlinux-keyring 过期

### 解决方案

**1. 删除部分下载的包**

```bash
sudo find /var/cache/pacman/pkg/ -iname "*.part" -delete
```

**2. 更新密钥环**

```bash
sudo pacman -Sy archlinux-keyring
sudo pacman -Su
```

> 💡 **Tip**: 定期升级系统可避免此问题。

## 数据库锁错误

### 错误信息

```
error: failed to init transaction (unable to lock database)
```

### 原因

Pacman 进程未正常退出，数据库锁未释放。

### 解决方案

**删除锁文件：**

```bash
sudo rm /var/lib/pacman/db.lck
```

> ⚠️ **Warning**: 确保没有其他 pacman 进程在运行。

检查是否有 pacman 进程：
```bash
ps aux | grep pacman
```

## 包无法检索

### 错误信息

```
error: failed retrieving package
```

### 原因

- 镜像站问题
- 网络连接问题
- 包已从仓库移除

### 解决方案

**1. 更新镜像列表**

```bash
sudo pacman -Syy
```

**2. 更换镜像站**

编辑 `/etc/pacman.d/mirrorlist`，使用更快的镜像。

**3. 从 Arch Linux Archive 获取**

如果包已被移除：
https://archive.archlinux.org

## 依赖问题

### 循环依赖

```
error: unresolvable package dependencies
```

**解决方案：**
```bash
sudo pacman -Syu
```

### 缺少依赖

```
error: unmet dependency 'package>=version'
```

**解决方案：**
1. 同步数据库：`sudo pacman -Syy`
2. 完整升级：`sudo pacman -Syu`
3. 检查是否有包被 IgnorePkg 跳过

## 签名验证失败

### 错误信息

```
error: signature from "..." is unknown trust
```

### 解决方案

**1. 刷新密钥**

```bash
sudo pacman-key --refresh-keys
```

**2. 重新初始化密钥环**

```bash
sudo pacman-key --init
sudo pacman-key --populate archlinux
```

**3. 升级密钥环**

```bash
sudo pacman -Sy archlinux-keyring
sudo pacman -Su
```

## 磁盘空间不足

### 错误信息

```
error: failed to commit transaction (not enough space)
```

### 解决方案

**1. 清理缓存**

```bash
sudo paccache -r
```

**2. 查找大文件**

```bash
sudo du -ah /var | sort -rh | head -n 10
```

**3. 清理日志**

```bash
sudo journalctl --vacuum-time=1week
```

## 事务中断恢复

### 问题

Pacman 事务被中断（如断电、强制退出）。

### 解决方案

**1. 尝试完成事务**

```bash
sudo pacman -Syu
```

**2. 检查数据库完整性**

```bash
sudo pacman -Dk
```

**3. 重建本地数据库**

```bash
sudo rm -rf /var/lib/pacman/local/*
sudo pacman -Syyu
```

> ⚠️ **Warning**: 这会重建整个本地数据库，仅在必要时使用。

## 常见问题速查

| 问题 | 命令 |
|------|------|
| 文件冲突 | `pacman -Qo /path/to/file` |
| 数据库锁 | `sudo rm /var/lib/pacman/db.lck` |
| 损坏包 | `find /var/cache/pacman/pkg/ -iname "*.part" -delete` |
| 签名失败 | `sudo pacman-key --refresh-keys` |
| 空间不足 | `sudo paccache -r` |
| 依赖问题 | `sudo pacman -Syyu` |

## 相关链接

- [[Pacman-MOC]] - 返回 Pacman 知识地图
- [[Pacman-查询数据库]] - 查询包信息
- [[Pacman-清理缓存]] - 清理缓存

## 外部资源

- [ArchWiki - Pacman troubleshooting](https://wiki.archlinux.org/title/Pacman#Troubleshooting)
- [Arch 论坛](https://bbs.archlinux.org/)
