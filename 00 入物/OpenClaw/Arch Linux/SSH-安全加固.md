---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, ssh, atomic, security, hardening]
source: https://wiki.archlinux.org/title/OpenSSH
aliases: [SSH 安全，sshd 加固，防暴力破解]
---

# SSH 安全加固

> [!summary] 核心概念
> 通过配置优化 SSH 安全性，防止暴力破解和未授权访问。

## 强制公钥认证

### 禁用密码登录

**`/etc/ssh/sshd_config.d/20-force_publickey_auth.conf`**：

```bash
PasswordAuthentication no
AuthenticationMethods publickey
```

> ⚠️ **警告**：配置前确保所有用户都已设置公钥认证！

### 测试配置

```bash
# 测试配置
sudo sshd -t

# 重启服务
sudo systemctl restart sshd

# 测试登录（不要关闭当前会话）
ssh user@server
```

## 限制 root 登录

### 禁用 root 登录（推荐）

**`/etc/ssh/sshd_config.d/20-deny_root.conf`**：

```bash
PermitRootLogin no
```

**优点**：
- 防止直接 root 访问
- 增加攻击难度
- 强制使用 sudo

### 仅允许密钥登录

```bash
PermitRootLogin prohibit-password
```

允许 root 使用公钥登录，禁用密码。

### 仅允许特定命令

**`~root/.ssh/authorized_keys`**：

```
command="rrsync -ro /" ssh-ed25519 AAAA...
```

**配置**：
```bash
PermitRootLogin forced-commands-only
```

## 限制用户访问

### 允许特定用户

```bash
AllowUsers admin deploy
```

### 允许特定用户组

```bash
AllowGroups sshusers wheel
```

### 拒绝特定用户

```bash
DenyUsers hacker
DenyGroups nogroup
```

## 修改默认端口

### 选择端口

```bash
# 查看已分配端口
cat /etc/services

# 选择未使用的端口（如 39901）
Port 39901
```

### 配置多端口

```bash
Port 22
Port 2222
```

**优点**：
- 减少自动化攻击
- 保留标准端口用于特定用途

**缺点**：
- 需要记住端口
- 不能消除针对性攻击

## 防止暴力破解

### OpenSSH 内置保护（9.8+）

```bash
# 默认已启用
PerSourcePenalties 5m:10,30m:100
```

**含义**：
- 10 次失败 → 封禁 5 分钟
- 100 次失败 → 封禁 30 分钟

### fail2ban

```bash
# 安装
sudo pacman -S fail2ban

# 启用 SSH 保护
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

**配置**：`/etc/fail2ban/jail.local`

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

### sshguard

```bash
# 安装
sudo pacman -S sshguard

# 启用服务
sudo systemctl enable sshguard
sudo systemctl start sshguard
```

### 防火墙限流

**ufw**：
```bash
# 限速连接
sudo ufw limit 22/tcp
```

**iptables**：
```bash
# 限制新连接
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP
```

## 禁用不安全的算法

### 配置安全算法

**`/etc/ssh/sshd_config.d/10-crypto.conf`**：

```bash
# 加密算法
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com

# 消息认证码
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

# 密钥交换
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
```

### 禁用弱算法

```bash
# 禁用旧的密钥交换
KexAlgorithms -diffie-hellman-group1-sha1

# 禁用弱加密
Ciphers -3des-cbc,-aes128-cbc

# 禁用弱 MAC
MACs -hmac-md5,-hmac-sha1
```

## 其他安全选项

### 登录限制

```bash
# 最大认证尝试
MaxAuthTries 3

# 最大会话数
MaxSessions 2

# 登录超时（秒）
LoginGraceTime 60

# 客户端活跃检查
ClientAliveInterval 300
ClientAliveCountMax 2
```

### 日志记录

```bash
# 详细日志
LogLevel VERBOSE

# 记录所有登录
SyslogFacility AUTH
```

### 转发限制

```bash
# 禁用 X11 转发
X11Forwarding no

# 禁用 TCP 转发
AllowTcpForwarding no

# 禁用代理转发
AllowAgentForwarding no
```

## 审计配置

### 使用 ssh-audit

```bash
# 安装
sudo pacman -S ssh-audit

# 审计服务器
ssh-audit server.example.com

# 审计客户端
ssh-audit -c client
```

### 检查配置

```bash
# 测试配置
sudo sshd -t

# 查看当前配置
sshd -T
```

## 监控和日志

### 查看 SSH 日志

```bash
# 实时查看
sudo journalctl -u sshd -f

# 查看失败登录
sudo journalctl -u sshd | grep "Failed"

# 查看成功登录
sudo journalctl -u sshd | grep "Accepted"
```

### 监控工具

```bash
# 查看当前连接
who

# 查看 SSH 进程
ps aux | grep sshd

# 查看网络连接
ss -tlnp | grep :22
```

## 完整安全配置示例

**`/etc/ssh/sshd_config.d/99-hardened.conf`**：

```bash
# 网络
Port 39901
AddressFamily inet
ListenAddress 0.0.0.0

# 认证
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthenticationMethods publickey
PermitEmptyPasswords no
ChallengeResponseAuthentication no

# 用户限制
AllowUsers admin deploy
AllowGroups wheel sshusers
MaxAuthTries 3
MaxSessions 2

# 转发
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no

# 超时
LoginGraceTime 60
ClientAliveInterval 300
ClientAliveCountMax 2

# 日志
LogLevel VERBOSE
SyslogFacility AUTH

# 加密
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256@libssh.org

# 其他
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
Compression no
UseDNS no
```

## 安全检查清单

- [ ] 禁用密码登录
- [ ] 禁用 root 登录
- [ ] 修改默认端口
- [ ] 限制允许的用户
- [ ] 启用 fail2ban 或 sshguard
- [ ] 配置防火墙
- [ ] 禁用不安全的算法
- [ ] 启用详细日志
- [ ] 定期审计配置
- [ ] 备份 authorized_keys

## 常见问题

### 问题 1：配置后无法登录

**解决**：
1. 不要关闭当前会话
2. 打开新终端测试
3. 如果失败，使用控制台修复
4. 恢复配置：`sudo sshd -t` 检查

### 问题 2：fail2ban 不工作

**检查**：
```bash
# 查看状态
systemctl status fail2ban

# 查看日志
journalctl -u fail2ban -f
```

### 问题 3：算法不匹配

**错误**：`no matching key exchange method found`

**解决**：客户端添加支持的算法：
```bash
ssh -o KexAlgorithms=+diffie-hellman-group1-sha1 user@host
```

## 最佳实践

### 1. 多层防护

```bash
# ✅ 推荐：纵深防御
# - 公钥认证
# - fail2ban
# - 防火墙
# - 修改端口
```

### 2. 定期审计

```bash
# 每月审计一次
ssh-audit server.example.com

# 检查配置
sudo sshd -T | grep -E "PermitRootLogin|PasswordAuthentication"
```

### 3. 监控日志

```bash
# 设置日志监控
tail -f /var/log/auth.log | grep -E "Failed|Accepted"
```

## 命令速查

| 操作 | 命令 |
|------|------|
| 测试配置 | `sudo sshd -t` |
| 审计服务器 | `ssh-audit host` |
| 查看日志 | `journalctl -u sshd -f` |
| 查看连接 | `who` |
| 重启服务 | `sudo systemctl restart sshd` |

## 相关链接

- [[SSH-MOC]] - 返回 SSH 知识地图
- [[SSH-密钥认证]] - 公钥认证
- [[SSH-安装配置]] - SSH 配置

## 外部资源

- [Mozilla Infosec SSH 指南](https://infosec.mozilla.org/guidelines/openssh.html)
- [SSH Hardening Guides](https://www.ssh-audit.com/hardening_guides.html)
- [ArchWiki - Security](https://wiki.archlinux.org/title/Security#SSH)
