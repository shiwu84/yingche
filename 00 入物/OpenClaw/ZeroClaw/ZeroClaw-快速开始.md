---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, installation, quickstart, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 安装，ZeroClaw 快速入门]
---

# ZeroClaw-快速开始

> [!summary] 核心概念
> ZeroClaw 提供多种安装方式，包括一键安装脚本、包管理器和预编译二进制文件。

## 安装方式

### 方式 1：Homebrew（macOS/Linuxbrew）⭐ 推荐

```bash
brew install zeroclaw
```

### 方式 2：一键安装脚本

```bash
# 克隆仓库并运行本地 bootstrap 脚本
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
./bootstrap.sh

# 在全新机器上安装系统依赖和 Rust
./bootstrap.sh --install-system-deps --install-rust

# 使用预编译二进制（推荐低内存/低磁盘主机）
./bootstrap.sh --prefer-prebuilt

# 仅使用二进制安装（无源码构建回退）
./bootstrap.sh --prebuilt-only

# 远程一键安装（先审查脚本）
curl -fsSL https://raw.githubusercontent.com/zeroclaw-labs/zeroclaw/main/scripts/bootstrap.sh | bash
```

### 方式 3：预编译二进制

```bash
# ARM64 Linux 示例
curl -fsSLO https://github.com/zeroclaw-labs/zeroclaw/releases/latest/download/zeroclaw-aarch64-unknown-linux-gnu.tar.gz
tar xzf zeroclaw-aarch64-unknown-linux-gnu.tar.gz
install -m 0755 zeroclaw "$HOME/.cargo/bin/zeroclaw"
```

**支持的平台：**
- Linux: x86_64, aarch64, armv7
- macOS: x86_64, aarch64
- Windows: x86_64

### 方式 4：源码编译

```bash
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw
cargo build --release --locked
cargo install --path . --force --locked

# 确保 ~/.cargo/bin 在 PATH 中
export PATH="$HOME/.cargo/bin:$PATH"
```

## 初始化配置

### 快速配置（无提示）

```bash
zeroclaw onboard --api-key sk-... --provider openrouter --model "openrouter/auto"
```

### 交互式配置向导

```bash
zeroclaw onboard --interactive
```

### 强制覆盖已有配置

```bash
zeroclaw onboard --force
```

### 仅修复通道/允许列表

```bash
zeroclaw onboard --channels-only
```

## 验证安装

```bash
# 检查状态
zeroclaw status

# 检查认证状态
zeroclaw auth status

# 运行系统诊断
zeroclaw doctor

# 检查通道健康
zeroclaw channel doctor
```

## 基本使用

### 命令行交互

```bash
# 发送消息给 Agent
zeroclaw agent -m "Hello, ZeroClaw!"

# 交互式模式
zeroclaw agent
```

### 启动网关

```bash
# 默认端口 127.0.0.1:3000
zeroclaw gateway

# 随机端口（安全加固）
zeroclaw gateway --port 0
```

### 启动守护进程

```bash
# 完整自主运行时
zeroclaw daemon
```

### 管理服务

```bash
# 安装后台服务
zeroclaw service install

# 查看服务状态
zeroclaw service status

# 重启服务
zeroclaw service restart

# Alpine (OpenRC) 上使用 sudo
sudo zeroclaw service install
```

## 通道配置

### 绑定 Telegram

```bash
# 将 Telegram ID 绑定到允许列表
zeroclaw channel bind-telegram 123456789

# 获取 Telegram 集成详情
zeroclaw integrations info Telegram
```

> [!note] 注意
> 通道（Telegram、Discord、Slack）需要守护进程运行才能工作。

## 生成 Shell 补全

```bash
# Bash
source <(zeroclaw completions bash)

# Zsh
zeroclaw completions zsh > ~/.zfunc/_zeroclaw
```

## 从 OpenClaw 迁移

```bash
# 先预览迁移（安全）
zeroclaw migrate openclaw --dry-run

# 执行迁移
zeroclaw migrate openclaw
```

## 开发模式

无需全局安装，使用 cargo 直接运行：

```bash
cargo run --release -- status
cargo run --release -- agent -m "Hello"
```

## 下一步

- [[ZeroClaw-系统要求]] - 了解编译和运行环境要求
- [[ZeroClaw-认证机制]] - 配置 API 密钥和 OAuth
- [[ZeroClaw-常用命令]] - CLI 命令速查表

---

_最后更新：2026-02-21_
