---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, network, atomic, manager]
source: https://wiki.archlinux.org/title/Network_configuration
aliases: [NetworkManager,systemd-networkd，网络管理工具]
---

# Network 网络管理器

> [!summary] 核心概念
> 网络管理器简化网络连接配置，支持 profiles 切换，自动处理 DHCP 和认证。

## 常见网络管理器

| 管理器 | CLI | TUI | GUI | 适用场景 |
|--------|-----|-----|-----|---------|
| **NetworkManager** | nmcli | nmtui | ✅ | 桌面环境 |
| **systemd-networkd** | networkctl | ❌ | ❌ | 服务器 |
| **netctl** | netctl | wifi-menu | ❌ | 简单配置 |
| **ConnMan** | connmanctl | ❌ | ✅ | 轻量级 |
| **iwd** | iwctl | impala | iwgtk | 无线网络 |

## NetworkManager

### 安装

```bash
sudo pacman -S networkmanager
```

### 启用服务

```bash
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
```

### CLI 命令（nmcli）

```bash
# 查看连接状态
nmcli connection show

# 查看设备状态
nmcli device status

# 连接网络
nmcli connection up "Wired connection 1"

# 断开网络
nmcli connection down "Wired connection 1"

# 查看 WiFi
nmcli device wifi list

# 连接 WiFi
nmcli device wifi connect "SSID" password "PASSWORD"
```

### TUI 界面（nmtui）

```bash
# 启动文本界面
nmtui
```

提供图形化的配置界面。

### 配置静态 IP

```bash
# 修改连接
nmcli connection modify "Wired" ipv4.method manual
nmcli connection modify "Wired" ipv4.addresses 192.168.1.100/24
nmcli connection modify "Wired" ipv4.gateway 192.168.1.1
nmcli connection modify "Wired" ipv4.dns "8.8.8.8 8.8.4.4"

# 应用配置
nmcli connection up "Wired"
```

### 删除连接

```bash
nmcli connection delete "Connection Name"
```

## systemd-networkd

### 安装

```bash
# 已包含在 systemd 中
sudo pacman -S systemd
```

### 启用服务

```bash
sudo systemctl enable systemd-networkd
sudo systemctl start systemd-networkd

# 可选：启用名称解析
sudo systemctl enable systemd-resolved
sudo systemctl start systemd-resolved
```

### 配置文件

`/etc/systemd/network/20-wired.network`：

```ini
[Match]
Name=enp0s25

[Network]
DHCP=yes
```

### 静态 IP 配置

```ini
[Match]
Name=enp0s25

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
DNS=8.8.8.8
DNS=8.8.4.4
```

### 查看状态

```bash
# 查看网络状态
networkctl status

# 查看特定接口
networkctl status enp0s25

# 列出所有接口
networkctl list
```

### 重新加载配置

```bash
sudo networkctl reload
```

## netctl

### 安装

```bash
sudo pacman -S netctl
```

### 配置文件

`/etc/netctl/ethernet-dhcp`：

```bash
Interface=enp0s25
Connection=ethernet
IP=dhcp
```

### 静态 IP 配置

`/etc/netctl/ethernet-static`：

```bash
Interface=enp0s25
Connection=ethernet
Address=('192.168.1.100/24')
Gateway='192.168.1.1'
DNS=('8.8.8.8' '8.8.4.4')
```

### 启动配置

```bash
# 启动配置
sudo netctl start ethernet-dhcp

# 开机启动
sudo netctl enable ethernet-dhcp

# 启动并启用
sudo netctl restart ethernet-dhcp
```

### WiFi 配置

```bash
# 扫描网络
sudo wifi-menu

# 选择网络并输入密码
# 自动创建配置文件
```

## iwd（无线）

### 安装

```bash
sudo pacman -S iwd
```

### 启用服务

```bash
sudo systemctl enable iwd
sudo systemctl start iwd
```

### CLI 命令（iwctl）

```bash
# 进入交互模式
iwctl

# 扫描网络
station wlan0 scan

# 列出网络
station wlan0 get-networks

# 连接网络
station wlan0 connect SSID

# 退出
exit
```

### 启用内置网络配置

`/etc/iwd/main.conf`：

```ini
[General]
EnableNetworkConfiguration=true
```

## ConnMan

### 安装

```bash
sudo pacman -S connman connman-gtk
```

### 启用服务

```bash
sudo systemctl enable connman
sudo systemctl start connman
```

### CLI 命令（connmanctl）

```bash
# 进入交互模式
connmanctl

# 扫描服务
scan wifi

# 列出服务
services

# 连接服务
connect wifi_xxxx_managed_psk

# 退出
quit
```

## 选择指南

### 桌面环境

**推荐：NetworkManager**

```bash
# ✅ 优点
# - 完整的 GUI 支持
# - 自动连接已知网络
# - VPN 支持
# - 移动宽带支持
```

### 服务器

**推荐：systemd-networkd**

```bash
# ✅ 优点
# - 轻量级
# - 与 systemd 集成
# - 配置简单
# - 资源占用少
```

### 最小化系统

**推荐：netctl 或 iwd**

```bash
# ✅ 优点
# - 简单直接
# - 配置清晰
# - 依赖少
```

## 多管理器冲突

> ⚠️ **Warning**: 每个网络接口应该只由一个管理器管理。

### 检查冲突

```bash
# 查看运行的服务
systemctl status NetworkManager
systemctl status systemd-networkd
systemctl status netctl
```

### 解决冲突

```bash
# 禁用不需要的服务
sudo systemctl disable --now systemd-networkd
sudo systemctl disable --now netctl

# 启用需要的服务
sudo systemctl enable --now NetworkManager
```

## 网络 Profiles

### NetworkManager Profiles

```bash
# 列出 profiles
nmcli connection show

# 创建新 profile
nmcli connection add type ethernet con-name "office"

# 修改 profile
nmcli connection modify "office" ipv4.addresses 10.0.0.100/24

# 切换 profile
nmcli connection up "office"
```

### netctl Profiles

```bash
# 配置文件位置
ls /etc/netctl/

# 切换配置
sudo netctl stop ethernet-dhcp
sudo netctl start ethernet-static
```

## 调试技巧

### NetworkManager 日志

```bash
journalctl -u NetworkManager -f
```

### systemd-networkd 日志

```bash
journalctl -u systemd-networkd -f
```

### 查看配置

```bash
# NetworkManager
nmcli connection show "Wired"

# systemd-networkd
networkctl status enp0s25

# netctl
netctl list
```

## 最佳实践

### 1. 只运行一个管理器

```bash
# ✅ 推荐：单一管理器
sudo systemctl enable --now NetworkManager

# ❌ 避免：多个管理器
# NetworkManager + systemd-networkd
```

### 2. 使用合适的工具

```bash
# ✅ 桌面：NetworkManager
# ✅ 服务器：systemd-networkd
# ✅ 简单：netctl
```

### 3. 备份配置

```bash
# NetworkManager
cp -r /etc/NetworkManager/system-connections/ ~/backup/

# systemd-networkd
cp /etc/systemd/network/*.network ~/backup/
```

## 相关链接

- [[Network-MOC]] - 返回网络知识地图
- [[Network-IP 地址配置]] - IP 配置
- [[Network-无线网络]] - WiFi 配置

## 外部资源

- [NetworkManager](https://wiki.archlinux.org/title/NetworkManager)
- [systemd-networkd](https://wiki.archlinux.org/title/Systemd-networkd)
- [netctl](https://wiki.archlinux.org/title/Netctl)
