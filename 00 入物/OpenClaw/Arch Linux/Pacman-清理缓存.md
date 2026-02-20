---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, pacman, atomic, cache]
source: https://wiki.archlinux.org/title/Pacman
aliases: [清理缓存，pacman 缓存，paccache]
---

# Pacman 清理缓存

> [!summary] 核心概念
> Pacman 存储下载的包在 `/var/cache/pacman/pkg/`，需要定期清理以防止无限增长。

## 缓存位置

Pacman 缓存目录：
```
/var/cache/pacman/pkg/
```

> 💡 **Note**: Pacman 不会自动删除旧版本或未安装包的缓存。

## 缓存的优势

保留缓存的好处：

1. **降级方便** - 无需重新下载旧版本
2. **快速重装** - 已卸载的包可直接从缓存重装
3. **离线安装** - 无网络时可重装已缓存的包

## 使用 paccache 清理（推荐）

`paccache` 是 `pacman-contrib` 包中的脚本。

### 保留最近 3 个版本

删除所有包的旧版本，只保留最近 3 个：

```bash
sudo paccache -r
```

### 保留指定数量版本

只保留 1 个版本：
```bash
sudo paccache -rk1
```

保留 2 个版本：
```bash
sudo paccache -rk2
```

### 仅清理已卸载包

删除所有已卸载包的缓存：
```bash
sudo paccache -ruk0
```

### 配置 paccache

编辑 `/etc/conf.d/pacman-contrib`：

```bash
PACCACHE_ARGS='-k1'    # 保留 1 个版本
PACCACHE_ARGS='-uk0'   # 仅清理已卸载包
```

### 启用自动清理

启用每周自动清理：
```bash
sudo systemctl enable --now paccache.timer
```

## 使用 pacman 清理

### 清理未安装包

移除未安装包的缓存和未使用的同步数据库：

```bash
sudo pacman -Sc
```

### 清理所有缓存（激进）

移除所有缓存文件：

```bash
sudo pacman -Scc
```

> ⚠️ **Warning**: 这会删除所有缓存，包括已安装包的旧版本。除非急需磁盘空间，否则不推荐。

## 其他清理工具

### pkgcacheclean

AUR 包：
```bash
yay -S pkgcacheclean
```

### pacleaner

AUR 包：
```bash
yay -S pacleaner
```

## 查看缓存大小

查看缓存目录大小：
```bash
du -sh /var/cache/pacman/pkg/
```

列出缓存中的包：
```bash
ls -lh /var/cache/pacman/pkg/
```

## 移动缓存目录

如果默认位置空间不足：

### 方法 1：修改 pacman.conf（推荐）

在 `/etc/pacman.conf` 的 `[options]` 部分：

```bash
CacheDir = /new/location/pkg/
```

> 💡 **Note**: 保留尾部斜杠。

### 方法 2：挂载专用分区

```bash
mount /dev/sdXn /var/cache/pacman/pkg/
```

或使用 Btrfs 子卷。

### 方法 3：绑定挂载

```bash
mount --bind /new/location /var/cache/pacman/pkg/
```

> ⚠️ **Warning**: 不要 symlink 缓存目录，会导致 pacman 行为异常。

## 清理策略建议

| 场景 | 推荐命令 | 频率 |
|------|---------|------|
| 日常维护 | `sudo paccache -r` | 每周 |
| 磁盘空间紧张 | `sudo paccache -rk1` | 按需 |
| 清理已卸载包 | `sudo paccache -ruk0` | 每月 |
| 紧急释放空间 | `sudo pacman -Scc` | 极少使用 |

## 相关链接

- [[Pacman-MOC]] - 返回 Pacman 知识地图
- [[Pacman-配置文件]] - pacman.conf 配置
- [[Pacman-查询数据库]] - 查询包信息
