---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, ssh, atomic, keys, authentication]
source: https://wiki.archlinux.org/title/OpenSSH
aliases: [SSH 密钥，公钥认证，免密登录]
---

# SSH 密钥认证

> [!summary] 核心概念
> 使用公私钥对进行认证，比密码更安全，支持免密登录。

## 密钥类型

### 推荐算法

| 类型 | 长度 | 安全性 | 兼容性 |
|------|------|--------|--------|
| **ED25519** | 256 位 | ⭐⭐⭐⭐⭐ | 现代系统 |
| **ECDSA** | 256/384/521 位 | ⭐⭐⭐⭐ | 广泛支持 |
| **RSA** | 3072+ 位 | ⭐⭐⭐ | 最佳兼容 |

### 选择建议

```bash
# ✅ 推荐：ED25519（现代系统）
ssh-keygen -t ed25519

# ✅ 兼容：RSA 4096 位（旧系统）
ssh-keygen -t rsa -b 4096

# ✅ 平衡：ECDSA
ssh-keygen -t ecdsa -b 521
```

## 生成密钥对

### 基本生成

```bash
# 生成 ED25519 密钥
ssh-keygen -t ed25519

# 生成 RSA 密钥
ssh-keygen -t rsa -b 4096
```

### 带注释生成

```bash
ssh-keygen -t ed25519 -C "user@example.com"
```

### 指定文件名

```bash
ssh-keygen -t ed25519 -f ~/.ssh/work_key
```

### 无密码短语（不推荐）

```bash
# ⚠️ 警告：降低安全性
ssh-keygen -t ed25519 -N ""
```

## 密钥文件

### 私钥

```
~/.ssh/id_ed25519
```

**权限要求**：
```bash
chmod 600 ~/.ssh/id_ed25519
```

### 公钥

```
~/.ssh/id_ed25519.pub
```

**可以公开分享**。

### 查看公钥

```bash
cat ~/.ssh/id_ed25519.pub
```

### 查看指纹

```bash
# 查看公钥指纹
ssh-keygen -lf ~/.ssh/id_ed25519.pub

# 查看指纹（SHA256）
ssh-keygen -lf ~/.ssh/id_ed25519.pub -E sha256
```

## 复制公钥到服务器

### 使用 ssh-copy-id（推荐）

```bash
# 基本用法
ssh-copy-id user@server

# 指定端口
ssh-copy-id -p 2222 user@server

# 指定密钥
ssh-copy-id -i ~/.ssh/custom_key user@server
```

### 手动复制

```bash
# 1. 创建目录
ssh user@server "mkdir -p ~/.ssh"

# 2. 复制公钥
cat ~/.ssh/id_ed25519.pub | ssh user@server "cat >> ~/.ssh/authorized_keys"

# 3. 设置权限
ssh user@server "chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

### 检查权限

```bash
# 服务器端权限必须正确
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R $USER:$USER ~/.ssh
```

## 测试免密登录

```bash
# 测试连接
ssh user@server

# 应该无需密码直接登录
```

## ssh-agent 密钥代理

### 启动代理

```bash
# 启动 ssh-agent
eval "$(ssh-agent -s)"
```

### 添加密钥

```bash
# 添加密钥到代理
ssh-add ~/.ssh/id_ed25519

# 添加默认密钥
ssh-add

# 查看已添加的密钥
ssh-add -l

# 删除所有密钥
ssh-add -D
```

### 自动启动（推荐）

**`~/.bashrc`** 或 **`~/.zshrc`**：

```bash
# 自动启动 ssh-agent
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)"
fi
```

### 密钥密码短语缓存

使用 `ssh-add` 添加密钥后，只需输入一次密码短语。

## 多密钥管理

### 配置不同服务器使用不同密钥

**`~/.ssh/config`**：

```bash
# 工作服务器
Host work
    Hostname work.example.com
    User admin
    IdentityFile ~/.ssh/work_key

# 个人服务器
Host personal
    Hostname home.example.com
    User user
    IdentityFile ~/.ssh/personal_key

# GitHub
Host github.com
    User git
    IdentityFile ~/.ssh/github_key
```

### 指定密钥连接

```bash
ssh -i ~/.ssh/work_key user@server
```

## 密钥安全

### 保护私钥

```bash
# 设置正确权限
chmod 600 ~/.ssh/id_ed25519

# 设置目录权限
chmod 700 ~/.ssh
```

### 锁定 authorized_keys

```bash
# 只读权限
chmod 400 ~/.ssh/authorized_keys

# 设置不可变属性（需要 root）
sudo chattr +i ~/.ssh/authorized_keys
```

> ⚠️ **警告**：这只能防止用户误操作，不能防止恶意程序。

### 使用硬件密钥

**YubiKey / Nitrokey**：
- 私钥存储在硬件中
- 无法导出
- 需要物理接触

## 双因素认证

### 配置 PAM + 公钥

**`/etc/ssh/sshd_config.d/20-pam.conf`**：

```bash
KbdInteractiveAuthentication yes
AuthenticationMethods publickey,keyboard-interactive:pam
```

### Google Authenticator

1. 安装 Google Authenticator
2. 配置 PAM
3. SSH 需要公钥 + 动态验证码

## 证书认证

### 创建 CA 密钥

```bash
ssh-keygen -t ed25519 -f ~/.ssh/ca_host_key -C 'Host CA for *.example.com'
```

### 签署主机密钥

```bash
ssh-keygen -h -s ~/.ssh/ca_host_key -I certLabel -n server01.example.com ./ssh_host_ed25519_key.pub
```

### 配置客户端信任 CA

**`~/.ssh/known_hosts`**：

```
@cert-authority *.example.com ssh-ed25519 AAAA...
```

## 常见问题

### 问题 1：权限错误

**错误**：`Permissions 0644 for '~/.ssh/id_rsa' are too open.`

**解决**：
```bash
chmod 600 ~/.ssh/id_rsa
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 问题 2：仍提示输入密码

**检查**：
```bash
# 查看 SSH 详细输出
ssh -v user@server
```

**可能原因**：
- 公钥未正确添加到服务器
- 服务器配置禁用公钥认证
- 权限不正确

### 问题 3：密钥太多

**解决**：
```bash
# 在 ~/.ssh/config 中指定
Host *
    IdentitiesOnly yes
```

## 最佳实践

### 1. 使用 ED25519

```bash
# ✅ 推荐
ssh-keygen -t ed25519

# ❌ 避免：DSA（已废弃）
ssh-keygen -t dsa
```

### 2. 使用密码短语

```bash
# ✅ 推荐：设置密码短语
ssh-keygen -t ed25519

# ❌ 避免：空密码
ssh-keygen -t ed25519 -N ""
```

### 3. 定期轮换密钥

```bash
# 每年更换一次
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_$(date +%Y)
```

### 4. 备份私钥

```bash
# 备份到安全位置
cp ~/.ssh/id_ed25519 /secure/backup/
```

## 命令速查

| 操作 | 命令 |
|------|------|
| 生成密钥 | `ssh-keygen -t ed25519` |
| 复制公钥 | `ssh-copy-id user@host` |
| 查看指纹 | `ssh-keygen -lf ~/.ssh/id.pub` |
| 启动代理 | `eval "$(ssh-agent -s)"` |
| 添加密钥 | `ssh-add ~/.ssh/key` |
| 查看密钥 | `ssh-add -l` |

## 相关链接

- [[SSH-MOC]] - 返回 SSH 知识地图
- [[SSH-安装配置]] - SSH 配置
- [[SSH-安全加固]] - 安全配置

## 外部资源

- [ssh-keygen(1)](https://man.archlinux.org/man/ssh-keygen.1)
- [ArchWiki - SSH keys](https://wiki.archlinux.org/title/SSH_keys)
