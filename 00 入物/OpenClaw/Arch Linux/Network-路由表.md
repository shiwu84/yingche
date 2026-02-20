---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, network, atomic, routing]
source: https://wiki.archlinux.org/title/Network_configuration
aliases: [路由表，网关，ip route]
---

# Network 路由表

> [!summary] 核心概念
> 路由表决定数据包如何到达目标地址，包括直接连接和通过网关。

## 什么是路由表

路由表用于确定：
- 是否可以**直接**到达 IP 地址
- 应该使用哪个**网关**（路由器）
- 如果没有匹配的路由，使用**默认网关**

## 查看路由表

### 查看 IPv4 路由

```bash
ip route show
```

简写：
```bash
ip r
```

**示例输出：**
```
default via 192.168.1.1 dev enp0s25 proto dhcp src 192.168.1.100 metric 100
192.168.1.0/24 dev enp0s25 proto kernel scope link src 192.168.1.100
```

### 查看 IPv6 路由

```bash
ip -6 route show
```

### 详细路由信息

```bash
ip route show table all
```

## 路由表条目解析

### 默认路由

```
default via 192.168.1.1 dev enp0s25
```

**含义：**
- `default` - 默认路由（0.0.0.0/0）
- `via 192.168.1.1` - 通过网关 192.168.1.1
- `dev enp0s25` - 使用接口 enp0s25

### 本地网络路由

```
192.168.1.0/24 dev enp0s25 proto kernel scope link
```

**含义：**
- `192.168.1.0/24` - 本地网络
- `dev enp0s25` - 直接通过接口
- `proto kernel` - 由内核自动添加
- `scope link` - 链路范围

## 管理路由

### 添加路由

```bash
# 添加默认网关
sudo ip route add default via 192.168.1.1

# 添加特定网络路由
sudo ip route add 10.0.0.0/8 via 192.168.1.254

# 添加主机路由
sudo ip route add 192.168.2.100 via 192.168.1.1
```

### 删除路由

```bash
# 删除默认路由
sudo ip route del default

# 删除特定路由
sudo ip route del 10.0.0.0/8 via 192.168.1.254
```

### 修改路由

```bash
# 替换默认网关
sudo ip route replace default via 192.168.1.254
```

## 多网关配置

### 添加多个默认路由

```bash
# 主网关（优先级高）
sudo ip route add default via 192.168.1.1 metric 100

# 备用网关（优先级低）
sudo ip route add default via 192.168.1.2 metric 200
```

**metric 值越小优先级越高。**

### 策略路由

```bash
# 创建自定义路由表
echo "100 custom" | sudo tee -a /etc/iproute2/rt_tables

# 添加规则
sudo ip rule add from 192.168.1.100 table custom

# 添加路由到自定义表
sudo ip route add default via 192.168.1.1 table custom
```

## 常见路由场景

### 场景 1：单网关

```bash
# 默认网关
ip route add default via 192.168.1.1
```

### 场景 2：多网络接口

```bash
# 内网
ip route add 10.0.0.0/8 dev enp0s25

# 外网
ip route add default via 192.168.1.1
```

### 场景 3：VPN 路由

```bash
# 默认走本地
ip route add default via 192.168.1.1

# 特定流量走 VPN
ip route add 10.8.0.0/24 dev tun0
```

## 持久化路由配置

### systemd-networkd

创建 `/etc/systemd/network/20-wired.network`：

```ini
[Match]
Name=enp0s25

[Network]
DHCP=no
Address=192.168.1.100/24
Gateway=192.168.1.1

# 静态路由
[Route]
Destination=10.0.0.0/8
Gateway=192.168.1.254
Metric=100
```

### NetworkManager

```bash
# 添加静态路由
nmcli connection modify "Wired" +ipv4.routes "10.0.0.0/8 192.168.1.254"

# 应用配置
nmcli connection up "Wired"
```

### netctl

创建 `/etc/netctl/ethernet-static`：

```bash
Routes=('10.0.0.0/8 via 192.168.1.254')
```

## 路由调试

### 测试路由

```bash
# ping 测试
ping -c 4 8.8.8.8

# 跟踪路由
traceroute 8.8.8.8

# 查看路由决策
ip route get 8.8.8.8
```

### 查看路由缓存

```bash
# 查看路由缓存
ip route show cache
```

### 刷新路由缓存

```bash
# 刷新缓存
sudo ip route flush cache
```

## 常见问题

### 问题 1：无法访问外网

**症状：** 可以 ping 通网关，无法 ping 通外网。

**检查：**
```bash
# 查看默认路由
ip route show | grep default

# 测试网关
ping 192.168.1.1

# 测试外网
ping 8.8.8.8
```

**解决：**
```bash
# 添加默认路由
sudo ip route add default via 192.168.1.1
```

### 问题 2：路由冲突

**症状：** 网络不稳定，时通时断。

**解决：**
```bash
# 查看冲突路由
ip route show

# 删除冲突路由
sudo ip route del 10.0.0.0/8 via 192.168.1.254

# 添加正确路由
sudo ip route add 10.0.0.0/8 via 192.168.1.1
```

### 问题 3：重启后路由丢失

**解决：** 使用 systemd-networkd 或 NetworkManager 持久化配置。

## 高级路由

### 源路由

```bash
# 基于源地址路由
ip rule add from 192.168.1.100 table 100
ip route add default via 192.168.1.1 table 100
```

### 负载均衡

```bash
# 多路径路由
ip route add default \
  nexthop via 192.168.1.1 weight 1 \
  nexthop via 192.168.1.2 weight 1
```

## 命令速查

| 操作 | 命令 |
|------|------|
| 查看路由 | `ip route show` |
| 添加默认路由 | `sudo ip route add default via GATEWAY` |
| 添加静态路由 | `sudo ip route add NETWORK via GATEWAY` |
| 删除路由 | `sudo ip route del ROUTE` |
| 查看路由决策 | `ip route get IP` |
| 刷新缓存 | `sudo ip route flush cache` |

## 相关链接

- [[Network-MOC]] - 返回网络知识地图
- [[Network-IP 地址配置]] - IP 地址配置
- [[Network-网络诊断]] - 网络故障排除

## 外部资源

- [ip-route(8)](https://man.archlinux.org/man/ip-route.8)
- [ArchWiki - Routing](https://wiki.archlinux.org/title/Network_configuration#Routing_table)
