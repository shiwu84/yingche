---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, configuration, config, reference, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 完整配置，ZeroClaw config.toml 详解]
---

# ZeroClaw-完整配置指南

> [!summary] 核心概念
> ZeroClaw 使用 `config.toml` 作为主要配置文件，位于 `~/.zeroclaw/config.toml`。本文档提供完整的配置项参考、示例和最佳实践。

## 配置文件位置

| 优先级 | 来源 | 说明 |
|--------|------|------|
| 1 | `ZEROCLAW_WORKSPACE` 环境变量 | 最高优先级 |
| 2 | `~/.zeroclaw/active_workspace.toml` | 持久化标记 |
| 3 | `~/.zeroclaw/config.toml` | 默认位置 |

**验证配置加载：**
```bash
# 查看配置加载路径
zeroclaw status

# 导出配置 Schema
zeroclaw config schema

# 验证配置有效性
zeroclaw doctor
```

---

## 核心配置

### 默认模型配置

```toml
# ~/.zeroclaw/config.toml

# 默认提供者（支持 29+ 内置提供者）
default_provider = "openrouter"

# 默认模型
default_model = "anthropic/claude-sonnet-4-6"

# 模型温度（0.0-2.0）
default_temperature = 0.7
```

**常用提供者 ID：**
| 提供者 | ID | 需要 API Key |
|--------|-----|--------------|
| OpenRouter | `openrouter` | ✅ |
| Anthropic | `anthropic` | ✅ |
| OpenAI | `openai` | ✅ |
| Ollama | `ollama` | ❌ |
| Groq | `groq` | ✅ |
| DeepSeek | `deepseek` | ✅ |
| Google Gemini | `gemini` | ✅ |

**环境变量覆盖：**
```bash
# 环境变量优先级高于配置文件
export ZEROCLAW_PROVIDER="anthropic"
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## Agent 配置

### 基础 Agent 配置

```toml
[agent]
# 紧凑上下文模式（适合小模型）
compact_context = false

# 最大工具调用迭代次数
max_tool_iterations = 10

# 最大历史消息数
max_history_messages = 50

# 并行工具执行
parallel_tools = false

# 工具分发策略
tool_dispatcher = "auto"
```

### 子代理配置

```toml
# 研究员子代理
[agents.researcher]
provider = "openrouter"
model = "anthropic/claude-sonnet-4-6"
system_prompt = "你是一个专业的研究助手，擅长信息收集和分析。"
max_depth = 2
agentic = true
allowed_tools = ["web_search", "http_request", "file_read"]
max_iterations = 8

# 编程子代理
[agents.coder]
provider = "ollama"
model = "qwen2.5-coder:32b"
temperature = 0.2
system_prompt = "你是一个专业的软件工程师，擅长代码编写和审查。"
agentic = true
allowed_tools = ["shell", "file_read", "file_write"]
max_iterations = 15
```

---

## 工具配置

### Shell 工具（bash 命令执行）

```toml
[autonomy]
# 自主级别：readonly | supervised | full
level = "supervised"

# 限制在工作目录
workspace_only = true

# 允许的命令白名单
allowed_commands = [
    "ls", "cat", "grep", "find", "head", "tail",
    "git", "npm", "cargo", "curl", "wget",
    "pwd", "wc", "sort", "uniq", "awk", "sed"
]

# 禁止的命令
forbidden_commands = ["rm", "sudo", "chmod", "chown", "dd", "mkfs"]

# 禁止访问的路径
forbidden_paths = [
    "/etc", "/root", "/proc", "/sys", "/boot",
    "~/.ssh", "~/.gnupg", "~/.aws", "~/.config"
]

# 每小时最大操作数
max_actions_per_hour = 100

# 高风险命令需要审批
require_approval_for_medium_risk = true
block_high_risk_commands = true

# 自动批准的操作
auto_approve = ["ls", "cat", "pwd", "git status"]

# 总是需要询问的操作
always_ask = ["git push", "npm publish", "cargo publish"]
```

### HTTP 请求工具

```toml
[tools.http_request]
# 启用 HTTP 请求工具
enabled = true

# 允许的域名（空数组 = 全部禁止）
allowed_domains = [
    "api.github.com",
    "api.anthropic.com",
    "api.openai.com",
    "api.example.com"
]

# 最大响应大小（字节）
max_response_size = 1000000  # 1MB

# 请求超时（秒）
timeout_secs = 30
```

### 网络搜索工具

```toml
[tools.web_search]
# 启用网络搜索
enabled = true

# 搜索提供者
provider = "brave"  # brave | duckduckgo | google | tavily | exa

# API Key（部分提供者需要）
api_key = "YOUR_BRAVE_API_KEY"

# 最大结果数
max_results = 10

# 时间范围
freshness = "pw"  # pd=今天，pw=本周，pm=本月，py=今年

# 搜索语言
search_lang = "zh-CN"

# 搜索结果区域
country = "CN"
```

**各搜索提供者对比：**

| 提供者 | API Key | 免费额度 | 特点 |
|--------|---------|----------|------|
| Brave | 需要 | 2000 次/月 | 高质量结果 |
| DuckDuckGo | 不需要 | 无限 | 隐私保护 |
| Google | 需要 | 100 次/天 | 最全面 |
| Tavily | 需要 | 1000 次/月 | AI 优化 |
| Exa | 需要 | 1000 次/月 | 技术搜索 |

### 网页抓取工具

```toml
[tools.web_fetch]
# 启用网页抓取
enabled = true

# 最大字符数
max_chars = 10000

# 超时时间（秒）
timeout_secs = 30

# 提取模式
extract_mode = "markdown"  # markdown | text

# 允许的域名
allowed_domains = [
    "docs.rs",
    "github.com",
    "wiki.archlinux.org",
    "stackoverflow.com"
]
```

### 浏览器工具

```toml
[browser]
# 启用浏览器工具
enabled = false  # 按需开启

# 允许的域名
allowed_domains = [
    "docs.rs",
    "github.com",
    "wiki.archlinux.org",
    "rust-lang.org"
]

# 浏览器后端
backend = "agent_browser"  # agent_browser | rust_native | computer_use | auto

# Rust 原生后端配置
native_headless = true
native_webdriver_url = "http://127.0.0.1:9515"
native_chrome_path = "/usr/bin/google-chrome"  # 可选

# Computer Use 侧车配置
[browser.computer_use]
endpoint = "http://127.0.0.1:8787/v1/actions"
api_key = ""  # 可选
timeout_ms = 15000
allow_remote_endpoint = false  # 安全：禁止远程端点
```

### Composio 集成（1000+ OAuth 应用）

```toml
[composio]
# 启用 Composio
enabled = false

# Composio API Key
api_key = "YOUR_COMPOSIO_API_KEY"

# 默认用户 ID
entity_id = "default"
```

**使用流程：**
1. 设置 `enabled = true` 并配置 `api_key`
2. 运行 `zeroclaw agent -m "连接 GitHub"`
3. 在浏览器完成 OAuth 授权
4. 使用 `execute` 工具调用应用功能

---

## 内存系统配置

### SQLite 后端（默认）

```toml
[memory]
# 内存后端类型
backend = "sqlite"  # sqlite | lucid | markdown | none

# 自动保存（仅保存用户输入，不保存助手响应）
auto_save = true

# 嵌入提供者
embedding_provider = "none"  # none | openai | custom:https://...

# 嵌入模型
embedding_model = "text-embedding-3-small"

# 嵌入维度
embedding_dimensions = 1536

# 混合搜索权重
vector_weight = 0.7  # 向量搜索权重
keyword_weight = 0.3  # 关键词搜索权重

# SQLite 打开超时（秒）
sqlite_open_timeout_secs = 30
```

### 自定义嵌入路由

```toml
[memory]
embedding_model = "hint:semantic"

[[embedding_routes]]
hint = "semantic"
provider = "openai"
model = "text-embedding-3-small"
dimensions = 1536

[[embedding_routes]]
hint = "archive"
provider = "openai"
model = "text-embedding-3-large"
dimensions = 3072
```

---

## 网关配置

### 基础网关

```toml
[gateway]
# 绑定地址（默认仅本地）
host = "127.0.0.1"

# 监听端口
port = 3000

# 需要配对码（首次连接）
require_pairing = true

# 禁止公开绑定（安全）
allow_public_bind = false
```

### 隧道配置

```toml
[tunnel]
# 隧道提供者
provider = "none"  # none | cloudflare | tailscale | ngrok | custom

# Cloudflare Tunnel
# provider = "cloudflare"
# cloudflare_tunnel_id = "your-tunnel-id"

# Tailscale
# provider = "tailscale"
# tailscale_hostname = "zeroclaw"

# ngrok
# provider = "ngrok"
# ngrok_auth_token = "your-ngrok-token"
```

---

## 通道配置

### Telegram 配置

```toml
[channels_config.telegram]
# 启用 Telegram
enabled = true

# Bot Token（从 @BotFather 获取）
bot_token = "123456789:ABCdefGHIjklMNOpqrsTUVwxyz"

# 允许的用户 ID（空数组 = 全部禁止）
allowed_users = ["123456789", "987654321"]

# 消息超时（秒）
message_timeout_secs = 300

# 新消息中断处理
interrupt_on_new_message = false
```

### Discord 配置

```toml
[channels_config.discord]
# 启用 Discord
enabled = true

# Bot Token
bot_token = "MTIzNDU2Nzg5MDEyMzQ1Njc4OQ.ABCDEF.GHIjklMNOpqrsTUVwxyz123456"

# 允许的频道 ID
allowed_channels = ["123456789012345678"]

# 允许的服务器 ID
allowed_guilds = ["987654321098765432"]

# 命令前缀
prefix = "!"

# 消息超时
message_timeout_secs = 300
```

### WhatsApp 配置（Cloud API）

```toml
[channels_config.whatsapp]
# 启用 WhatsApp
enabled = true

# Meta Cloud API 访问令牌
access_token = "EAABsbCS1iHgBO..."

# 电话号码 ID
phone_number_id = "123456789012345"

# Webhook 验证令牌
verify_token = "your-verify-token"

# 应用密钥（用于签名验证）
app_secret = "1234567890abcdef"

# 允许的号码（"*" = 全部）
allowed_numbers = ["+861234567890"]
```

### WhatsApp 配置（Web 模式）

```toml
[channels_config.whatsapp]
# WhatsApp Web 模式（需要 build flag）
session_path = "~/.zeroclaw/whatsapp-session"
pair_phone = "861234567890"  # 可选，配对手机号
pair_code = "ABC123"  # 可选，自定义配对码
allowed_numbers = ["+861234567890"]
```

---

## 运行时配置

### 原生运行时

```toml
[runtime]
# 运行时类型
kind = "native"  # native | docker

# 全局推理开关（部分提供者支持）
reasoning_enabled = false
```

### Docker 沙箱运行时

```toml
[runtime]
kind = "docker"

# 容器镜像
image = "zeroclaw/sandbox:latest"

# 网络模式
network = "none"  # none | bridge | host

# 资源限制
memory_limit_mb = 512
cpu_limit = 1.0

# 只读根文件系统
read_only_rootfs = true

# 挂载工作目录
mount_workspace = true

# 临时文件系统
tmpfs = ["/tmp", "/var/tmp"]
```

---

## 计划任务配置

### Cron 配置

```toml
[cron]
# 启用计划任务
enabled = true

# 执行超时（分钟）
exec_timeout_minutes = 5

# 时区
timezone = "Asia/Shanghai"
```

### 添加计划任务（通过 CLI）

```bash
# 每日新闻推送（每天 9:00）
zeroclaw cron add "0 9 * * *" --tz Asia/Shanghai \
  "获取今日技术新闻并推送给用户"

# 系统健康检查（每天 3:00）
zeroclaw cron add "0 3 * * *" \
  "检查系统状态：CPU、内存、磁盘、服务状态"

# 每 30 分钟检查一次
zeroclaw cron add-every 1800000 \
  "检查邮箱和日历"

# 一次性任务（10 分钟后）
zeroclaw cron once 600 \
  "提醒我开会"
```

---

## 心跳任务配置

```toml
[heartbeat]
# 启用心跳任务
enabled = true

# 检查间隔（分钟）
interval_minutes = 30

# 心跳文件路径
heartbeat_file = "HEARTBEAT.md"
```

**HEARTBEAT.md 示例：**
```markdown
# HEARTBEAT.md

- [ ] 检查未读邮件
- [ ] 查看日历（未来 24 小时）
- [ ] 检查天气（如需外出）
- [ ] 查看重要通知
```

---

## 技能配置

```toml
[skills]
# 启用开放技能库
open_skills_enabled = false

# 技能库目录
open_skills_dir = "~/open-skills"  # 可选

# 提示注入模式
prompt_injection_mode = "full"  # full | compact
```

**环境变量覆盖：**
```bash
export ZEROCLAW_OPEN_SKILLS_ENABLED=true
export ZEROCLAW_OPEN_SKILLS_DIR=~/my-skills
export ZEROCLAW_SKILLS_PROMPT_MODE=compact
```

---

## 模型路由配置

### 查询分类路由

```toml
# 模型路由
[[model_routes]]
hint = "reasoning"
provider = "openrouter"
model = "anthropic/claude-3-opus"

[[model_routes]]
hint = "fast"
provider = "openrouter"
model = "meta-llama/llama-3-70b"

[[model_routes]]
hint = "code"
provider = "ollama"
model = "qwen2.5-coder:32b"

# 查询分类规则
[query_classification]
enabled = true

[[query_classification.rules]]
hint = "reasoning"
keywords = ["解释", "分析", "为什么", "详细说明"]
min_length = 200
priority = 10

[[query_classification.rules]]
hint = "fast"
keywords = ["你好", "谢谢", "再见"]
max_length = 50
priority = 5

[[query_classification.rules]]
hint = "code"
patterns = ["```", "fn ", "pub ", "impl "]
priority = 8
```

---

## 成本追踪配置

```toml
[cost]
# 启用成本追踪
enabled = true

# 每日限额（美元）
daily_limit_usd = 10.00

# 每月限额（美元）
monthly_limit_usd = 100.00

# 警告阈值（百分比）
warn_at_percent = 80

# 允许超额（需要 --override 标志）
allow_override = false
```

---

## 多模态配置

```toml
[multimodal]
# 最大图片数
max_images = 4

# 单张图片大小限制（MB）
max_image_size_mb = 5

# 允许远程获取图片 URL
allow_remote_fetch = false
```

---

## 身份配置

### OpenClaw 格式（默认）

```toml
[identity]
format = "openclaw"
# 使用 IDENTITY.md 等 markdown 文件
```

### AIEOS 格式

```toml
[identity]
format = "aieos"

# 从文件加载
aieos_path = "identity.json"

# 或内联配置
# aieos_inline = '{"identity":{"names":{"first":"Nova"}}}'
```

---

## 可观测性配置

```toml
[observability]
# 后端类型
backend = "none"  # none | log | prometheus | otel

# OpenTelemetry 配置（当 backend = "otel"）
otel_endpoint = "http://localhost:4318"
otel_service_name = "zeroclaw"
```

---

## 硬件和外设配置

### 硬件访问

```toml
[hardware]
# 启用硬件访问
enabled = false

# 传输模式
transport = "none"  # none | native | serial | probe

# 串口配置
serial_port = "/dev/ttyACM0"
baud_rate = 115200

# 调试探头目标
probe_target = "STM32F401RE"

# 工作目录数据手册 RAG
workspace_datasheets = false
```

### 外设配置

```toml
[peripherals]
# 启用外设支持
enabled = false

# 数据手册目录
datasheet_dir = "docs/datasheets"

# 开发板配置
[[peripherals.boards]]
board = "nucleo-f401re"
transport = "serial"
path = "/dev/ttyACM0"
baud = 115200

[[peripherals.boards]]
board = "rpi-gpio"
transport = "native"

[[peripherals.boards]]
board = "esp32"
transport = "serial"
path = "/dev/ttyUSB0"
baud = 921600
```

---

## 安全配置

### 密钥加密

```toml
[secrets]
# 加密存储 API 密钥
encrypt = true

# 密钥文件路径（默认）
# key_file = "~/.zeroclaw/.secret_key"
```

### 自主级别说明

| 级别 | 描述 | 适用场景 |
|------|------|----------|
| `readonly` | 只读操作 | 审计、监控 |
| `supervised` | 中等风险需审批 | 日常使用（推荐） |
| `full` | 完全自主 | 可信环境、自动化 |

---

## 完整配置示例

```toml
# ~/.zeroclaw/config.toml
# ZeroClaw 完整配置示例

# ========== 核心配置 ==========
default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4-6"
default_temperature = 0.7

# ========== Agent 配置 ==========
[agent]
compact_context = false
max_tool_iterations = 10
max_history_messages = 50
parallel_tools = false
tool_dispatcher = "auto"

# ========== 自主和安全配置 ==========
[autonomy]
level = "supervised"
workspace_only = true
allowed_commands = ["ls", "cat", "grep", "find", "git", "curl"]
forbidden_paths = ["/etc", "/root", "/proc", "/sys", "~/.ssh"]
max_actions_per_hour = 100
require_approval_for_medium_risk = true
block_high_risk_commands = true

# ========== 工具配置 ==========
[tools.http_request]
enabled = true
allowed_domains = ["api.github.com", "api.anthropic.com"]
max_response_size = 1000000
timeout_secs = 30

[tools.web_search]
enabled = true
provider = "brave"
api_key = "YOUR_BRAVE_API_KEY"
max_results = 10
freshness = "pw"

[tools.web_fetch]
enabled = true
max_chars = 10000
timeout_secs = 30
allowed_domains = ["docs.rs", "github.com", "wiki.archlinux.org"]

# ========== 内存配置 ==========
[memory]
backend = "sqlite"
auto_save = true
embedding_provider = "none"
vector_weight = 0.7
keyword_weight = 0.3

# ========== 网关配置 ==========
[gateway]
host = "127.0.0.1"
port = 3000
require_pairing = true
allow_public_bind = false

# ========== Telegram 通道 ==========
[channels_config.telegram]
enabled = true
bot_token = "YOUR_BOT_TOKEN"
allowed_users = ["123456789"]
message_timeout_secs = 300

# ========== 运行时配置 ==========
[runtime]
kind = "native"
reasoning_enabled = false

# ========== 成本追踪 ==========
[cost]
enabled = true
daily_limit_usd = 10.00
monthly_limit_usd = 100.00
warn_at_percent = 80

# ========== 技能配置 ==========
[skills]
open_skills_enabled = false
prompt_injection_mode = "full"

# ========== 心跳任务 ==========
[heartbeat]
enabled = true
interval_minutes = 30
```

---

## 配置验证命令

```bash
# 查看当前配置
zeroclaw status

# 运行诊断
zeroclaw doctor

# 检查通道健康
zeroclaw channel doctor

# 检查认证状态
zeroclaw auth status

# 导出配置 Schema
zeroclaw config schema

# 验证配置语法
cat ~/.zeroclaw/config.toml | python3 -c "import toml, sys; toml.load(sys.stdin)"
```

---

## 环境变量参考

| 变量 | 描述 | 示例 |
|------|------|------|
| `ZEROCLAW_WORKSPACE` | 工作目录路径 | `/home/user/zeroclaw-ws` |
| `ZEROCLAW_PROVIDER` | 提供者覆盖 | `anthropic` |
| `ZEROCLAW_CONFIG` | 配置文件路径 | `~/.zeroclaw/prod.toml` |
| `ZEROCLAW_OPEN_SKILLS_ENABLED` | 开放技能开关 | `true` |
| `ZEROCLAW_OPEN_SKILLS_DIR` | 技能目录 | `~/my-skills` |
| `ZEROCLAW_SKILLS_PROMPT_MODE` | 技能提示模式 | `compact` |
| `ANTHROPIC_API_KEY` | Anthropic API Key | `sk-ant-...` |
| `OPENAI_API_KEY` | OpenAI API Key | `sk-...` |
| `RUST_LOG` | 日志级别 | `debug` |

---

## 下一步

- [[ZeroClaw-常用命令]] - CLI 命令速查
- [[ZeroClaw-认证机制]] - 认证配置详解
- [[ZeroClaw-沙箱运行时]] - Docker 沙箱配置
- [[ZeroClaw-故障排除]] - 常见问题解决

---

_最后更新：2026-02-21_
