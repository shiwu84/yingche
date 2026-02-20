---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, ssh, atomic, installation, config]
source: https://wiki.archlinux.org/title/OpenSSH
aliases: [SSH 安装，sshd 配置，ssh 配置]
---

# SSH 安装配置

> [!summary] 核心概念
> 安装 openssh 包，配置服务端和客户端，管理 sshd 服务。

## 安装 OpenSSH

### 安装

```bash
sudo pacman -S openssh
```

### 包含的组件

| 程序 | 用途 |
|------|------|
| `ssh` | 客户端 |
| `sshd` | 服务器守护进程 |
| `ssh-keygen` | 密钥生成工具 |
| `ssh-agent` | 密钥代理 |
| `ssh-add` | 添加密钥到代理 |
| `scp` | 安全文件复制 |
| `sftp` | 安全文件传输 |
| `ssh-copy-id` | 复制公钥到服务器 |

## 服务端配置

### 配置文件位置

- **主配置**：`/etc/ssh/sshd_config`
- **扩展配置**：`/etc/ssh/sshd_config.d/*.conf`（推荐）
- **主机密钥**：`/etc/ssh/ssh_host_*_key`

### 启动服务

```bash
# 启用并启动 sshd 服务
sudo systemctl enable sshd
sudo systemctl start sshd

# 查看服务状态
systemctl status sshd
```

### 测试配置

修改配置后，先测试再重启：

```bash
# 测试配置（无输出表示正确）
sudo sshd -t

# 重启服务
sudo systemctl restart sshd
```

### 基本配置选项

**`/etc/ssh/sshd_config.d/20-custom.conf`**：

```bash
# 监听端口
Port 22

# 允许的用户
AllowUsers user1 user2

# 允许的用户组
AllowGroups sshusers

# 欢迎信息
Banner /etc/issue

# 使用的主机密钥
HostKey /etc/ssh/ssh_host_ed25519_key
```

### 修改默认端口

> 💡 **提示**：修改端口可减少自动化攻击。

```bash
# 选择未分配的端口（如 39901）
Port 39901
```

**查看已分配端口**：
```bash
cat /etc/services | grep -E "^[a-z].*\d"
```

### 多端口监听

```bash
Port 22
Port 2222
```

SSH 可以同时监听多个端口。

## 客户端配置

### 配置文件位置

- **用户配置**：`~/.ssh/config`
- **全局配置**：`/etc/ssh/ssh_config`
- **扩展配置**：`/etc/ssh/ssh_config.d/*.conf`

### 基本配置示例

**`~/.ssh/config`**：

```bash
# 全局选项
User myuser
IdentityFile ~/.ssh/id_ed25519

# 主机特定配置
Host myserver
    Hostname server.example.com
    Port 2222
    User admin
    IdentityFile ~/.ssh/work_key

Host github.com
    User git
    IdentityFile ~/.ssh/github_key

# 跳板机配置
Host jump
    Hostname jump.example.com
    ProxyJump none

Host internal
    Hostname internal.server
    ProxyJump jump
```

### 常用配置选项

| 选项 | 说明 | 示例 |
|------|------|------|
| `Host` | 主机别名 | `Host myserver` |
| `Hostname` | 实际主机名 | `Hostname 192.168.1.100` |
| `Port` | 端口 | `Port 2222` |
| `User` | 用户名 | `User admin` |
| `IdentityFile` | 私钥文件 | `IdentityFile ~/.ssh/key` |
| `ProxyJump` | 跳板机 | `ProxyJump jump` |
| `ForwardAgent` | 代理转发 | `ForwardAgent yes` |

### 命令行选项

```bash
# 指定端口
ssh -p 2222 user@host

# 指定密钥
ssh -i ~/.ssh/custom_key user@host

# 指定配置选项
ssh -o "StrictHostKeyChecking=no" user@host

# 组合使用
ssh -p 2222 -i ~/.ssh/key -o "User=admin" host
```

## 主机密钥管理

### 自动生成

首次启动 sshd 时自动生成：
```bash
sudo systemctl start sshd
```

密钥位置：
- `/etc/ssh/ssh_host_ed25519_key`
- `/etc/ssh/ssh_host_ecdsa_key`
- `/etc/ssh/ssh_host_rsa_key`

### 手动生成

```bash
# 生成所有类型的密钥
sudo ssh-keygen -A

# 生成特定类型的密钥
sudo ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ""
```

### 指定使用的密钥

**`/etc/ssh/sshd_config.d/10-hostkey.conf`**：

```bash
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key
```

## Socket 激活（不推荐）

> ⚠️ **警告**：OpenSSH 8.0p1-3 移除了 sshd.socket，因为容易受到 DoS 攻击。

### 问题

- 可能导致拒绝服务
- 无法设置 `Restart=always`
- 忽略 `ListenAddress` 设置

### 迁移到 sshd.service

```bash
# 禁用 socket
sudo systemctl disable sshd.socket

# 启用服务
sudo systemctl enable sshd.service
sudo systemctl start sshd.service
```

## 配置最佳实践

### 1. 使用扩展配置

```bash
# ✅ 推荐：使用 /etc/ssh/sshd_config.d/
sudo nano /etc/ssh/sshd_config.d/20-custom.conf

# ❌ 避免：直接修改主配置文件
# /etc/ssh/sshd_config
```

### 2. 限制访问用户

```bash
# 只允许特定用户
AllowUsers admin deploy

# 只允许特定组
AllowGroups sshusers
```

### 3. 添加欢迎信息

```bash
# 创建欢迎信息
sudo nano /etc/issue

# 配置
Banner /etc/issue
```

### 4. 备份配置

```bash
# 备份配置
sudo cp -r /etc/ssh ~/backup/ssh-$(date +%Y%m%d)
```

## 调试技巧

### 测试配置

```bash
# 测试 sshd 配置
sudo sshd -t

# 详细测试
sudo sshd -t -f /etc/ssh/sshd_config
```

### 查看日志

```bash
# 实时查看日志
sudo journalctl -u sshd -f

# 查看最近的日志
sudo journalctl -u sshd -n 50
```

### 调试模式

```bash
# 前台运行，详细输出
sudo systemctl stop sshd
sudo /usr/bin/sshd -d -e
```

## 常见问题

### 问题 1：服务无法启动

**检查**：
```bash
# 测试配置
sudo sshd -t

# 查看错误
sudo journalctl -u sshd -n 50
```

**解决**：修复配置错误。

### 问题 2：端口被占用

**检查**：
```bash
sudo ss -tlnp | grep :22
```

**解决**：修改端口或停止占用服务。

### 问题 3：配置不生效

**原因**：配置文件语法错误或权限问题。

**解决**：
```bash
# 检查权限
ls -l /etc/ssh/sshd_config

# 正确权限
sudo chmod 644 /etc/ssh/sshd_config
sudo chown root:root /etc/ssh/sshd_config
```

## 命令速查

| 操作 | 命令 |
|------|------|
| 安装 | `sudo pacman -S openssh` |
| 启动服务 | `sudo systemctl start sshd` |
| 启用服务 | `sudo systemctl enable sshd` |
| 测试配置 | `sudo sshd -t` |
| 重启服务 | `sudo systemctl restart sshd` |
| 查看状态 | `systemctl status sshd` |
| 查看日志 | `journalctl -u sshd -f` |

## 相关链接

- [[SSH-MOC]] - 返回 SSH 知识地图
- [[SSH-密钥认证]] - 密钥认证配置
- [[SSH-安全加固]] - 安全配置选项

## 外部资源

- [sshd_config(5)](https://man.archlinux.org/man/sshd_config.5)
- [ssh_config(5)](https://man.archlinux.org/man/ssh_config.5)
- [ArchWiki - OpenSSH](https://wiki.archlinux.org/title/OpenSSH)
