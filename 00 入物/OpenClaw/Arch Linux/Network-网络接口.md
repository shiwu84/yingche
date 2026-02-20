---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, network, atomic, interface]
source: https://wiki.archlinux.org/title/Network_configuration
aliases: [网络接口，网卡命名，udev 网络]
---

# Network 网络接口

> [!summary] 核心概念
> 网络接口由 udev 管理，systemd.link 配置文件控制命名规则。

## 什么是网络接口

网络接口是系统与网络之间的连接点，类型包括：
- **Ethernet**（有线网络）- `en` 前缀
- **WLAN**（无线网络）- `wl` 前缀
- **WWAN**（移动宽带）- `ww` 前缀

## 网络接口命名

### Predictable Network Interface Names

systemd 使用可预测的网络接口命名规则：

**前缀规则：**

| 前缀 | 类型 | 示例 |
|------|------|------|
| `en` | Ethernet（有线） | `enp0s25`, `eno1` |
| `wl` | WLAN（无线） | `wlp2s0`, `wlan0` |
| `ww` | WWAN（移动宽带） | `wwp0s20u1` |

### 命名方案

**常见格式：**

1. **基于物理位置**
   - `eno1` - 板载设备
   - `ens1` - PCI Express 热插拔插槽
   - `enp0s25` - PCI 总线位置

2. **基于 MAC 地址**
   - `enx78e7d1ea46da` - 使用 MAC 地址

3. **传统命名**
   - `eth0` - 第一个以太网接口
   - `wlan0` - 第一个无线接口

### 查看接口名称

```bash
# 列出所有网络接口
ip link show

# 查看接口详细信息
ip -s link

# 查看接口状态
ip link show dev enp0s25
```

## 管理网络接口

### 启用/禁用接口

```bash
# 启用接口
sudo ip link set enp0s25 up

# 禁用接口
sudo ip link set enp0s25 down
```

### 查看接口状态

```bash
# 查看状态
ip link show enp0s25

# 查看统计信息
ip -s link show enp0s25
```

### 更改 MTU

```bash
# 设置 MTU
sudo ip link set enp0s25 mtu 9000

# 查看 MTU
ip link show enp0s25 | grep mtu
```

## 修改接口名称

### 使用 .link 文件

创建 `/etc/systemd/network/10-custom-name.link`：

```ini
[Match]
MACAddress=00:11:22:33:44:55

[Link]
Name=my-eth0
```

应用更改：
```bash
sudo udevadm test-builtin net_setup_link /sys/class/net/enp0s25
sudo reboot
```

### 恢复传统命名

**方法 1：内核参数**

在 GRUB 中添加：
```
net.ifnames=0 biosdevname=0
```

**方法 2：屏蔽 systemd 命名**

创建 `/etc/systemd/network/99-default.link`：

```ini
[Match]
Type=network

[Link]
NamePolicy=kernel database slot path
```

## 接口类型

### 有线网络（Ethernet）

**特点：**
- 稳定可靠
- 速度快
- 需要物理连接

**驱动检查：**
```bash
# 查看网卡驱动
lspci -k | grep -A 3 Ethernet

# 查看驱动模块
ethtool -i enp0s25
```

### 无线网络（WLAN）

**特点：**
- 灵活方便
- 需要认证
- 速度受环境影响

**驱动检查：**
```bash
# 查看无线网卡
lspci -k | grep -A 3 Wireless

# 查看接口
iw dev
```

### 移动宽带（WWAN）

**特点：**
- 使用蜂窝网络
- 需要 SIM 卡
- 按流量计费

## 常见问题

### 问题 1：接口未列出

**原因：**
- 驱动未加载
- 硬件被禁用
- 接口被 down 掉

**解决：**
```bash
# 检查驱动
lspci -k

# 加载驱动模块
sudo modprobe e1000e

# 启用接口
sudo ip link set enp0s25 up
```

### 问题 2：接口名称变化

**原因：** 添加或移除 PCIe 设备后，固件可能重新编号。

**解决：** 使用基于 MAC 地址的命名或固定 .link 文件。

### 问题 3：iwd 导致命名失效

**原因：** iwd 包包含禁用可预测命名的 .link 文件。

**解决：**
```bash
# 检查 iwd 配置
cat /usr/lib/systemd/network/99-default.link
```

## udev 调试

### 测试 .link 文件

```bash
# 测试网络设置链接
sudo udevadm test-builtin net_setup_link /sys/class/net/enp0s25
```

### 查看 udev 规则

```bash
# 查看网络相关的 udev 规则
udevadm info -a -p /sys/class/net/enp0s25
```

### 重新加载 udev 规则

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

## 接口配置文件位置

| 位置 | 用途 |
|------|------|
| `/usr/lib/systemd/network/` | 系统默认配置 |
| `/etc/systemd/network/` | 用户自定义配置 |
| `/usr/lib/udev/rules.d/` | 系统 udev 规则 |
| `/etc/udev/rules.d/` | 用户 udev 规则 |

## 最佳实践

### 1. 使用可预测命名

```bash
# ✅ 推荐：可预测命名
enp0s25

# ❌ 避免：传统命名（可能冲突）
eth0
```

### 2. 固定重要接口名称

```ini
# /etc/systemd/network/10-server-eth.link
[Match]
MACAddress=00:11:22:33:44:55

[Link]
Name=server-eth0
```

### 3. 仅启用需要的接口

```bash
# 禁用不用的接口
sudo ip link set enp0s26 down
```

## 相关链接

- [[Network-MOC]] - 返回网络知识地图
- [[Network-IP 地址配置]] - IP 地址配置
- [[Network-网络管理器]] - 网络管理工具

## 外部资源

- [systemd.net-naming-scheme(7)](https://man.archlinux.org/man/systemd.net-naming-scheme.7)
- [systemd.link(5)](https://man.archlinux.org/man/systemd.link.5)
- [Predictable Interface Names](https://systemd.io/PREDICTABLE_INTERFACE_NAMES/)
