---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, pacman, atomic, install]
source: https://wiki.archlinux.org/title/Pacman
aliases: [安装包，pacman 安装]
---

# Pacman 安装软件包

> [!summary] 核心概念
> 使用 `pacman -S` 命令安装软件包，pacman 会自动处理所有依赖关系。

## 安装单个包

安装一个或多个包（包括依赖）：

```bash
sudo pacman -S package_name1 package_name2 ...
```

**示例：**
```bash
sudo pacman -S firefox
sudo pacman -S vim git curl
```

## 从特定仓库安装

当包在多个仓库中存在时，指定仓库名：

```bash
sudo pacman -S extra/package_name
```

## 使用正则表达式安装

```bash
sudo pacman -S $(pacman -Ssq package_regex)
```

## 使用大括号展开

安装多个相关包：

```bash
sudo pacman -S plasma-{desktop,mediacenter,nm}
```

可以嵌套展开：
```bash
sudo pacman -S plasma-{workspace{,-wallpapers},pa}
```

## 安装包组

安装整个包组（如 gnome、kde 等）：

```bash
sudo pacman -S gnome
```

系统会提示选择要安装的包：
```
Enter a selection (default=all): 1-10 15
```

选择特定范围：
```
Enter a selection (default=all): 1-10 15
```

排除某些包：
```
Enter a selection (default=all): ^5-8 ^2
```

查看包组包含的包：
```bash
pacman -Sg gnome
```

## 虚拟包

虚拟包不能直接安装，只能安装提供它的包：

```bash
# 错误：虚拟包不能直接安装
sudo pacman -S dbus-units

# 正确：安装提供虚拟包的实际包
sudo pacman -S dbus
```

## 安装本地包

安装不在仓库中的本地包（如 AUR 下载的包）：

```bash
sudo pacman -U /path/to/package/package_name-version.pkg.tar.zst
```

保留本地包副本：
```bash
sudo pacman -U file:///path/to/package/package_name-version.pkg.tar.zst
```

## 安装远程包

从 URL 安装包：

```bash
sudo pacman -U http://www.example.com/repo/example.pkg.tar.zst
```

## 安装原因

**显式安装**（默认）：
```bash
sudo pacman -S package_name
```

**作为依赖安装**（可选依赖推荐）：
```bash
sudo pacman -S --asdeps package_name
```

> 💡 **Tip**: 安装可选依赖时使用 `--asdeps`，这样在清理孤儿包时会被自动移除。

## 重要警告

> ⚠️ **Warning**: 避免只刷新包列表而不升级系统。不要运行 `pacman -Sy package_name`，应该运行 `pacman -Syu package_name`，否则可能导致依赖问题。

## 相关链接

- [[Pacman-MOC]] - 返回 Pacman 知识地图
- [[Pacman-升级系统]] - 系统升级操作
- [[Pacman-查询数据库]] - 搜索包信息
