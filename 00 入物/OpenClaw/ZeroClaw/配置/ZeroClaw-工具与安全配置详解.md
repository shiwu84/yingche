---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, tools, security, configuration, reference]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 工具配置，ZeroClaw 安全配置]
---

# ZeroClaw-工具与安全配置详解

> [!summary] 本文档
> ZeroClaw 工具系统配置详解，涵盖 Shell、HTTP、网络搜索等工具配置，以及自主级别、命令白名单等安全策略。

---

## 自主级别配置

### autonomy.level

```toml
[autonomy]
# 自主级别
level = "supervised"  # readonly | supervised | full
```

### 自主级别说明

| 级别 | 描述 | 工具调用 | 审批要求 | 适用场景 |
|------|------|----------|----------|----------|
| `readonly` | 只读模式 | 仅读取类工具 | 所有写操作禁止 | 审计、监控、学习 |
| `supervised` | 监督模式 | 全部工具 | 中等风险需审批 | 日常使用（推荐） |
| `full` | 完全自主 | 全部工具 | 仅高风险审批 | 可信环境、自动化 |

### 风险级别定义

| 风险级别 | 示例命令 | 默认行为 |
|----------|----------|----------|
| 低风险 | `ls`, `cat`, `pwd`, `git status` | 自动批准 |
| 中风险 | `git commit`, `npm install`, `cargo build` | 需要审批 |
| 高风险 | `rm -rf`, `sudo`, `dd`, `mkfs` | 禁止执行 |

---

## Shell 工具配置

### 基础配置

```toml
[autonomy]
# 限制在工作目录内
workspace_only = true

# 允许的命令白名单
allowed_commands = [
    "ls", "cat", "grep", "find", "head", "tail",
    "pwd", "wc", "sort", "uniq", "awk", "sed",
    "git", "npm", "cargo", "curl", "wget",
    "rsync", "tar", "gzip", "zip", "unzip"
]

# 禁止的命令黑名单
forbidden_commands = [
    "rm", "sudo", "chmod", "chown", "dd", "mkfs",
    "shutdown", "reboot", "kill", "pkill",
    "iptables", "firewall-cmd"
]

# 禁止访问的路径
forbidden_paths = [
    "/etc", "/root", "/proc", "/sys", "/boot",
    "/dev", "/var/log", "~/.ssh", "~/.gnupg",
    "~/.aws", "~/.config", "~/.local"
]
```

### 命令白名单最佳实践

#### 开发场景

```toml
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
    "go", "go mod",
    
    # 网络
    "curl", "wget", "ssh", "scp",
    
    # 压缩
    "tar", "gzip", "zip", "unzip", "7z",
    
    # 系统信息
    "pwd", "whoami", "uname", "hostname",
    "ps", "top", "free", "df", "du"
]
```

#### 数据分析场景

```toml
allowed_commands = [
    # Python 相关
    "python", "python3", "ipython",
    "pip", "pip3",
    
    # 数据处理
    "jq", "csvkit", "sqlite3",
    
    # 文件操作
    "ls", "cat", "head", "tail", "wc",
    "find", "grep", "awk", "sed",
    
    # 网络
    "curl", "wget"
]
```

### 路径限制

```toml
[autonomy]
# 工作目录限制
workspace_only = true

# 明确允许的路径（覆盖 workspace_only）
allowed_paths = [
    "/home/user/workspace",
    "/home/user/projects",
    "/tmp/zeroclaw"
]

# 明确禁止的路径
forbidden_paths = [
    "/etc",                    # 系统配置
    "/root",                   # root 用户目录
    "/proc", "/sys",           # 虚拟文件系统
    "/boot",                   # 引导分区
    "~/.ssh",                  # SSH 密钥
    "~/.gnupg",                # GPG 密钥
    "~/.aws",                  # AWS 凭证
    "~/.config/gh",            # GitHub 凭证
    "~/.kube/config"           # Kubernetes 配置
]
```

### 操作限制

```toml
[autonomy]
# 每小时最大操作数
max_actions_per_hour = 100

# 每日最大操作数
max_actions_per_day = 1000

# 单次请求最大操作数
max_actions_per_request = 10
```

### 审批配置

```toml
[autonomy]
# 中等风险命令需要审批
require_approval_for_medium_risk = true

# 禁止高风险命令
block_high_risk_commands = true

# 自动批准的操作（即使在 supervised 模式）
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
```

### Shell 命令解析

```toml
[autonomy]
# 启用严格命令解析
strict_parsing = true
```

**解析规则：**
- 分号 `;` 分隔的命令会被分别检查
- 管道 `|` 连接的每个命令都会被检查
- 重定向 `>` `>>` 会被检查目标路径
- 引号内的字符不作为分隔符

**示例：**
```bash
# ✅ 允许（如果 ls 和 grep 在白名单）
ls -la | grep ".md"

# ❌ 禁止（rm 在黑名单）
ls -la | grep ".tmp" | xargs rm

# ❌ 禁止（重定向到禁止路径）
cat file.txt > /etc/config

# ✅ 允许（重定向到工作目录）
cat file.txt > output.txt
```

---

## HTTP 请求工具配置

### 基础配置

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

# 禁止的域名
forbidden_domains = [
    "malicious-site.com",
    "phishing-example.com"
]

# 最大响应大小（字节）
max_response_size = 1000000  # 1MB

# 请求超时（秒）
timeout_secs = 30

# 请求头配置
[tools.http_request.headers]
User-Agent = "ZeroClaw/1.0"
Accept = "application/json"
```

### 域名匹配规则

```toml
[tools.http_request]
# 精确匹配
allowed_domains = ["api.github.com"]

# 子域名匹配（允许所有 *.github.com）
allowed_domains = ["github.com"]

# 通配符（不推荐）
# allowed_domains = ["*"]  # 允许所有域名
```

### 方法限制

```toml
[tools.http_request]
# 允许的 HTTP 方法
allowed_methods = ["GET", "POST"]

# 禁止的方法
forbidden_methods = ["DELETE", "PATCH"]
```

### 认证配置

```toml
[tools.http_request]
# 自动添加认证头
[tools.http_request.auth]
# Bearer Token
type = "bearer"
token = "your-api-token"

# 或 Basic Auth
# type = "basic"
# username = "user"
# password = "pass"

# 或自定义头
# type = "header"
# header_name = "X-API-Key"
# header_value = "your-key"
```

### 重试配置

```toml
[tools.http_request]
# 重试策略
max_retries = 3
retry_delay_secs = 2
retry_on_status = [429, 500, 502, 503, 504]
retry_on_timeout = true
```

---

## 网络搜索工具配置

### 基础配置

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

# 安全搜索
safe_search = true
```

### 提供者配置对比

#### Brave Search

```toml
[tools.web_search]
provider = "brave"
api_key = "YOUR_BRAVE_API_KEY"  # 2000 次/月免费
max_results = 10
freshness = "pw"
```

**特点：**
- ✅ 高质量搜索结果
- ✅ 隐私保护
- ❌ 需要 API Key
- ❌ 免费额度有限

#### DuckDuckGo

```toml
[tools.web_search]
provider = "duckduckgo"
# 无需 API Key
max_results = 10
```

**特点：**
- ✅ 无需 API Key
- ✅ 隐私保护
- ❌ 功能有限
- ❌ 结果质量一般

#### Google Custom Search

```toml
[tools.web_search]
provider = "google"
api_key = "YOUR_GOOGLE_API_KEY"
search_engine_id = "YOUR_CX_ID"
max_results = 10
```

**特点：**
- ✅ 最全面的搜索结果
- ❌ 需要 API Key + Search Engine ID
- ❌ 免费额度 100 次/天

#### Tavily

```toml
[tools.web_search]
provider = "tavily"
api_key = "YOUR_TAVILY_API_KEY"
max_results = 10
search_depth = "basic"  # basic | advanced
```

**特点：**
- ✅ AI 优化的搜索结果
- ✅ 返回摘要
- ❌ 需要 API Key
- ❌ 1000 次/月免费

#### Exa

```toml
[tools.web_search]
provider = "exa"
api_key = "YOUR_EXA_API_KEY"
max_results = 10
type = "neural"  # neural | keyword
```

**特点：**
- ✅ 语义搜索
- ✅ 技术内容友好
- ❌ 需要 API Key
- ❌ 1000 次/月免费

---

## 网页抓取工具配置

### 基础配置

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
    "stackoverflow.com",
    "rust-lang.org"
]

# 禁止的域名
forbidden_domains = [
    "malicious-site.com"
]

# 请求头
[tools.web_fetch.headers]
User-Agent = "ZeroClaw/1.0 Web Fetcher"
Accept = "text/html,application/xhtml+xml"
```

### 内容过滤

```toml
[tools.web_fetch]
# 移除的元素
remove_elements = [
    "script", "style", "nav", "footer",
    ".advertisement", ".sidebar"
]

# 保留的元素
keep_elements = [
    "article", "main", "section",
    ".content", ".post"
]
```

---

## 浏览器工具配置

### 基础配置

```toml
[browser]
# 启用浏览器工具
enabled = false  # 按需开启

# 允许的域名
allowed_domains = [
    "docs.rs",
    "github.com",
    "wiki.archlinux.org"
]

# 浏览器后端
backend = "agent_browser"  # agent_browser | rust_native | computer_use | auto

# 会话配置
session_name = "zeroclaw-browser"
session_persist = true
session_dir = "~/.zeroclaw/browser-sessions"
```

### Rust Native 后端

```toml
[browser]
backend = "rust_native"

# Chrome/Chromium 配置
native_headless = true
native_chrome_path = "/usr/bin/google-chrome"
native_user_data_dir = "~/.zeroclaw/chrome-data"

# WebDriver 配置
native_webdriver_url = "http://127.0.0.1:9515"
native_webdriver_args = [
    "--no-sandbox",
    "--disable-dev-shm-usage",
    "--disable-gpu"
]
```

### Computer Use 后端

```toml
[browser]
backend = "computer_use"

[browser.computer_use]
# 侧车端点
endpoint = "http://127.0.0.1:8787/v1/actions"

# 认证
api_key = ""  # 可选

# 超时
timeout_ms = 15000

# 安全：禁止远程端点
allow_remote_endpoint = false

# 窗口白名单
window_allowlist = [
    "Google Chrome",
    "Chromium"
]

# 坐标限制（可选）
max_coordinate_x = 1920
max_coordinate_y = 1080
```

---

## Composio 集成配置

### 基础配置

```toml
[composio]
# 启用 Composio
enabled = false

# Composio API Key
api_key = "YOUR_COMPOSIO_API_KEY"

# 默认用户 ID
entity_id = "default"

# 端点（可选）
endpoint = "https://backend.composio.dev"
```

### 使用流程

1. **配置 Composio**
```toml
[composio]
enabled = true
api_key = "YOUR_API_KEY"
```

2. **连接应用**
```bash
zeroclaw agent -m "帮我连接 GitHub"
# Agent 会调用 composio connect 工具
# 在浏览器完成 OAuth 授权
```

3. **使用工具**
```bash
zeroclaw agent -m "获取我的 GitHub 通知"
# Agent 会调用 composio execute 工具
```

### 支持的应用

Composio 支持 1000+ OAuth 应用：
- GitHub, GitLab, Bitbucket
- Slack, Discord, Telegram
- Google, Microsoft, Notion
- Linear, Jira, Trello
- 等等...

---

## 安全最佳实践

### 1. 最小权限原则

```toml
# ✅ 推荐：精确的白名单
allowed_commands = ["ls", "cat", "git"]

# ❌ 避免：过于宽松
allowed_commands = ["*"]
```

### 2. 路径隔离

```toml
# ✅ 推荐：限制工作目录
workspace_only = true
allowed_paths = ["/home/user/workspace"]

# ❌ 避免：无限制
workspace_only = false
```

### 3. 敏感路径保护

```toml
# ✅ 推荐：明确禁止敏感路径
forbidden_paths = [
    "/etc", "/root", "~/.ssh", "~/.aws"
]
```

### 4. 操作审计

```toml
# ✅ 推荐：启用日志
[logging]
level = "info"
file = "/var/log/zeroclaw/daemon.log"

# 定期审查日志
# grep "shell_exec" /var/log/zeroclaw/daemon.log
```

### 5. Docker 沙箱

```toml
# 高安全场景使用沙箱
[runtime]
kind = "docker"
network = "none"
read_only_rootfs = true
```

---

## 相关文档

- [[ZeroClaw-沙箱运行时]] - Docker 沙箱配置
- [[ZeroClaw-核心配置详解]] - 基础配置
- [[ZeroClaw-完整配置指南]] - 配置总览

---

_最后更新：2026-02-21_
