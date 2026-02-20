---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, network, atomic, troubleshooting]
source: https://wiki.archlinux.org/title/Network_configuration
aliases: [网络诊断，ping,traceroute，网络故障排除]
---

# Network 网络诊断

> [!summary] 核心概念
> 使用 ping、traceroute 等工具诊断网络连接问题。

## 检查连接步骤

### 1. 检查网络接口

```bash
# 查看接口列表
ip link show

# 查看接口状态
ip -s link show enp0s25
```

**确保：**
- 接口已列出
- 接口状态为 `UP`

### 2. 检查物理连接

**有线网络：**
- 网线已插入
- 路由器/交换机已通电

**无线网络：**
- 已连接到 WiFi
- 信号强度足够

### 3. 检查 IP 地址

```bash
# 查看 IP 地址
ip address show

# 查看特定接口
ip address show dev enp0s25
```

**确保：**
- 接口有 IP 地址
- IP 地址在正确网段

### 4. 检查路由表

```bash
# 查看路由表
ip route show

# 查看默认网关
ip route show | grep default
```

**确保：**
- 有默认路由
- 网关地址正确

### 5. Ping 测试

```bash
# Ping 本地网关
ping -c 4 192.168.1.1

# Ping 公共 DNS
ping -c 4 8.8.8.8

# Ping 域名
ping -c 4 archlinux.org
```

### 6. 检查 DNS

```bash
# 查看 DNS 配置
cat /etc/resolv.conf

# 测试域名解析
nslookup archlinux.org
```

## ping 命令

### 基本用法

```bash
# Ping 主机
ping www.example.com

# 指定次数
ping -c 4 www.example.com

# 指定间隔
ping -i 2 www.example.com

# 快速 ping
ping -f www.example.com
```

### 输出解读

```
PING www.example.com (93.184.216.34) 56(84) bytes of data.
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=1 ttl=56 time=11.6 ms
64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=2 ttl=56 time=11.7 ms
```

**字段说明：**
- `icmp_seq` - ICMP 序列号
- `ttl` - 生存时间
- `time` - 往返时间（延迟）

### 常见错误

| 错误 | 含义 |
|------|------|
| `Destination Host Unreachable` | 目标不可达 |
| `Request timeout` | 请求超时 |
| `Network is unreachable` | 网络不可达 |
| `Unknown host` | 域名解析失败 |

## traceroute 命令

### 安装

```bash
sudo pacman -S traceroute
```

### 基本用法

```bash
# 跟踪路由
traceroute www.example.com

# 指定最大跳数
traceroute -m 20 www.example.com

# 使用 ICMP
traceroute -I www.example.com
```

### 输出解读

```
traceroute to www.example.com (93.184.216.34), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  2.123 ms  1.987 ms  2.045 ms
 2  10.0.0.1 (10.0.0.1)  10.234 ms  10.123 ms  10.456 ms
 3  * * *
 4  93.184.216.34 (93.184.216.34)  11.234 ms  11.123 ms  11.456 ms
```

**字段说明：**
- `1, 2, 3...` - 跳数
- `192.168.1.1` - 路由器 IP
- `2.123 ms` - 往返时间
- `*` - 无响应（可能被防火墙阻止）

## mtr 命令

### 安装

```bash
sudo pacman -S mtr
```

### 基本用法

```bash
# 交互式
mtr www.example.com

# 生成报告
mtr -r -c 10 www.example.com

# 使用 TCP
mtr -T www.example.com
```

### 输出解读

```
Host              Loss%   Snt   Last   Avg  Best  Wrst StDev
192.168.1.1       0.0%    10    2.1    2.0  1.8   2.5   0.2
10.0.0.1          0.0%    10    10.2   10.1 9.8   10.5  0.3
```

**字段说明：**
- `Loss%` - 丢包率
- `Snt` - 发送的包数
- `Last` - 最后一次延迟
- `Avg` - 平均延迟
- `Best/Wrst` - 最好/最差延迟
- `StDev` - 标准差

## 其他诊断工具

### netstat

```bash
# 查看网络连接
netstat -tuln

# 查看路由表
netstat -rn

# 查看接口统计
netstat -i
```

### ss（netstat 替代品）

```bash
# 查看连接
ss -tuln

# 查看统计
ss -s
```

### dig

```bash
# 查询 DNS
dig www.example.com

# 指定 DNS 服务器
dig @8.8.8.8 www.example.com

# 查询 MX 记录
dig www.example.com MX
```

### arp

```bash
# 查看 ARP 缓存
arp -a

# 查看特定接口
arp -i enp0s25
```

### nmap

```bash
# 安装
sudo pacman -S nmap

# 扫描主机
nmap 192.168.1.1

# 扫描网段
nmap 192.168.1.0/24

# 检测操作系统
nmap -O 192.168.1.1
```

## 常见问题诊断

### 问题 1：无法上网

**诊断步骤：**

```bash
# 1. 检查接口
ip link show

# 2. 检查 IP
ip address show

# 3. Ping 网关
ping 192.168.1.1

# 4. Ping 外网 IP
ping 8.8.8.8

# 5. Ping 域名
ping www.example.com

# 6. 检查 DNS
cat /etc/resolv.conf
```

**可能原因：**
- 接口未启用
- 没有 IP 地址
- 网关配置错误
- DNS 配置错误

### 问题 2：DNS 解析失败

**症状：** 可以 ping 通 IP，无法 ping 通域名。

**诊断：**
```bash
# 测试 DNS
nslookup www.example.com

# 查看 DNS 配置
cat /etc/resolv.conf
```

**解决：**
```bash
# 修改 DNS
sudo nano /etc/resolv.conf
# 添加：nameserver 8.8.8.8
```

### 问题 3：连接不稳定

**症状：** 时断时续，丢包严重。

**诊断：**
```bash
# 持续 ping 测试
ping -c 100 192.168.1.1

# 跟踪路由
mtr 192.168.1.1
```

**可能原因：**
- 网线质量问题
- WiFi 信号弱
- 路由器问题
- 网络拥塞

### 问题 4：速度慢

**诊断：**
```bash
# 测试延迟
ping -c 10 www.example.com

# 测试带宽
# 使用 speedtest-cli
sudo pacman -S speedtest-cli
speedtest
```

**可能原因：**
- 网络拥塞
- WiFi 干扰
- 路由器性能
- ISP 限速

## 无线诊断

### 查看 WiFi 信号

```bash
# 扫描网络
iw dev wlan0 scan | grep -E "SSID|signal"

# 查看连接质量
iw dev wlan0 link
```

### 查看干扰

```bash
# 安装
sudo pacman -S wavemon

# 运行
wavemon
```

显示 WiFi 信号强度和干扰情况。

## 日志分析

### NetworkManager 日志

```bash
journalctl -u NetworkManager -f
```

### systemd-networkd 日志

```bash
journalctl -u systemd-networkd -f
```

### 内核网络日志

```bash
dmesg | grep -i network
dmesg | grep -i ethernet
dmesg | grep -i wifi
```

## 诊断脚本

### 快速诊断

```bash
#!/bin/bash
echo "=== Network Diagnostics ==="
echo ""
echo "1. Interface Status:"
ip link show | grep -E "^[0-9]+:"
echo ""
echo "2. IP Address:"
ip address show | grep "inet "
echo ""
echo "3. Default Gateway:"
ip route show | grep default
echo ""
echo "4. DNS Servers:"
cat /etc/resolv.conf | grep nameserver
echo ""
echo "5. Ping Test:"
ping -c 2 8.8.8.8
```

## 最佳实践

### 1. 系统化诊断

```bash
# ✅ 推荐：从下到上
# 物理层 → 数据链路层 → 网络层 → 传输层 → 应用层
```

### 2. 记录基线

```bash
# ✅ 推荐：记录正常时的状态
# - 正常延迟
# - 正常带宽
# - 路由路径
```

### 3. 使用合适工具

```bash
# ✅ 连通性：ping
# ✅ 路由：traceroute
# ✅ DNS：dig/nslookup
# ✅ 综合：mtr
```

## 命令速查

| 操作 | 命令 |
|------|------|
| 测试连通性 | `ping host` |
| 跟踪路由 | `traceroute host` |
| 综合诊断 | `mtr host` |
| DNS 查询 | `dig domain` |
| 查看连接 | `ss -tuln` |
| 查看接口 | `ip link show` |
| 查看 IP | `ip address show` |
| 查看路由 | `ip route show` |

## 相关链接

- [[Network-MOC]] - 返回网络知识地图
- [[Network-IP 地址配置]] - IP 配置
- [[Network-DNS 配置]] - DNS 配置

## 外部资源

- [ping(8)](https://man.archlinux.org/man/ping.8)
- [traceroute(8)](https://man.archlinux.org/man/traceroute.8)
- [ArchWiki - Network troubleshooting](https://wiki.archlinux.org/title/Network_configuration#Troubleshooting)
