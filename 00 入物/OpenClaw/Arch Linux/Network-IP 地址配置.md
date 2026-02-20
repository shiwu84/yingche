---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, network, atomic, ip-address]
source: https://wiki.archlinux.org/title/Network_configuration
aliases: [IP 配置，静态 IP,DHCP,iproute2]
---

# Network IP 地址配置

> [!summary] 核心概念
> 使用 iproute2 工具管理 IP 地址，支持静态 IP 和 DHCP 自动配置。

## iproute2 工具

iproute2 是 base 元包的依赖，提供 `ip` 命令：
- 管理网络接口
- 配置 IP 地址
- 管理路由表

> ⚠️ **Warning**: 使用 `ip` 命令的配置在重启后会丢失，需要 systemd units 或脚本持久化。

## 查看 IP 地址

### 列出所有 IP 地址

```bash
ip address show
```

简写：
```bash
ip a show
ip a
```

### 查看特定接口

```bash
ip address show dev enp0s25
```

### 查看 IPv6 地址

```bash
ip -6 address show
```

## 配置 IP 地址

### 添加 IP 地址

```bash
sudo ip address add 192.168.1.100/24 broadcast + dev enp0s25
```

**参数说明：**
- `192.168.1.100/24` - CIDR 格式的 IP 地址和子网掩码
- `broadcast +` - 自动计算广播地址
- `dev enp0s25` - 网络接口名称

### 删除 IP 地址

```bash
sudo ip address del 192.168.1.100/24 dev enp0s25
```

### 删除所有 IP 地址

```bash
sudo ip address flush dev enp0s25
```

## CIDR 表示法

| CIDR | 子网掩码 | 可用主机 |
|------|---------|---------|
| /8 | 255.0.0.0 | 16,777,214 |
| /16 | 255.255.0.0 | 65,534 |
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /30 | 255.255.255.252 | 2 |

## 静态 IP 配置

### 手动配置步骤

1. **添加 IP 地址**
```bash
sudo ip address add 192.168.1.100/24 dev enp0s25
```

2. **设置默认网关**
```bash
sudo ip route add default via 192.168.1.1
```

3. **配置 DNS**
```bash
# 编辑 /etc/resolv.conf
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

### 持久化静态 IP

**使用 systemd-networkd：**

创建 `/etc/systemd/network/20-wired.network`：

```ini
[Match]
Name=enp0s25

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
DNS=8.8.8.8
DNS=8.8.4.4
```

启用服务：
```bash
sudo systemctl enable systemd-networkd
sudo systemctl start systemd-networkd
```

## DHCP 自动配置

### 使用 dhcpcd

```bash
# 安装
sudo pacman -S dhcpcd

# 启用服务（所有接口）
sudo systemctl enable dhcpcd
sudo systemctl start dhcpcd

# 启用特定接口
sudo systemctl enable dhcpcd@enp0s25
sudo systemctl start dhcpcd@enp0s25
```

### 使用 dhclient

```bash
# 安装
sudo pacman -S dhclient

# 获取 IP
sudo dhclient enp0s25

# 释放 IP
sudo dhclient -r enp0s25
```

> ⚠️ **Warning**: dhclient 自 2022 年初已不再维护，建议使用 dhcpcd。

### 使用 NetworkManager

```bash
# 启动连接
nmcli connection up "Wired connection 1"

# 查看连接状态
nmcli connection show
```

## 多 IP 地址配置

### 添加多个 IP

```bash
# 主 IP
sudo ip address add 192.168.1.100/24 dev enp0s25

# 第二个 IP
sudo ip address add 192.168.1.101/24 dev enp0s25

# 第三个 IP
sudo ip address add 10.0.0.100/24 dev enp0s25
```

### 查看多 IP

```bash
ip address show dev enp0s25
```

## IPv6 配置

### 添加 IPv6 地址

```bash
sudo ip -6 address add 2001:db8::1/64 dev enp0s25
```

### 查看 IPv6

```bash
ip -6 address show
```

### IPv6 自动配置

```bash
# 启用 IPv6 自动配置
sudo sysctl -w net.ipv6.conf.all.autoconf=1
sudo sysctl -w net.ipv6.conf.all.accept_ra=1
```

## 临时 vs 持久配置

### 临时配置（重启丢失）

```bash
# 使用 ip 命令
sudo ip address add 192.168.1.100/24 dev enp0s25
```

### 持久配置

**方法 1：systemd-networkd**
```ini
# /etc/systemd/network/20-wired.network
[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
```

**方法 2：NetworkManager**
```bash
nmcli connection modify "Wired" ipv4.addresses 192.168.1.100/24
nmcli connection modify "Wired" ipv4.gateway 192.168.1.1
```

**方法 3：netctl**
```bash
# /etc/netctl/ethernet-static
Interface=enp0s25
Address=('192.168.1.100/24')
Gateway='192.168.1.1'
DNS=('8.8.8.8')
```

## IP 计算工具

### ipcalc

```bash
# 安装
sudo pacman -S ipcalc

# 计算网络信息
ipcalc 192.168.1.100/24
```

输出：
```
Address:   192.168.1.100
Netmask:   255.255.255.0 = 24
Network:   192.168.1.0/24
HostMin:   192.168.1.1
HostMax:   192.168.1.254
Broadcast: 192.168.1.255
```

## 常见问题

### 问题 1：IP 冲突

**症状：** 网络不稳定，提示 IP 冲突。

**解决：**
```bash
# 检查当前 IP
ip address show

# 更换 IP
sudo ip address del 192.168.1.100/24 dev enp0s25
sudo ip address add 192.168.1.150/24 dev enp0s25
```

### 问题 2：DHCP 获取失败

**解决：**
```bash
# 重启 dhcpcd
sudo systemctl restart dhcpcd

# 查看日志
journalctl -u dhcpcd -f
```

### 问题 3：配置不持久

**解决：** 使用 systemd-networkd 或 NetworkManager 配置持久化。

## 最佳实践

### 1. 服务器使用静态 IP

```ini
# ✅ 推荐：服务器固定 IP
Address=192.168.1.10/24
Gateway=192.168.1.1
```

### 2. 桌面使用 DHCP

```bash
# ✅ 推荐：桌面自动获取
sudo systemctl enable dhcpcd
```

### 3. 避免 IP 冲突

```bash
# ✅ 推荐：检查 DHCP 范围
# 在路由器设置中保留静态 IP 在 DHCP 范围外
```

## 命令速查

| 操作 | 命令 |
|------|------|
| 查看 IP | `ip address show` |
| 添加 IP | `sudo ip address add IP/CIDR dev IFACE` |
| 删除 IP | `sudo ip address del IP/CIDR dev IFACE` |
| 清空 IP | `sudo ip address flush dev IFACE` |
| 启用 DHCP | `sudo systemctl start dhcpcd` |
| 查看 IPv6 | `ip -6 address show` |

## 相关链接

- [[Network-MOC]] - 返回网络知识地图
- [[Network-路由表]] - 路由配置
- [[Network-DNS 配置]] - DNS 配置

## 外部资源

- [ip-address(8)](https://man.archlinux.org/man/ip-address.8)
- [systemd.network(5)](https://man.archlinux.org/man/systemd.network.5)
- [ArchWiki - DHCP](https://wiki.archlinux.org/title/Dhcp)
