---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, requirements, system, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 系统要求，ZeroClaw 环境要求]
---

# ZeroClaw-系统要求

> [!summary] 核心概念
> ZeroClaw 运行资源需求极低，但编译源码需要更多资源。推荐使用预编译二进制在低配置设备上运行。

## 运行要求

### 最低运行配置

| 资源 | 需求 |
|------|------|
| 内存 | <5MB RAM |
| 磁盘 | ~10MB（二进制 + 配置） |
| CPU | 任意 x86/ARM/RISC-V |
| 系统 | Linux/macOS/Windows |

### 推荐运行环境

- **硬件**：$10 级别的开发板（如 Raspberry Pi Zero）
- **内存**：64MB+ RAM
- **磁盘**：100MB+ 可用空间
- **网络**：用于 API 调用和更新

## 编译要求

### 最低编译配置

| 资源 | 需求 |
|------|------|
| 内存 + 交换 | 2GB |
| 可用磁盘 | 6GB |

### 推荐编译配置

| 资源 | 需求 |
|------|------|
| 内存 + 交换 | 4GB+ |
| 可用磁盘 | 10GB+ |

> [!warning] 资源不足
> 如果主机低于最低配置，请使用预编译二进制：
> ```bash
> ./bootstrap.sh --prefer-prebuilt
> ```

## 系统依赖

### Linux (Debian/Ubuntu)

```bash
# 安装构建工具
sudo apt install build-essential pkg-config

# 可选：Docker 沙箱运行时
sudo apt install docker.io
```

### Linux (Fedora/RHEL)

```bash
# 安装开发工具组
sudo dnf group install development-tools
sudo dnf install pkg-config

# 可选：Docker
sudo dnf install docker
```

### macOS

```bash
# 安装 Xcode 命令行工具
xcode-select --install

# 使用 Homebrew 安装（可选）
brew install pkg-config
```

### Windows

```powershell
# 安装 Visual Studio Build Tools（提供 MSVC 链接器和 Windows SDK）
winget install Microsoft.VisualStudio.2022.BuildTools

# 安装时选择"Desktop development with C++"工作负载

# 安装 Rust 工具链
winget install Rustlang.Rustup

# 验证安装
rustc --version
cargo --version

# 可选：Docker Desktop
winget install Docker.DockerDesktop
```

## Rust 工具链

### 安装 Rust

```bash
# Linux/macOS
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 安装后打开新终端并运行
rustup default stable
```

### 验证 Rust 安装

```bash
rustc --version
cargo --version
```

### Rust 版本要求

- **最低**：Rust 1.70+
- **推荐**：最新稳定版

## 可选依赖

### Docker 沙箱运行时

当配置 `runtime.kind = "docker"` 时需要 Docker：

```bash
# Linux
sudo apt install docker.io
sudo systemctl enable docker
sudo systemctl start docker

# macOS
brew install --cask docker

# Windows
winget install Docker.DockerDesktop
```

### Podman 替代方案

```bash
# 使用 Podman 作为容器 CLI
ZEROCLAW_CONTAINER_CLI=podman ./bootstrap.sh --docker
```

## 性能基准

### 本地测量方法

```bash
# 编译发布版本
cargo build --release
ls -lh target/release/zeroclaw

# 测量内存和时间
/usr/bin/time -l target/release/zeroclaw --help
/usr/bin/time -l target/release/zeroclaw status
```

### 示例结果（macOS arm64, 2026-02-18）

| 指标 | 数值 |
|------|------|
| 二进制大小 | 8.8M |
| `--help` 时间 | ~0.02s |
| `--help` 内存 | ~3.9MB |
| `status` 时间 | ~0.01s |
| `status` 内存 | ~4.1MB |

## 与 OpenClaw 对比

| 指标 | OpenClaw | ZeroClaw | 改进 |
|------|----------|----------|------|
| 运行时内存 | >1GB | <5MB | 99.5% ↓ |
| 启动时间 | >500s | <10ms | 99.8% ↓ |
| 二进制大小 | ~28MB | ~8.8MB | 68% ↓ |
| 最低硬件 | Mac Mini $599 | 任意 $10 | 98% ↓ |

> [!note] 注意
> OpenClaw 需要 Node.js 运行时（通常额外 ~390MB 内存开销），而 ZeroClaw 是静态二进制文件。

## 下一步

- [[ZeroClaw-快速开始]] - 安装和初始化
- [[ZeroClaw-架构设计]] - 了解 Trait 驱动架构
- [[ZeroClaw-性能优化]] - 性能调优指南

---

_最后更新：2026-02-21_
