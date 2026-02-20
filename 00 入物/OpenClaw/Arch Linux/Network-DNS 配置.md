---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, network, atomic, dns]
source: https://wiki.archlinux.org/title/Network_configuration
aliases: [DNS 配置，域名解析，resolv.conf]
---

# Network DNS 配置

> [!summary] 核心概念
> DNS 将域名解析为 IP 地址，通过 /etc/resolv.conf 配置 DNS 服务器。

## 什么是 DNS

DNS（Domain Name System）将域名转换为 IP 地址：
- `archlinux.org` → `95.217.163.246`
- `www.google.com` → `142.250.185.68`

## /etc/resolv.conf

### 查看当前 DNS

```bash
cat /etc/resolv.conf
```

**示例：**
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

### 配置 DNS 服务器

```bash
# 编辑文件
sudo nano /etc/resolv.conf

# 添加 DNS 服务器
nameserver 8.8.8.8
nameserver 1.1.1.1
```

### 多个 DNS 服务器

```bash
nameserver 8.8.8.8      # 主 DNS
nameserver 8.8.4.4      # 备用 DNS
nameserver 1.1.1.1      # 第三 DNS
```

系统按顺序尝试，第一个响应的 DNS 被使用。

## 常用 DNS 服务器

### 公共 DNS

| 提供商 | 主 DNS | 备用 DNS |
|--------|-------|---------|
| Google | 8.8.8.8 | 8.8.4.4 |
| Cloudflare | 1.1.1.1 | 1.0.0.1 |
| Quad9 | 9.9.9.9 | 149.112.112.112 |
| OpenDNS | 208.67.222.222 | 208.67.220.220 |

### 国内 DNS

| 提供商 | DNS |
|--------|-----|
| 阿里 DNS | 223.5.5.5, 223.6.6.6 |
| 腾讯 DNS | 119.29.29.29 |
| 百度 DNS | 180.76.76.76 |
| 114 DNS | 114.114.114.114 |

## DNS 管理工具

### systemd-resolved

```bash
# 查看 DNS 状态
resolvectl status

# 查看特定接口 DNS
resolvectl status enp0s25

# 临时设置 DNS
sudo resolvectl dns enp0s25 8.8.8.8 8.8.4.4
```

**持久化配置：**

`/etc/systemd/resolved.conf`：
```ini
[Resolve]
DNS=8.8.8.8 8.8.4.4
FallbackDNS=1.1.1.1
```

### NetworkManager

```bash
# 查看 DNS
nmcli dev show enp0s25 | grep DNS

# 设置 DNS
nmcli connection modify "Wired" ipv4.dns "8.8.8.8 8.8.4.4"

# 忽略 DHCP DNS
nmcli connection modify "Wired" ipv4.ignore-auto-dns yes

# 应用配置
nmcli connection up "Wired"
```

### systemd-networkd

`/etc/systemd/network/20-wired.network`：
```ini
[Network]
DNS=8.8.8.8
DNS=8.8.4.4
```

### dhcpcd

`/etc/dhcpcd.conf`：
```bash
# 使用 DHCP 提供的 DNS
# 默认行为

# 或指定静态 DNS
static domain_name_servers=8.8.8.8 8.8.4.4
```

## 防止 resolv.conf 被覆盖

### 方法 1：设置为不可变

```bash
# 创建静态文件
sudo nano /etc/resolv.conf

# 设置为不可变
sudo chattr +i /etc/resolv.conf

# 验证
lsattr /etc/resolv.conf
```

> ⚠️ **Warning**: 这会阻止所有程序修改 DNS，包括 NetworkManager。

### 方法 2：使用 systemd-resolved

```bash
# 启用服务
sudo systemctl enable systemd-resolved
sudo systemctl start systemd-resolved

# 创建符号链接
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

### 方法 3：使用 resolvconf

```bash
# 安装
sudo pacman -S resolvconf

# 配置网络管理器使用 resolvconf
```

## DNS 测试

### 使用 ping

```bash
# 测试域名解析
ping -c 4 archlinux.org
```

### 使用 dig

```bash
# 安装
sudo pacman -S bind

# 查询 A 记录
dig archlinux.org

# 查询 MX 记录
dig archlinux.org MX

# 指定 DNS 服务器
dig @8.8.8.8 archlinux.org
```

### 使用 nslookup

```bash
# 查询域名
nslookup archlinux.org

# 指定 DNS 服务器
nslookup archlinux.org 8.8.8.8
```

### 使用 host

```bash
# 简单查询
host archlinux.org

# 查询所有记录
host -a archlinux.org
```

## DNS 缓存

### systemd-resolved 缓存

```bash
# 查看缓存统计
resolvectl statistics

# 刷新缓存
sudo resolvectl flush-caches
```

### nscd 缓存

```bash
# 安装
sudo pacman -S nscd

# 重启服务
sudo systemctl restart nscd

# 刷新缓存
sudo nscd -i hosts
```

## 本地 DNS

### /etc/hosts

```bash
# 添加本地域名解析
sudo nano /etc/hosts
```

**示例：**
```
127.0.0.1   localhost
192.168.1.100   myserver.local
192.168.1.101   mynas.local
```

### mDNS

```bash
# 安装 avahi
sudo pacman -S avahi

# 启用服务
sudo systemctl enable avahi-daemon
sudo systemctl start avahi-daemon
```

使用 `.local` 域名：
```bash
ping myserver.local
```

## 常见问题

### 问题 1：无法解析域名

**症状：** 可以 ping 通 IP，无法 ping 通域名。

**检查：**
```bash
# 查看 DNS 配置
cat /etc/resolv.conf

# 测试 DNS
nslookup archlinux.org
```

**解决：**
```bash
# 修改 DNS
sudo nano /etc/resolv.conf
# 添加：nameserver 8.8.8.8
```

### 问题 2：DNS 泄露

**症状：** 使用 VPN 时 DNS 请求仍走本地。

**解决：**
```bash
# 使用 VPN 提供的 DNS
nmcli connection modify "VPN" ipv4.ignore-auto-dns yes
nmcli connection modify "VPN" ipv4.dns "10.8.0.1"
```

### 问题 3：DNS 污染

**症状：** 解析到错误的 IP。

**解决：**
```bash
# 使用加密 DNS（DNS over HTTPS）
# 安装 dnscrypt-proxy
sudo pacman -S dnscrypt-proxy
```

## DNS over HTTPS/TLS

### dnscrypt-proxy

```bash
# 安装
sudo pacman -S dnscrypt-proxy

# 配置
sudo nano /etc/dnscrypt-proxy/dnscrypt-proxy.toml

# 启用服务
sudo systemctl enable dnscrypt-proxy
sudo systemctl start dnscrypt-proxy
```

### systemd-resolved DoT

```ini
# /etc/systemd/resolved.conf
[Resolve]
DNS=8.8.8.8
DNSOverTLS=yes
```

## 最佳实践

### 1. 使用多个 DNS

```bash
# ✅ 推荐：主备 DNS
nameserver 8.8.8.8
nameserver 1.1.1.1
```

### 2. 选择快速 DNS

```bash
# 测试 DNS 速度
ping 8.8.8.8
ping 1.1.1.1
```

### 3. 使用加密 DNS

```bash
# ✅ 推荐：DNS over TLS
DNSOverTLS=yes
```

## 命令速查

| 操作 | 命令 |
|------|------|
| 查看 DNS | `cat /etc/resolv.conf` |
| 测试解析 | `nslookup domain` |
| 详细查询 | `dig domain` |
| 刷新缓存 | `sudo resolvectl flush-caches` |
| 查看状态 | `resolvectl status` |

## 相关链接

- [[Network-MOC]] - 返回网络知识地图
- [[Network-IP 地址配置]] - IP 地址配置
- [[Network-网络诊断]] - 网络故障排除

## 外部资源

- [resolv.conf(5)](https://man.archlinux.org/man/resolv.conf.5)
- [systemd-resolved.service(8)](https://man.archlinux.org/man/systemd-resolved.service.8)
- [ArchWiki - DNS](https://wiki.archlinux.org/title/Domain_name_resolution)
