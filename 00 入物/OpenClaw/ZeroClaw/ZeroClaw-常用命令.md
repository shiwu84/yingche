---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, cli, commands, reference, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw CLI, ZeroClaw 命令参考]
---

# ZeroClaw-常用命令

> [!summary] 核心概念
> ZeroClaw 提供完整的 CLI 工具集，涵盖 Agent 交互、服务管理、认证配置、通道绑定、系统诊断等功能。

## Agent 命令

### 基本交互

```bash
# 发送单条消息
zeroclaw agent -m "Hello, ZeroClaw!"

# 交互式模式（REPL）
zeroclaw agent

# 指定模型
zeroclaw agent -m "任务描述" --model "claude-sonnet-4-5-20250929"

# 指定提供者
zeroclaw agent -m "任务" --provider anthropic

# 使用特定认证配置
zeroclaw agent -m "任务" --auth-profile openai-codex:work
```

### 高级选项

```bash
# 附加文件上下文
zeroclaw agent -m "分析这个文件" --file ./data.txt

# 启用详细输出
zeroclaw agent -m "任务" --verbose

# 限制思考时间
zeroclaw agent -m "任务" --timeout 60
```

## 服务管理

### 守护进程

```bash
# 启动守护进程（前台）
zeroclaw daemon

# 启动守护进程（后台）
zeroclaw daemon --background
```

### 网关服务

```bash
# 启动网关（默认 127.0.0.1:3000）
zeroclaw gateway

# 指定端口
zeroclaw gateway --port 8080

# 随机端口（安全）
zeroclaw gateway --port 0

# 绑定到所有接口
zeroclaw gateway --host 0.0.0.0
```

### 系统服务

```bash
# 安装系统服务
zeroclaw service install

# 查看服务状态
zeroclaw service status

# 启动服务
zeroclaw service start

# 停止服务
zeroclaw service stop

# 重启服务
zeroclaw service restart

# 卸载服务
zeroclaw service uninstall

# Alpine (OpenRC) 使用 sudo
sudo zeroclaw service install
```

## 认证管理

```bash
# 登录（OAuth）
zeroclaw auth login --provider openai-codex --profile default

# 设备代码认证
zeroclaw auth login --provider openai-codex --device-code

# 粘贴重定向 URL
zeroclaw auth paste-redirect --provider openai-codex --profile default

# 粘贴令牌（Anthropic）
zeroclaw auth paste-token --provider anthropic --profile default

# 设置令牌
zeroclaw auth setup-token --provider anthropic --profile default

# 查看状态
zeroclaw auth status

# 刷新令牌
zeroclaw auth refresh --provider openai-codex --profile default

# 切换配置
zeroclaw auth use --provider openai-codex --profile work

# 列出所有配置
zeroclaw auth status --list
```

## 通道配置

```bash
# 绑定 Telegram 用户
zeroclaw channel bind-telegram 123456789

# 解除绑定
zeroclaw channel unbind-telegram 123456789

# 查看绑定列表
zeroclaw channel list

# 检查通道健康
zeroclaw channel doctor

# 获取集成信息
zeroclaw integrations info Telegram
zeroclaw integrations info Discord
zeroclaw integrations info Slack
```

## 系统诊断

```bash
# 运行完整诊断
zeroclaw doctor

# 检查状态
zeroclaw status

# 深度状态检查
zeroclaw status --deep

# 查看配置
zeroclaw config show

# 编辑配置
zeroclaw config edit
```

## 迁移工具

```bash
# 从 OpenClaw 迁移（预览）
zeroclaw migrate openclaw --dry-run

# 执行迁移
zeroclaw migrate openclaw

# 迁移特定组件
zeroclaw migrate openclaw --memory-only
zeroclaw migrate openclaw --config-only
```

## Shell 补全

```bash
# Bash（直接 source）
source <(zeroclaw completions bash)

# Bash（保存到文件）
zeroclaw completions bash > ~/.bash_completion

# Zsh
zeroclaw completions zsh > ~/.zfunc/_zeroclaw

# Fish
zeroclaw completions fish > ~/.config/fish/completions/zeroclaw.fish
```

## 开发命令

```bash
# 不安装直接运行
cargo run --release -- status
cargo run --release -- agent -m "hello"

# 构建发布版本
cargo build --release

# 快速构建（多codegen单元）
cargo build --profile release-fast

# 运行测试
cargo test

# 格式化代码
cargo fmt

# 代码检查
cargo clippy
```

## 实用命令组合

### 快速设置

```bash
# 一键安装 + 配置
./bootstrap.sh --onboard --api-key "sk-..." --provider openrouter
```

### 调试模式

```bash
# 启用详细日志
RUST_LOG=debug zeroclaw agent -m "test"

# 查看日志文件
tail -f ~/.zeroclaw/zeroclaw.log
```

### 性能测试

```bash
# 测量启动时间
/usr/bin/time -l zeroclaw status

# 测量内存使用
/usr/bin/time -l zeroclaw --help
```

## 命令速查表

| 命令 | 描述 | 示例 |
|------|------|------|
| `agent` | 与 Agent 交互 | `zeroclaw agent -m "hello"` |
| `daemon` | 启动守护进程 | `zeroclaw daemon` |
| `gateway` | 启动网关服务 | `zeroclaw gateway --port 8080` |
| `service` | 管理系统服务 | `zeroclaw service restart` |
| `auth` | 认证管理 | `zeroclaw auth login` |
| `channel` | 通道配置 | `zeroclaw channel bind-telegram` |
| `doctor` | 系统诊断 | `zeroclaw doctor` |
| `status` | 查看状态 | `zeroclaw status` |
| `migrate` | 数据迁移 | `zeroclaw migrate openclaw` |
| `completions` | Shell 补全 | `zeroclaw completions bash` |

## 环境变量

| 变量 | 描述 | 默认值 |
|------|------|--------|
| `ZEROCLAW_CONFIG` | 配置文件路径 | `~/.zeroclaw/config.toml` |
| `ZEROCLAW_MEMORY` | 记忆存储路径 | `~/.zeroclaw/memory` |
| `ZEROCLAW_LOG` | 日志文件路径 | `~/.zeroclaw/zeroclaw.log` |
| `ZEROCLAW_CONTAINER_CLI` | 容器 CLI | `docker` |
| `RUST_LOG` | 日志级别 | `info` |

## 下一步

- [[ZeroClaw-快速开始]] - 安装指南
- [[ZeroClaw-Agent 模式]] - Agent 详细用法
- [[ZeroClaw-Daemon 模式]] - 守护进程配置

---

_最后更新：2026-02-21_
