---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, agent, configuration, reference]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw Agent 配置，ZeroClaw 子代理]
---

# ZeroClaw-Agent 配置详解

> [!summary] 本文档
> ZeroClaw Agent 系统配置详解，涵盖主 Agent 参数、子代理委托、工具执行策略等核心配置。

---

## 主 Agent 配置

### 基础配置

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

### 配置项详解

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `compact_context` | boolean | `false` | 紧凑模式适合 13B 以下小模型 |
| `max_tool_iterations` | integer | `10` | 每次请求最大工具调用轮数 |
| `max_history_messages` | integer | `50` | 保留的历史消息数 |
| `parallel_tools` | boolean | `false` | 是否并行执行独立工具调用 |
| `tool_dispatcher` | string | `"auto"` | 工具分发策略 |

### compact_context 模式

```toml
# 小模型优化（13B 以下）
[agent]
compact_context = true
```

**效果：**
- `bootstrap_max_chars = 6000`（减少上下文大小）
- `rag_chunk_limit = 2`（减少 RAG 检索块数）

**适用场景：**
- 本地小模型（Ollama 7B/13B）
- 内存受限环境
- 需要更快响应

### max_tool_iterations

```toml
[agent]
# 增加迭代次数（复杂任务）
max_tool_iterations = 20

# 减少迭代次数（简单任务）
max_tool_iterations = 5

# 禁止工具调用
max_tool_iterations = 0  # 实际会使用默认值 10
```

**超限错误：**
```
Agent exceeded maximum tool iterations (10)
```

**建议值：**
| 场景 | 建议值 |
|------|--------|
| 简单对话 | 3-5 |
| 一般任务 | 10（默认） |
| 复杂研究 | 15-20 |
| 代码审查 | 10-15 |

### parallel_tools

```toml
[agent]
# 启用并行工具执行
parallel_tools = true
```

**效果：**
- 独立工具调用并发执行
- 结果顺序保持稳定
- 减少总执行时间

**适用场景：**
- 多个独立文件读取
- 并行 API 调用
- 不依赖结果的工具组合

**注意事项：**
- 依赖顺序的工具调用仍需串行
- 调试时建议关闭以便追踪

---

## 子代理配置

### 什么是子代理

子代理允许主 Agent 将特定任务委托给专门配置的子 Agent，每个子代理可以有：
- 独立的提供者/模型
- 独立的系统提示
- 独立的工具白名单
- 独立的 API Key

### 基础配置

```toml
# 定义子代理
[agents.researcher]
provider = "openrouter"
model = "anthropic/claude-sonnet-4-6"
system_prompt = "你是一个专业的研究助手，擅长信息收集和分析。"
max_depth = 2
agentic = true
allowed_tools = ["web_search", "http_request", "file_read"]
max_iterations = 8

[agents.coder]
provider = "ollama"
model = "qwen2.5-coder:32b"
temperature = 0.2
system_prompt = "你是一个专业的软件工程师，擅长代码编写和审查。"
agentic = true
allowed_tools = ["shell", "file_read", "file_write"]
max_iterations = 15
```

### 子代理配置项

| 配置项 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `provider` | string | ✅ | - | 提供者名称 |
| `model` | string | ✅ | - | 模型名称 |
| `system_prompt` | string | ❌ | unset | 系统提示覆盖 |
| `api_key` | string | ❌ | unset | API Key 覆盖（加密存储） |
| `temperature` | float | ❌ | unset | 温度覆盖 |
| `max_depth` | integer | ❌ | `3` | 最大递归深度 |
| `agentic` | boolean | ❌ | `false` | 启用多轮工具调用 |
| `allowed_tools` | array | ❌ | `[]` | 工具白名单 |
| `max_iterations` | integer | ❌ | `10` | 最大工具迭代次数 |

### agentic 模式

```toml
[agents.researcher]
# 单轮模式（默认）
agentic = false
# 仅执行一次 prompt→response

[agents.coder]
# 多轮模式
agentic = true
# 支持工具调用循环
allowed_tools = ["shell", "file_read", "file_write"]
max_iterations = 15
```

**区别：**
| 模式 | 行为 | 适用场景 |
|------|------|----------|
| `agentic = false` | 单次 prompt→response | 简单查询、总结 |
| `agentic = true` | 多轮工具调用循环 | 复杂任务、代码编写 |

### 工具白名单

```toml
[agents.researcher]
agentic = true
allowed_tools = [
    "web_search",      # 网络搜索
    "http_request",    # API 调用
    "file_read",       # 读取文件
    "web_fetch"        # 网页抓取
]

[agents.coder]
agentic = true
allowed_tools = [
    "shell",           # Shell 命令
    "file_read",       # 读取文件
    "file_write",      # 写入文件
    "git"              # Git 操作
]

[agents.analyst]
agentic = true
allowed_tools = [
    "shell",           # 数据分析命令
    "file_read",
    "http_request"     # 获取外部数据
]
```

**注意事项：**
- `delegate` 工具自动从子代理白名单排除（防止递归循环）
- 至少需要一个工具才能启用 `agentic = true`

### 递归深度

```toml
[agents.researcher]
max_depth = 2  # 最多允许 2 层递归委托
```

**递归示例：**
```
用户 → 主 Agent → researcher → (委托) → coder
                              ↓
                          深度 = 2
```

**建议：**
- 简单任务：`max_depth = 1`
- 复杂任务：`max_depth = 2-3`
- 避免过深：`max_depth > 4` 可能导致复杂度过高

---

## 子代理使用场景

### 场景 1：研究助手

```toml
[agents.researcher]
provider = "openrouter"
model = "anthropic/claude-sonnet-4-6"
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
agentic = true
allowed_tools = ["web_search", "web_fetch", "http_request", "file_read", "file_write"]
max_iterations = 12
max_depth = 2
```

**使用方式：**
```bash
zeroclaw agent -m "帮我研究一下 ZeroClaw 和 OpenClaw 的区别"
# 主 Agent 会自动委托给 researcher 子代理
```

### 场景 2：代码专家

```toml
[agents.coder]
provider = "ollama"
model = "qwen2.5-coder:32b"
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
```

### 场景 3：数据分析师

```toml
[agents.analyst]
provider = "openai"
model = "gpt-4o"
system_prompt = """
你是一个数据分析师。你的职责：
1. 分析和解释数据
2. 生成统计报告
3. 创建数据可视化建议
4. 识别趋势和异常

输出要求：
- 使用清晰的表格呈现数据
- 提供关键指标摘要
- 指出数据局限性
"""
agentic = true
allowed_tools = ["shell", "file_read", "http_request"]
max_iterations = 10
max_depth = 1
```

### 场景 4：多代理协作

```toml
# 研究 + 编码协作
[agents.researcher]
provider = "anthropic"
model = "claude-sonnet-4-6"
agentic = true
allowed_tools = ["web_search", "web_fetch"]
max_iterations = 8

[agents.coder]
provider = "ollama"
model = "qwen2.5-coder:32b"
agentic = true
allowed_tools = ["shell", "file_read", "file_write"]
max_iterations = 15

# 使用示例
# 用户："帮我创建一个爬取 GitHub 趋势的 Rust 项目"
# 流程：
# 1. 主 Agent → researcher: 研究 GitHub 趋势 API
# 2. researcher → 主 Agent: 返回 API 文档
# 3. 主 Agent → coder: 根据 API 编写代码
# 4. coder → 主 Agent: 返回完整项目
```

---

## 工具执行策略

### 工具分发策略

```toml
[agent]
tool_dispatcher = "auto"  # auto | round_robin | load_balance
```

**策略说明：**
| 策略 | 描述 | 适用场景 |
|------|------|----------|
| `auto` | 自动选择最佳工具 | 默认推荐 |
| `round_robin` | 轮询分发 | 多个同类工具 |
| `load_balance` | 负载均衡 | 高并发场景 |

### 工具超时配置

```toml
[tools]
# 全局超时（秒）
timeout_secs = 120

# 单个工具超时
[tools.shell]
timeout_secs = 60

[tools.http_request]
timeout_secs = 30

[tools.web_search]
timeout_secs = 30
```

### 工具重试配置

```toml
[tools.http_request]
# 重试配置
max_retries = 3
retry_delay_secs = 2
retry_on_status = [429, 500, 502, 503, 504]
```

---

## 上下文管理

### 历史消息限制

```toml
[agent]
# 保留最近 50 条消息
max_history_messages = 50

# 小上下文（节省 Token）
max_history_messages = 20

# 大上下文（完整历史）
max_history_messages = 100
```

### 上下文压缩

```toml
[agent]
# 启用紧凑模式
compact_context = true
```

**效果：**
- 减少 bootstrap 上下文到 6000 字符
- RAG 检索限制为 2 个块
- 适合小模型或 Token 受限场景

---

## 调试和监控

### 查看 Agent 状态

```bash
# 查看当前 Agent 配置
zeroclaw status

# 测试 Agent 响应
zeroclaw agent -m "Hello" --verbose

# 测试子代理
zeroclaw agent -m "研究 Rust 最新特性" --verbose
```

### 日志分析

```bash
# 启用详细日志
RUST_LOG=debug zeroclaw daemon

# 查看工具调用日志
grep "tool_call" ~/.zeroclaw/zeroclaw.log

# 查看子代理委托
grep "delegate" ~/.zeroclaw/zeroclaw.log
```

### 性能监控

```bash
# 查看工具执行时间
zeroclaw agent -m "任务" --verbose 2>&1 | grep "duration"

# 监控 Token 使用
zeroclaw status --verbose
```

---

## 最佳实践

### 1. 合理配置子代理

```toml
# ✅ 推荐：专业化分工
[agents.researcher]
allowed_tools = ["web_search", "web_fetch"]

[agents.coder]
allowed_tools = ["shell", "file_write"]

# ❌ 不推荐：所有工具都给所有代理
[agents.*]
allowed_tools = ["*"]  # 安全风险
```

### 2. 限制递归深度

```toml
# ✅ 推荐
max_depth = 2

# ❌ 避免
max_depth = 10  # 可能导致复杂度过高
```

### 3. 使用本地模型降低成本

```toml
# 子代理使用本地模型
[agents.coder]
provider = "ollama"
model = "qwen2.5-coder:32b"

# 主代理使用云模型（高质量）
default_provider = "anthropic"
default_model = "claude-sonnet-4-6"
```

### 4. 明确系统提示

```toml
# ✅ 推荐：具体明确的提示
system_prompt = """
你是一个专业的研究助手。你的职责：
1. 使用网络搜索收集信息
2. 验证信息来源的可靠性
3. 整理和总结关键发现
"""

# ❌ 避免：模糊的提示
system_prompt = "帮助我完成任务"
```

---

## 相关文档

- [[ZeroClaw-工具集成]] - 工具开发和配置
- [[ZeroClaw-核心配置详解]] - 基础配置
- [[ZeroClaw-完整配置指南]] - 配置总览

---

_最后更新：2026-02-21_
