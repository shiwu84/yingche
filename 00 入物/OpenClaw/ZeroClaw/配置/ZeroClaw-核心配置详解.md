---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, configuration, core, reference]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 核心配置，ZeroClaw 基础设置]
---

# ZeroClaw-核心配置详解

> [!summary] 本文档
> ZeroClaw 核心配置项详解，涵盖默认模型、提供者选择、环境变量覆盖等基础但关键的配置。

---

## 配置文件位置与加载顺序

### 配置解析路径

ZeroClaw 启动时按以下优先级查找配置文件：

```
优先级 1: ZEROCLAW_WORKSPACE 环境变量
           ↓
优先级 2: ~/.zeroclaw/active_workspace.toml 标记文件
           ↓
优先级 3: ~/.zeroclaw/config.toml (默认)
```

### 验证配置加载

```bash
# 查看配置加载信息（启动日志）
zeroclaw daemon --log-level info 2>&1 | grep "Config loaded"

# 查看当前状态和配置路径
zeroclaw status

# 导出配置 Schema 验证结构
zeroclaw config schema | jq '.properties'
```

### 工作目录结构

```
~/.zeroclaw/
├── config.toml              # 主配置文件
├── active_workspace.toml    # 工作空间标记（可选）
├── auth-profiles.json       # 认证配置（加密）
├── .secret_key              # 加密密钥
├── zeroclaw.log             # 日志文件
├── memory/                  # 记忆存储
│   └── sqlite.db
└── workspace/               # 工作目录
    ├── MEMORY.md
    └── ...
```

---

## 默认模型配置

### 基础配置项

```toml
# ~/.zeroclaw/config.toml
# ================================
# ZeroClaw 核心配置文件
# ================================

# --------------------------------
# 默认模型配置
# --------------------------------

# 默认提供者 ID
# 可选值：openrouter, anthropic, openai, ollama, groq, deepseek, gemini 等
# 完整列表见下文"内置提供者列表"
default_provider = "openrouter"

# 默认模型 ID
# 格式：提供者/模型名 或 直接模型名
# 常用模型：claude-sonnet-4-6, gpt-4o, llama-3-70b, qwen2.5-coder:32b
default_model = "anthropic/claude-sonnet-4-6"

# 模型温度（0.0 - 2.0）
# 控制输出随机性：值越低越确定，值越高越有创意
# 建议：代码生成 0.2, 日常对话 0.7, 创意写作 1.0+
default_temperature = 0.7
```

### 配置项详解

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `default_provider` | string | `"openrouter"` | 提供者 ID 或别名 |
| `default_model` | string | `"anthropic/claude-sonnet-4-6"` | 模型 ID |
| `default_temperature` | float | `0.7` | 温度值，控制随机性 |

### 温度值建议

| 温度 | 适用场景 | 示例 |
|------|----------|------|
| 0.0 - 0.3 | 代码生成、事实查询 | `temperature = 0.2` |
| 0.4 - 0.7 | 日常对话、一般任务 | `temperature = 0.7` (默认) |
| 0.8 - 1.2 | 创意写作、头脑风暴 | `temperature = 1.0` |
| 1.3 - 2.0 | 高度创意、实验性 | `temperature = 1.5` |

---

## 提供者配置

### 内置提供者列表

ZeroClaw 支持 29+ 内置提供者：

#### 主流云 API

| 提供者 | ID | API Key 环境变量 | 官网 |
|--------|-----|------------------|------|
| OpenRouter | `openrouter` | `OPENROUTER_API_KEY` | openrouter.ai |
| Anthropic | `anthropic` | `ANTHROPIC_API_KEY` | anthropic.com |
| OpenAI | `openai` | `OPENAI_API_KEY` | platform.openai.com |
| Google Gemini | `gemini` | `GEMINI_API_KEY` | ai.google.dev |
| Cohere | `cohere` | `COHERE_API_KEY` | cohere.com |

#### 高性能推理

| 提供者 | ID | 特点 |
|--------|-----|------|
| Groq | `groq` | 极速推理（LPU） |
| Fireworks | `fireworks` | 高速推理 |
| Together AI | `together-ai` | 开源模型托管 |
| DeepSeek | `deepseek` | 高性价比 |

#### 开源/本地模型

| 提供者 | ID | 部署方式 |
|--------|-----|----------|
| Ollama | `ollama` | 本地运行 |
| LM Studio | `llamacpp` | 本地运行 |
| vLLM | `vllm` | 本地/服务器 |

#### 区域提供者

| 提供者 | ID | 地区 |
|--------|-----|------|
| Moonshot | `moonshot` | 中国（月之暗面） |
| Zhipu | `glm` | 中国（智谱 AI） |
| Z.AI | `zai` | 中国 |
| Qwen | `qwen` | 中国（阿里云） |
| Nvidia | `nvidia` | 全球 |

#### 其他提供者

| 提供者 | ID | 备注 |
|--------|-----|------|
| Mistral | `mistral` | 欧洲 |
| XAI | `xai` | Grok 模型 |
| Venice | `venice` | 隐私优先 |
| AstrAI | `astrai` | - |

### 自定义提供者

```toml
# 使用 OpenAI 兼容接口
default_provider = "custom:https://api.example.com/v1"

# 使用 Anthropic 兼容接口
default_provider = "anthropic-custom:https://api.example.com"
```

---

## 环境变量覆盖

### 优先级规则

```
环境变量 ZEROCLAW_PROVIDER > 配置文件 default_provider
```

### 核心环境变量

| 变量名 | 描述 | 示例 |
|--------|------|------|
| `ZEROCLAW_PROVIDER` | 提供者覆盖 | `anthropic` |
| `ZEROCLAW_MODEL` | 模型覆盖 | `claude-sonnet-4-6` |
| `ZEROCLAW_CONFIG` | 配置文件路径 | `~/.zeroclaw/prod.toml` |
| `ZEROCLAW_WORKSPACE` | 工作目录 | `/opt/zeroclaw` |

### API Key 环境变量

```bash
# Anthropic
export ANTHROPIC_API_KEY="sk-ant-..."
export ANTHROPIC_AUTH_TOKEN="..."  # OAuth 令牌

# OpenAI
export OPENAI_API_KEY="sk-..."

# OpenRouter
export OPENROUTER_API_KEY="sk-or-..."

# Groq
export GROQ_API_KEY="gsk_..."

# Google Gemini
export GEMINI_API_KEY="..."

# Cohere
export COHERE_API_KEY="..."
```

### 其他环境变量

```bash
# 日志级别
export RUST_LOG="info"      # debug | info | warn | error

# 技能配置
export ZEROCLAW_OPEN_SKILLS_ENABLED="true"
export ZEROCLAW_OPEN_SKILLS_DIR="~/my-skills"
export ZEROCLAW_SKILLS_PROMPT_MODE="compact"

# 容器 CLI（Docker 沙箱）
export ZEROCLAW_CONTAINER_CLI="podman"

# Lucid 内存后端
export ZEROCLAW_LUCID_CMD="/usr/local/bin/lucid"
export ZEROCLAW_LUCID_BUDGET="200"
export ZEROCLAW_LUCID_LOCAL_HIT_THRESHOLD="3"
export ZEROCLAW_LUCID_RECALL_TIMEOUT_MS="120"
export ZEROCLAW_LUCID_STORE_TIMEOUT_MS="800"
export ZEROCLAW_LUCID_FAILURE_COOLDOWN_MS="15000"

# Nextcloud Talk
export ZEROCLAW_NEXTCLOUD_TALK_WEBHOOK_SECRET="secret"
```

---

## 模型路由配置

### 什么是模型路由

模型路由允许你为不同任务类型配置不同的模型，通过 `hint` 进行抽象引用。

### 配置示例

```toml
# 定义模型路由
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

[[model_routes]]
hint = "vision"
provider = "openai"
model = "gpt-4o"
```

### 使用路由

```toml
# 在配置中引用路由
default_model = "hint:reasoning"

# 或在内存嵌入中使用
[memory]
embedding_model = "hint:semantic"
```

### 查询分类（自动路由）

```toml
# 启用自动查询分类
[query_classification]
enabled = true

# 定义分类规则（按优先级排序）
[[query_classification.rules]]
hint = "reasoning"
keywords = ["解释", "分析", "为什么", "详细说明", "compare", "analyze"]
min_length = 200
priority = 10

[[query_classification.rules]]
hint = "fast"
keywords = ["你好", "谢谢", "再见", "hi", "hello", "thanks"]
max_length = 50
priority = 5

[[query_classification.rules]]
hint = "code"
patterns = ["```", "fn ", "pub ", "impl ", "struct ", "const "]
priority = 8

[[query_classification.rules]]
hint = "vision"
keywords = ["图片", "图像", "看图", "分析这张图"]
priority = 7
```

### 分类规则参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `hint` | string | ✅ | 必须匹配 `[[model_routes]]` 中的 hint |
| `keywords` | array | ❌ | 不区分大小写的子字符串匹配 |
| `patterns` | array | ❌ | 区分大小写的字面匹配 |
| `min_length` | integer | ❌ | 消息最小长度 |
| `max_length` | integer | ❌ | 消息最大长度 |
| `priority` | integer | ❌ | 优先级（高优先级先匹配） |

---

## 嵌入路由配置

### 配置嵌入模型

```toml
# 定义嵌入路由
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

[[embedding_routes]]
hint = "code"
provider = "openai"
model = "text-embedding-3-large"
dimensions = 3072
```

### 嵌入提供者

| 提供者 | ID | 说明 |
|--------|-----|------|
| 无 | `none` | 不使用嵌入 |
| OpenAI | `openai` | OpenAI 嵌入 API |
| 自定义 | `custom:https://...` | 自定义嵌入端点 |

### 使用嵌入路由

```toml
[memory]
# 引用嵌入路由
embedding_model = "hint:semantic"

# 或直接指定
# embedding_model = "text-embedding-3-small"
```

---

## 提供者切换最佳实践

### 开发环境

```toml
# 使用本地模型（免费、快速迭代）
default_provider = "ollama"
default_model = "qwen2.5-coder:7b"
default_temperature = 0.7
```

### 生产环境

```toml
# 使用云 API（高质量、稳定）
default_provider = "anthropic"
default_model = "claude-sonnet-4-6"
default_temperature = 0.7
```

### 成本敏感

```toml
# 使用高性价比模型
default_provider = "deepseek"
default_model = "deepseek-chat"
default_temperature = 0.7
```

### 隐私优先

```toml
# 完全本地运行
default_provider = "ollama"
default_model = "llama-3-70b"

[privacy]
no_telemetry = true
local_only = true
```

---

## 配置验证与调试

### 验证命令

```bash
# 查看当前配置摘要
zeroclaw status

# 完整诊断
zeroclaw doctor

# 查看提供者列表
zeroclaw providers

# 刷新模型目录
zeroclaw models refresh

# 测试 Agent
zeroclaw agent -m "Hello" --verbose
```

### 调试技巧

```bash
# 启用详细日志
RUST_LOG=debug zeroclaw daemon

# 查看配置加载日志
zeroclaw daemon 2>&1 | grep -E "(Config|provider|model)"

# 测试特定提供者
zeroclaw agent --provider anthropic -m "test"

# 测试模型路由
zeroclaw agent --model "hint:reasoning" -m "分析这个问题"
```

### 常见问题

#### 问题 1：提供者无法识别

```toml
# ❌ 错误：拼写错误
default_provider = "anthopic"

# ✅ 正确
default_provider = "anthropic"
```

#### 问题 2：模型不存在

```bash
# 刷新模型目录
zeroclaw models refresh --provider openrouter

# 查看可用模型
zeroclaw models list | grep claude
```

#### 问题 3：API Key 无效

```bash
# 验证 API Key
curl -X POST https://api.anthropic.com/v1/messages \
  -H "Authorization: Bearer $ANTHROPIC_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-3-haiku-20240307","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
```

---

## 相关文档

- [[ZeroClaw-认证机制]] - API Key 和 OAuth 配置
- [[ZeroClaw-完整配置指南]] - 配置总览
- [[ZeroClaw-常用命令]] - CLI 命令参考

---

_最后更新：2026-02-21_
