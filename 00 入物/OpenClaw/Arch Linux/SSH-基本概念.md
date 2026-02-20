---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, ssh, atomic, basics]
source: https://wiki.archlinux.org/title/OpenSSH
aliases: [SSH 协议，OpenSSH 介绍]
---

# SSH 基本概念

> [!summary] 核心概念
> SSH（Secure Shell）是用于安全远程登录的加密网络协议，OpenSSH 是其开源实现。

## 什么是 SSH

**SSH（Secure Shell）**：
- 加密的网络协议
- 用于安全的远程登录
- 在不安全的网络上安全运行网络服务
- 默认监听 TCP 端口 22

**主要应用**：
- 远程登录和命令行执行
- Git 版本控制
- rsync 文件同步
- X11 图形界面转发
- SCP 和 SFTP 文件传输

## OpenSSH vs OpenSSL

> ⚠️ **注意**：OpenSSH 和 OpenSSL 名称相似但完全不同！

| 项目 | OpenSSH | OpenSSL |
|------|---------|---------|
| **用途** | SSH 协议实现 | SSL/TLS 加密库 |
| **开发团队** | OpenBSD 项目 | OpenSSL 团队 |
| **主要程序** | ssh, sshd, scp, sftp | openssl |
| **依赖关系** | 依赖 OpenSSL | 独立库 |

## SSH 架构

### 客户端 - 服务器模型

```
SSH Client ←→ SSH Server (sshd)
     ↓              ↓
  用户终端      远程系统
```

**组件**：
- **SSH 客户端** - 发起连接的程序（ssh）
- **SSH 服务器** - 接受连接的守护进程（sshd）
- **SSH 协议** - 加密通信规则

### 工作流程

1. **TCP 连接** - 客户端连接到服务器端口 22
2. **协议协商** - 协商 SSH 版本和加密算法
3. **密钥交换** - 生成会话密钥
4. **用户认证** - 密码或公钥认证
5. **加密会话** - 建立加密通道

## SSH 软件

### OpenSSH（推荐）

**包含组件**：
- `ssh` - 客户端
- `sshd` - 服务器守护进程
- `ssh-keygen` - 密钥生成
- `ssh-agent` - 密钥代理
- `scp` - 安全复制
- `sftp` - 安全文件传输
- `ssh-add` - 添加密钥到代理

**安装**：
```bash
sudo pacman -S openssh
```

### Dropbear

**特点**：
- 轻量级 SSH 实现
- 适合嵌入式系统
- 命令：`dbclient`（客户端）

```bash
sudo pacman -S dropbear
```

### TinySSH

**特点**：
- 极简 SSH 服务器
- 仅实现 SSHv2 部分功能
- 仅依赖 glibc

```bash
sudo pacman -S tinyssh
```

### PuTTY（Windows）

**特点**：
- 终端集成 SSH/Telnet 客户端
- Windows 平台常用

```bash
sudo pacman -S putty
```

## SSH 端口

### 默认端口

- **TCP 22** - SSH 标准端口

### 修改端口

**原因**：
- 减少自动化攻击
- 避免端口冲突
- 安全策略要求

**配置**（`/etc/ssh/sshd_config`）：
```bash
Port 39901
```

> 💡 **提示**：选择未分配的端口，查看 `/etc/services`。

### 多端口监听

```bash
Port 22
Port 2222
```

SSH 可以同时监听多个端口。

## SSH 密钥类型

### 主机密钥

服务器用于标识自己的密钥对：
- `/etc/ssh/ssh_host_ed25519_key` - ED25519 算法（推荐）
- `/etc/ssh/ssh_host_ecdsa_key` - ECDSA 算法
- `/etc/ssh/ssh_host_rsa_key` - RSA 算法

### 用户密钥

用户用于认证的密钥对：
- `~/.ssh/id_ed25519` - 私钥
- `~/.ssh/id_ed25519.pub` - 公钥

## SSH 认证方式

### 1. 密码认证

```
用户输入密码 → 服务器验证 → 允许/拒绝
```

**缺点**：
- 容易受到暴力破解
- 密码可能被窃听
- 不方便自动化

### 2. 公钥认证（推荐）

```
客户端使用私钥签名 → 服务器用公钥验证 → 允许/拒绝
```

**优点**：
- 更安全
- 支持免密登录
- 适合自动化

### 3. 双因素认证

```
公钥 + PAM（如 Google Authenticator）
```

**适用场景**：
- 高安全性要求
- 合规性要求

## SSH 会话

### 加密会话

SSH 建立加密会话后：
- 所有通信都被加密
- 防止窃听
- 防止中间人攻击

### 会话复用

```bash
# 启用连接共享
Host *
  ControlMaster auto
  ControlPath ~/.ssh/sockets/%r@%h-%p
  ControlPersist 600
```

## 相关技术

### Terminal Multiplexers

常与 SSH 配合使用：
- **tmux** - 终端复用器
- **screen** - 多路复用器

**用途**：
- 保持会话
- 多窗口管理
- 断线重连

### X11 Forwarding

通过 SSH 转发图形界面：
```bash
ssh -X user@server
```

### Agent Forwarding

转发 SSH 代理：
```bash
ssh -A user@server
```

## 安全考虑

### 主要威胁

1. **暴力破解** - 尝试大量密码组合
2. **中间人攻击** - 拦截通信
3. **密钥泄露** - 私钥被盗
4. **配置错误** - 安全设置不当

### 防护措施

- 使用公钥认证
- 禁用 root 登录
- 修改默认端口
- 使用 fail2ban
- 定期更新密钥

## 版本历史

- **SSH-1** - 第一代协议（已废弃）
- **SSH-2** - 当前标准协议
- **OpenSSH** - 持续开发的开源实现

## 命令速查

| 操作 | 命令 |
|------|------|
| 连接服务器 | `ssh user@host` |
| 指定端口 | `ssh -p 2222 user@host` |
| 生成密钥 | `ssh-keygen -t ed25519` |
| 复制公钥 | `ssh-copy-id user@host` |
| 安全复制 | `scp file user@host:/path` |
| 安全 FTP | `sftp user@host` |

## 相关链接

- [[SSH-MOC]] - 返回 SSH 知识地图
- [[SSH-密钥认证]] - 公钥认证详解
- [[SSH-安全加固]] - 安全配置

## 外部资源

- [OpenSSH 官方网站](https://www.openssh.com/)
- [ArchWiki - OpenSSH](https://wiki.archlinux.org/title/OpenSSH)
- [Wikipedia - SSH](https://en.wikipedia.org/wiki/Secure_Shell)
