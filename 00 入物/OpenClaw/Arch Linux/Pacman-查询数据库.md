---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, pacman, atomic, query]
source: https://wiki.archlinux.org/title/Pacman
aliases: [搜索包，查询包，pacman 搜索]
---

# Pacman 查询数据库

> [!summary] 核心概念
> 使用不同的标志查询本地数据库（-Q）、同步数据库（-S）和文件数据库（-F）。

## 查询标志

- `-Q` - 查询本地已安装的包
- `-S` - 查询同步数据库（远程仓库）
- `-F` - 查询文件数据库

## 搜索包

### 在远程仓库中搜索

搜索包名和描述：
```bash
pacman -Ss string1 string2 ...
```

**示例：**
```bash
pacman -Ss firefox
pacman -Ss web browser
```

仅匹配包名（避免过多结果）：
```bash
pacman -Ss '^vim-'
```

### 搜索已安装的包

```bash
pacman -Qs string1 string2 ...
```

### 搜索文件名

在远程包中搜索文件名：
```bash
pacman -F string1 string2 ...
```

先同步文件数据库：
```bash
sudo pacman -Fy
```

> 💡 **Tip**: 启用 `pacman-filesdb-refresh.timer` 每周自动刷新文件数据库。

## 查看包信息

### 远程包信息

```bash
pacman -Si package_name
```

显示依赖、大小、描述等详细信息。

### 本地包信息

```bash
pacman -Qi package_name
```

查看已安装包的详细信息。

显示备份文件状态：
```bash
pacman -Qii package_name
```

## 列出文件

### 已安装包的文件列表

```bash
pacman -Ql package_name
```

### 远程包的文件列表

```bash
pacman -Fl package_name
```

## 验证文件完整性

检查已安装文件：
```bash
pacman -Qk package_name
```

更彻底的检查：
```bash
pacman -Qkk package_name
```

## 查询文件所属包

### 本地文件属于哪个包

```bash
pacman -Qo /path/to/file_name
```

**示例：**
```bash
pacman -Qo /usr/bin/vim
```

### 远程文件属于哪个包

```bash
pacman -F /path/to/file_name
```

## 列出孤儿包

列出不再作为依赖需要的包：
```bash
pacman -Qdt
```

## 列出显式安装的包

列出显式安装（非依赖）的包：
```bash
pacman -Qet
```

## 查看依赖树

### 正向依赖树

查看包依赖哪些包：
```bash
pactree package_name
```

**示例：**
```bash
pactree firefox
```

### 反向依赖树

查看哪些包依赖该包：
```bash
pactree -r package_name
```

## 安装原因

### 查看安装原因

显式安装的包：
```bash
pacman -Qe
```

依赖安装的包：
```bash
pacman -Qd
```

### 改变安装原因

标记为依赖：
```bash
sudo pacman -D --asdeps package_name
```

标记为显式安装：
```bash
sudo pacman -D --asexplicit package_name
```

> 💡 **Tip**: 可选依赖推荐用 `--asdeps` 安装，清理孤儿包时会自动移除。

## 数据库结构

Pacman 数据库位于 `/var/lib/pacman/sync/`：

```bash
tree /var/lib/pacman/sync/which-2.21-5
which-2.21-5
├── desc  # 元数据（描述、依赖、大小、MD5）
```

每个仓库在 `/etc/pacman.conf` 中配置，对应一个数据库文件。

## 相关链接

- [[Pacman-MOC]] - 返回 Pacman 知识地图
- [[Pacman-清理缓存]] - 清理缓存
- [[Pacman-故障排除]] - 故障排除
