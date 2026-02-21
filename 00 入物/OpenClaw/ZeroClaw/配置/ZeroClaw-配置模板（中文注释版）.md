---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, configuration, template, chinese-comments]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 配置模板，ZeroClaw config.toml 中文注释版]
---

# ZeroClaw-配置模板（完整中文注释版）

> [!summary] 本文档
> ZeroClaw 完整配置模板，所有配置项均带有详细的中文注释，可直接复制使用。

---

## 完整配置模板

```toml
# ~/.zeroclaw/config.toml
# ================================================================================
# ZeroClaw 完整配置文件（中文注释版）
# 最后更新：2026-02-21
# ================================================================================

# ================================================================================
# 一、核心配置
# ================================================================================

# --------------------------------
# 默认模型配置
# --------------------------------

# 默认提供者 ID
# 可选值：
#   - openrouter: 模型聚合平台（推荐，支持多个提供者）
#   - anthropic: Anthropic Claude 系列
#   - openai: OpenAI GPT 系列
#   - ollama: 本地运行开源模型
#   - groq: 高速推理（LPU）
#   - deepseek: 高性价比
#   - gemini: Google Gemini 系列
#   - 完整列表见"核心配置详解"笔记
default_provider = "openrouter"

# 默认模型 ID
# 格式说明：
#   - 云模型：提供者/模型名 (如：anthropic/claude-sonnet-4-6)
#   - 本地模型：模型名 (如：qwen2.5-coder:32b)
# 常用模型推荐：
#   - 高质量：anthropic/claude-sonnet-4-6, openai/gpt-4o
#   - 性价比：deepseek/deepseek-chat
#   - 本地：qwen2.5-coder:32b, llama-3-70b
default_model = "anthropic/claude-sonnet-4-6"

# 模型温度（0.0 - 2.0）
# 控制输出随机性：
#   - 0.0-0.3: 非常确定，适合代码生成、事实查询
#   - 0.4-0.7: 平衡，适合日常对话（推荐）
#   - 0.8-1.2: 有创意，适合写作、头脑风暴
#   - 1.3-2.0: 高度创意，实验性使用
default_temperature = 0.7


# ================================================================================
# 二、Agent 配置
# ================================================================================

[agent]

# 紧凑上下文模式
# true: 减少上下文大小，适合 13B 以下小模型或 Token 受限场景
# false: 完整上下文，适合云模型（推荐）
compact_context = false

# 最大工具调用迭代次数
# 每次请求 Agent 可以调用工具的最大轮数
# 建议：简单任务 3-5，一般任务 10，复杂任务 15-20
max_tool_iterations = 10

# 最大历史消息数
# 保留的对话历史消息数量
# 值越大上下文越完整，但消耗更多 Token
max_history_messages = 50

# 并行工具执行
# true: 独立的工具调用并发执行（更快）
# false: 工具调用串行执行（便于调试）
parallel_tools = false

# 工具分发策略
# auto: 自动选择最佳工具（推荐）
# round_robin: 轮询分发
# load_balance: 负载均衡
tool_dispatcher = "auto"


# ================================================================================
# 三、子代理配置
# ================================================================================

# --------------------------------
# 研究员子代理（信息收集）
# --------------------------------
[agents.researcher]

# 提供者
provider = "openrouter"

# 模型
model = "anthropic/claude-sonnet-4-6"

# 系统提示（定义子代理的角色和职责）
system_prompt = """
你是一个专业的研究助手。你的职责：
1. 使用网络搜索收集信息
2. 验证信息来源的可靠性
3. 整理和总结关键发现
4. 提供引用链接

回答要求：
- 事实准确，注明来源
- 区分事实和观点
- 提供多个视角
"""

# 最大递归深度（子代理可以再委托给其他子代理的层数）
max_depth = 2

# 启用多轮工具调用模式
# true: 支持工具调用循环（可以多次调用工具）
# false: 仅执行一次 prompt→response
agentic = true

# 工具白名单（仅允许使用这些工具）
allowed_tools = ["web_search", "web_fetch", "http_request", "file_read", "file_write"]

# 子代理的最大工具迭代次数
max_iterations = 12

# --------------------------------
# 程序员子代理（代码编写）
# --------------------------------
[agents.coder]

# 使用本地模型（节省成本）
provider = "ollama"
model = "qwen2.5-coder:32b"

# 代码生成使用较低温度
temperature = 0.2

system_prompt = """
你是一个资深的软件工程师。你的职责：
1. 编写高质量、可维护的代码
2. 遵循最佳实践和设计模式
3. 进行代码审查和安全检查
4. 提供清晰的注释和文档

代码风格：
- Rust: 遵循 Rust 风格指南
- Python: 遵循 PEP 8
- TypeScript: 使用严格模式
"""

agentic = true
allowed_tools = ["shell", "file_read", "file_write", "git", "file_list"]
max_iterations = 20
max_depth = 2


# ================================================================================
# 四、自主级别与安全配置
# ================================================================================

[autonomy]

# 自主级别（重要安全配置）
# readonly   - 只读模式：仅允许读取类工具，禁止所有写操作
# supervised - 监督模式（推荐）：允许所有工具，中等风险操作需审批
# full       - 完全自主：允许所有工具，仅高风险操作被禁止
level = "supervised"

# 限制在工作目录内执行命令
# true: 命令只能在工作目录内执行（推荐）
# false: 可以在任意目录执行（不安全）
workspace_only = true

# 允许的命令白名单
# 仅这些命令可以被执行
allowed_commands = [
    # 文件操作
    "ls", "cat", "head", "tail", "less", "more",
    "cp", "mv", "mkdir", "rm", "touch",
    
    # 文本处理
    "grep", "awk", "sed", "cut", "sort", "uniq",
    "wc", "diff", "patch",
    
    # 版本控制
    "git", "gh",
    
    # 包管理
    "npm", "npx", "yarn", "pnpm",
    "cargo", "rustc",
    "pip", "pip3", "poetry",
    
    # 网络
    "curl", "wget", "ssh", "scp",
    
    # 压缩
    "tar", "gzip", "zip", "unzip",
    
    # 系统信息
    "pwd", "whoami", "uname", "hostname",
    "ps", "top", "free", "df", "du"
]

# 禁止的命令黑名单
# 这些命令永远不能被执行
forbidden_commands = [
    "rm", "sudo", "chmod", "chown", "dd", "mkfs",
    "shutdown", "reboot", "kill", "pkill",
    "iptables", "firewall-cmd"
]

# 禁止访问的路径
# 这些路径不能被访问（即使是白名单命令）
forbidden_paths = [
    "/etc",                    # 系统配置
    "/root",                   # root 用户目录
    "/proc", "/sys",           # 虚拟文件系统
    "/boot",                   # 引导分区
    "~/.ssh",                  # SSH 密钥
    "~/.gnupg",                # GPG 密钥
    "~/.aws",                  # AWS 凭证
    "~/.config",               # 用户配置
    "~/.local"                 # 本地数据
]

# 每小时最大操作数
max_actions_per_hour = 100

# 每日最大操作数
max_actions_per_day = 1000

# 中等风险命令需要审批
require_approval_for_medium_risk = true

# 禁止高风险命令
block_high_risk_commands = true

# 自动批准的操作（即使在 supervised 模式也无需审批）
auto_approve = [
    "ls", "cat", "pwd", "whoami",
    "git status", "git log", "git diff",
    "cargo check", "cargo fmt", "cargo clippy"
]

# 总是需要询问的操作
always_ask = [
    "git push", "git force-push",
    "npm publish", "cargo publish",
    "rm -rf", "git clean -fdx"
]


# ================================================================================
# 五、工具配置
# ================================================================================

# --------------------------------
# HTTP 请求工具
# --------------------------------
[tools.http_request]

# 启用 HTTP 请求工具
enabled = true

# 允许的域名（空数组 = 全部禁止）
# 支持精确匹配和子域名匹配
allowed_domains = [
    "api.github.com",
    "api.anthropic.com",
    "api.openai.com",
    "api.example.com"
]

# 禁止的域名
forbidden_domains = [
    "malicious-site.com"
]

# 最大响应大小（字节）
# 1MB = 1048576 字节
max_response_size = 1048576

# 请求超时（秒）
timeout_secs = 30

# 允许的 HTTP 方法
allowed_methods = ["GET", "POST"]

# 重试配置
max_retries = 3
retry_delay_secs = 2
retry_on_status = [429, 500, 502, 503, 504]

# --------------------------------
# 网络搜索工具
# --------------------------------
[tools.web_search]

# 启用网络搜索
enabled = true

# 搜索提供者
# brave       - Brave Search（高质量，2000 次/月免费）
# duckduckgo  - DuckDuckGo（无需 API Key，功能有限）
# google      - Google Custom Search（最全面，100 次/天免费）
# tavily      - Tavily（AI 优化，1000 次/月免费）
# exa         - Exa（语义搜索，1000 次/月免费）
provider = "brave"

# API Key（根据提供者填写）
api_key = "YOUR_BRAVE_API_KEY"

# 最大结果数
max_results = 10

# 时间范围
# pd - 今天，pw - 本周，pm - 本月，py - 今年
freshness = "pw"

# 搜索语言
search_lang = "zh-CN"

# 搜索结果区域
country = "CN"

# 安全搜索（过滤成人内容）
safe_search = true

# --------------------------------
# 网页抓取工具
# --------------------------------
[tools.web_fetch]

# 启用网页抓取
enabled = true

# 最大字符数
max_chars = 10000

# 超时时间（秒）
timeout_secs = 30

# 提取模式
# markdown - 提取为 Markdown 格式（推荐）
# text     - 提取为纯文本
extract_mode = "markdown"

# 允许的域名
allowed_domains = [
    "docs.rs",
    "github.com",
    "wiki.archlinux.org",
    "stackoverflow.com"
]

# --------------------------------
# 浏览器工具
# --------------------------------
[browser]

# 启用浏览器工具
# 注意：浏览器工具可能执行任意网页操作，建议按需开启
enabled = false

# 允许的域名
allowed_domains = [
    "docs.rs",
    "github.com",
    "wiki.archlinux.org"
]

# 浏览器后端
# agent_browser - Agent 浏览器（默认）
# rust_native   - Rust 原生后端
# computer_use  - Computer Use 侧车
# auto          - 自动选择
backend = "agent_browser"


# ================================================================================
# 六、内存系统配置
# ================================================================================

[memory]

# 内存后端类型
# sqlite     - SQLite 数据库（默认推荐）
# lucid      - Lucid 外部服务
# postgres   - PostgreSQL 数据库
# markdown   - Markdown 文件（人工可读）
# none       - 无持久化（临时会话）
backend = "sqlite"

# 自动保存
# true: 自动保存用户输入（不保存助手响应，防止幻觉污染）
# false: 不自动保存
auto_save = true

# 嵌入提供者
# none   - 不使用嵌入（仅关键词搜索）
# openai - OpenAI 嵌入 API
# custom - 自定义端点
embedding_provider = "none"

# 嵌入模型（当 embedding_provider 不为 none 时）
embedding_model = "text-embedding-3-small"

# 嵌入维度
embedding_dimensions = 1536

# 混合搜索权重
# vector_weight + keyword_weight = 1.0
# vector_weight 高：语义搜索更强
# keyword_weight 高：关键词匹配更强
vector_weight = 0.7
keyword_weight = 0.3

# SQLite 打开超时（秒）
sqlite_open_timeout_secs = 30


# ================================================================================
# 七、网关配置
# ================================================================================

[gateway]

# 绑定地址
# 127.0.0.1 - 仅本地访问（推荐，安全）
# 0.0.0.0   - 所有接口（需要防火墙，不安全）
host = "127.0.0.1"

# 监听端口
port = 3000

# 需要配对码
# true: 首次连接需要配对码（推荐）
# false: 无需配对（不安全）
require_pairing = true

# 禁止公开绑定
# true: 拒绝绑定到公网接口（推荐）
# false: 允许公网绑定（需要隧道）
allow_public_bind = false


# ================================================================================
# 八、通道配置
# ================================================================================

[channels_config]

# 全局消息超时（秒）
# 云 API 建议 60-120，本地模型建议 300-600
message_timeout_secs = 300

# --------------------------------
# Telegram 配置
# --------------------------------
[channels_config.telegram]

# 启用 Telegram
enabled = false

# Bot Token（从 @BotFather 获取）
bot_token = "YOUR_BOT_TOKEN"

# 允许的用户 ID（空数组 = 全部禁止）
# 获取方法：发送消息给 @userinfobot
allowed_users = ["123456789"]

# 允许的群组 ID（可选）
allowed_groups = []

# 新消息中断（取消进行中的请求）
interrupt_on_new_message = false

# --------------------------------
# Discord 配置
# --------------------------------
[channels_config.discord]

# 启用 Discord
enabled = false

# Bot Token（从 Discord Developer Portal 获取）
bot_token = "YOUR_BOT_TOKEN"

# 允许的频道 ID
allowed_channels = ["123456789012345678"]

# 允许的服务器 ID
allowed_guilds = ["987654321098765432"]

# 命令前缀
prefix = "!"

# --------------------------------
# WhatsApp 配置（Cloud API）
# --------------------------------
[channels_config.whatsapp]

# 启用 WhatsApp
enabled = false

# Meta Cloud API 访问令牌
access_token = "EAABsbCS1iHgBO..."

# 电话号码 ID
phone_number_id = "123456789012345"

# Webhook 验证令牌
verify_token = "your-verify-token"

# 应用密钥（用于签名验证）
app_secret = "1234567890abcdef"

# 允许的号码
allowed_numbers = ["+861234567890"]


# ================================================================================
# 九、运行时配置
# ================================================================================

[runtime]

# 运行时类型
# native - 本地执行（默认，性能好）
# docker - Docker 沙箱（安全，有开销）
kind = "native"

# 全局推理开关（部分提供者支持）
# true: 启用推理/思考模式
# false: 禁用推理模式
# unset: 使用提供者默认
reasoning_enabled = false

# --------------------------------
# Docker 沙箱配置（当 kind = "docker" 时）
# --------------------------------
# [runtime.docker]
# image = "zeroclaw/sandbox:latest"
# network = "none"  # none=禁用网络，bridge=桥接模式
# memory_limit_mb = 512
# cpu_limit = 1.0
# read_only_rootfs = true


# ================================================================================
# 十、成本追踪配置
# ================================================================================

[cost]

# 启用成本追踪
enabled = true

# 每日限额（美元）
daily_limit_usd = 10.00

# 每月限额（美元）
monthly_limit_usd = 100.00

# 警告阈值（百分比）
# 达到此百分比时发出警告
warn_at_percent = 80

# 允许超额
# true: 可以使用 --override 标志超额
# false: 严格限制
allow_override = false


# ================================================================================
# 十一、技能配置
# ================================================================================

[skills]

# 启用开放技能库
# true: 克隆并加载 open-skills 仓库
# false: 不加载外部技能（推荐，安全）
open_skills_enabled = false

# 技能库目录（可选）
open_skills_dir = "~/open-skills"

# 技能提示模式
# full   - 完整提示（技能指令 + 工具）
# compact - 紧凑模式（仅名称/描述）
prompt_injection_mode = "full"


# ================================================================================
# 十二、心跳任务配置
# ================================================================================

[heartbeat]

# 启用心跳任务
enabled = false

# 检查间隔（分钟）
interval_minutes = 30

# 心跳文件路径
heartbeat_file = "HEARTBEAT.md"


# ================================================================================
# 十三、模型路由配置
# ================================================================================

# 定义模型路由（为不同任务类型使用不同模型）

# 推理模型（复杂分析任务）
[[model_routes]]
hint = "reasoning"
provider = "openrouter"
model = "anthropic/claude-3-opus"

# 快速模型（简单对话）
[[model_routes]]
hint = "fast"
provider = "openrouter"
model = "meta-llama/llama-3-70b"

# 代码模型（编程任务）
[[model_routes]]
hint = "code"
provider = "ollama"
model = "qwen2.5-coder:32b"

# 视觉模型（图像理解）
[[model_routes]]
hint = "vision"
provider = "openai"
model = "gpt-4o"

# --------------------------------
# 查询分类规则（自动路由到不同模型）
# --------------------------------
[query_classification]

# 启用自动查询分类
enabled = true

# 分类规则（按优先级排序）

# 推理任务
[[query_classification.rules]]
hint = "reasoning"
keywords = ["解释", "分析", "为什么", "详细说明", "compare", "analyze"]
min_length = 200
priority = 10

# 快速对话
[[query_classification.rules]]
hint = "fast"
keywords = ["你好", "谢谢", "再见", "hi", "hello", "thanks"]
max_length = 50
priority = 5

# 代码任务
[[query_classification.rules]]
hint = "code"
patterns = ["```", "fn ", "pub ", "impl ", "struct ", "const "]
priority = 8


# ================================================================================
# 十四、嵌入路由配置
# ================================================================================

# 定义嵌入路由（用于记忆搜索）

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


# ================================================================================
# 十五、可观测性配置
# ================================================================================

[observability]

# 后端类型
# none        - 禁用
# log         - 日志输出
# prometheus  - Prometheus 指标
# otel        - OpenTelemetry
backend = "none"

# OpenTelemetry 配置（当 backend = "otel" 时）
# otel_endpoint = "http://localhost:4318"
# otel_service_name = "zeroclaw"


# ================================================================================
# 十六、环境变量覆盖说明
# ================================================================================

# 以下环境变量可以覆盖配置文件中的设置：

# ZEROCLAW_PROVIDER     - 提供者覆盖（如：anthropic）
# ZEROCLAW_MODEL        - 模型覆盖（如：claude-sonnet-4-6）
# ZEROCLAW_CONFIG       - 配置文件路径（如：~/.zeroclaw/prod.toml）
# ZEROCLAW_WORKSPACE    - 工作目录路径（如：/opt/zeroclaw）

# API Key 环境变量：
# ANTHROPIC_API_KEY     - Anthropic API Key
# OPENAI_API_KEY        - OpenAI API Key
# OPENROUTER_API_KEY    - OpenRouter API Key
# GROQ_API_KEY          - Groq API Key
# GEMINI_API_KEY        - Google Gemini API Key

# 其他环境变量：
# RUST_LOG              - 日志级别（debug/info/warn/error）
# ZEROCLAW_OPEN_SKILLS_ENABLED  - 开放技能开关
# ZEROCLAW_CONTAINER_CLI        - 容器 CLI（docker/podman）
```

---

## 使用指南

### 快速开始

1. **复制模板**
```bash
cp config-template.toml ~/.zeroclaw/config.toml
```

2. **填写必要配置**
```toml
# 必须配置
default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4-6"

# API Key（通过环境变量或配置文件）
export OPENROUTER_API_KEY="sk-or-..."
```

3. **验证配置**
```bash
zeroclaw status
zeroclaw doctor
```

### 场景化配置

#### 个人开发（推荐配置）

```toml
default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4-6"

[autonomy]
level = "supervised"
workspace_only = true

[memory]
backend = "sqlite"
auto_save = true

[cost]
enabled = true
daily_limit_usd = 10.00
```

#### 本地运行（零成本）

```toml
default_provider = "ollama"
default_model = "qwen2.5-coder:32b"

[agent]
compact_context = true
max_tool_iterations = 5

[memory]
embedding_provider = "none"
```

#### 生产部署

```toml
default_provider = "anthropic"
default_model = "claude-sonnet-4-6"

[runtime]
kind = "docker"
network = "none"

[gateway]
host = "127.0.0.1"
require_pairing = true

[cost]
enabled = true
daily_limit_usd = 50.00
```

---

## 相关文档

- [[ZeroClaw-核心配置详解]] - 配置项详细说明
- [[ZeroClaw-工具与安全配置详解]] - 安全配置详解
- [[ZeroClaw-通道配置详解]] - 消息通道配置

---

_最后更新：2026-02-21_
