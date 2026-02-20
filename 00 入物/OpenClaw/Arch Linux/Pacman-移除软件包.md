---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, pacman, atomic, remove]
source: https://wiki.archlinux.org/title/Pacman
aliases: [删除包，卸载包，pacman 删除]
---

# Pacman 移除软件包

> [!summary] 核心概念
> 使用 `pacman -R` 命令移除软件包，可选择同时移除不再需要的依赖。

## 基本移除

移除单个包，保留依赖：

```bash
sudo pacman -R package_name
```

## 移除包和依赖

移除包及其不再被其他包需要的依赖：

```bash
sudo pacman -Rs package_name
```

**示例：**
```bash
sudo pacman -Rs firefox
```

## 递归移除（谨慎使用）

移除包、依赖和依赖该包的其他包：

```bash
# ⚠️ Warning: 递归操作，可能移除大量包
sudo pacman -Rsc package_name
```

> ⚠️ **Warning**: 这个操作是递归的，必须谨慎使用，因为它可能移除许多可能需要的包。

## 强制移除

移除被其他包依赖的包，不移除依赖包：

```bash
sudo pacman -Rdd package_name
```

> ⚠️ **Warning**: 这会破坏依赖关系，仅在特殊情况下使用。

## 移除时不保存配置

移除包但不保存配置文件（.pacsave）：

```bash
sudo pacman -Rn package_name
```

## 移除包组

移除整个包组：

```bash
sudo pacman -R gnome
```

> ⚠️ **Warning**: 移除包组时会忽略包的安装原因，可能误删需要的包。

如果失败，尝试：
```bash
sudo pacman -Rsu package_name
```

## 配置文件处理

Pacman 在移除某些应用时会保存重要配置文件为 `.pacsave` 扩展名：

```
/etc/config.pacsave
```

> 💡 **Note**: Pacman 不会移除应用程序在用户主目录创建的配置文件（如 "dotfiles"）。

## 改变安装原因

将已安装包标记为依赖：
```bash
sudo pacman -D --asdeps package_name
```

将依赖包标记为显式安装：
```bash
sudo pacman -D --asexplicit package_name
```

> 💡 **Tip**: 这不会影响当前安装，只影响后续清理孤儿包时的行为。

## 相关链接

- [[Pacman-MOC]] - 返回 Pacman 知识地图
- [[Pacman-清理缓存]] - 清理未使用的包
- [[Pacman-配置文件]] - 配置选项
