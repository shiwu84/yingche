---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, memory, storage, configuration, reference]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 内存配置，ZeroClaw 记忆系统]
---

# ZeroClaw-内存与存储配置详解

> [!summary] 本文档
> ZeroClaw 内存系统配置详解，涵盖 SQLite、Lucid、PostgreSQL 等后端配置，以及嵌入模型、混合搜索、成本追踪等高级功能。

---

## 内存系统概述

### 支持的内存后端

| 后端 | 类型 | 适用场景 | 复杂度 |
|------|------|----------|--------|
| SQLite | 本地文件 | 默认推荐 | 简单 |
| Lucid | 外部服务 | 高级搜索 | 中等 |
| PostgreSQL | 远程数据库 | 企业部署 | 复杂 |
| Markdown | 文本文件 | 人工可读 | 简单 |
| None | 无持久化 | 临时会话 | 简单 |

### 内存系统架构

```
┌─────────────────────────────────────────────────┐
│              Agent 记忆管理                      │
│  自动保存 │ 召回 │  chunking │ 嵌入             │
└────────────────────┬────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│            混合搜索引擎                          │
│  向量搜索 (cosine) + 关键词搜索 (BM25)          │
│  加权合并 (vector_weight + keyword_weight)      │
└────────────────────┬────────────────────────────┘
                     ↓
        ┌────────────┴────────────┐
        ↓                         ↓
┌───────────────┐         ┌───────────────┐
│  SQLite       │         │  PostgreSQL   │
│  (FTS5 + BLOB)│         │  (pgvector)   │
└───────────────┘         └───────────────┘
```

---

## SQLite 后端配置（默认）

### 基础配置

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

### 配置项详解

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `backend` | string | `"sqlite"` | 内存后端类型 |
| `auto_save` | boolean | `true` | 是否自动保存用户输入 |
| `embedding_provider` | string | `"none"` | 嵌入提供者 |
| `embedding_model` | string | `"text-embedding-3-small"` | 嵌入模型 |
| `embedding_dimensions` | integer | `1536` | 嵌入向量维度 |
| `vector_weight` | float | `0.7` | 向量搜索权重 |
| `keyword_weight` | float | `0.3` | 关键词搜索权重 |
| `sqlite_open_timeout_secs` | integer | `30` | SQLite 打开超时 |

### SQLite 文件位置

```
~/.zeroclaw/memory/sqlite.db
```

**目录结构：**
```
~/.zeroclaw/
└── memory/
    ├── sqlite.db           # 主数据库
    ├── sqlite.db-shm       # 共享内存
    ├── sqlite.db-wal       # 预写日志
    └── embedding_cache.db  # 嵌入缓存（可选）
```

---

## 嵌入配置

### 嵌入提供者

#### OpenAI 嵌入

```toml
[memory]
embedding_provider = "openai"
embedding_model = "text-embedding-3-small"
embedding_dimensions = 1536
```

**支持模型：**
| 模型 | 维度 | 价格 | 性能 |
|------|------|------|------|
| `text-embedding-3-small` | 1536 | $ | 良好 |
| `text-embedding-3-large` | 3072 | $$ | 优秀 |
| `text-embedding-ada-002` | 1536 | $ | 良好 |

#### 自定义嵌入端点

```toml
[memory]
embedding_provider = "custom:https://api.example.com/v1"
embedding_model = "bge-large-zh"
embedding_dimensions = 1024
```

#### 无嵌入（仅关键词搜索）

```toml
[memory]
embedding_provider = "none"
# 仅使用 FTS5 关键词搜索
```

### 嵌入路由

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

# 使用路由
[memory]
embedding_model = "hint:semantic"
```

### 嵌入缓存

```toml
[memory]
# 嵌入缓存配置
embedding_cache_size = 10000  # 缓存条目数
embedding_cache_ttl_secs = 86400  # 缓存 TTL（24 小时）
```

---

## Lucid 后端配置

### 什么是 Lucid

Lucid 是 ZeroClaw 的外部记忆服务，提供：
- 分布式存储
- 高级搜索
- 跨会话记忆共享

### 基础配置

```toml
[memory]
backend = "lucid"

# Lucid 命令（可选，默认使用 PATH 中的 lucid）
# lucid_cmd = "/usr/local/bin/lucid"

# 预算限制（可选）
# lucid_budget = 200

# 本地命中阈值（可选）
# lucid_local_hit_threshold = 3

# 召回超时（毫秒）
# lucid_recall_timeout_ms = 120

# 存储超时（毫秒）
# lucid_store_timeout_ms = 800

# 失败冷却（毫秒）
# lucid_failure_cooldown_ms = 15000
```

### 环境变量配置

```bash
export ZEROCLAW_LUCID_CMD="/usr/local/bin/lucid"
export ZEROCLAW_LUCID_BUDGET="200"
export ZEROCLAW_LUCID_LOCAL_HIT_THRESHOLD="3"
export ZEROCLAW_LUCID_RECALL_TIMEOUT_MS="120"
export ZEROCLAW_LUCID_STORE_TIMEOUT_MS="800"
export ZEROCLAW_LUCID_FAILURE_COOLDOWN_MS="15000"
```

### 混合模式（本地 + Lucid）

```toml
[memory]
# 本地优先，未命中时使用 Lucid
backend = "lucid"

# 本地命中阈值
lucid_local_hit_threshold = 3  # 本地命中 3 次后跳过 Lucid

# 低延迟召回预算
lucid_recall_timeout_ms = 120  # 超时后降级为本地搜索
```

---

## PostgreSQL 后端配置

### 适用场景

- 多实例共享记忆
- 企业级部署
- 需要高可用性

### 基础配置

```toml
[memory]
backend = "sqlite"  # 保持 sqlite

# PostgreSQL 存储提供者配置
[storage.provider.config]
provider = "postgres"
db_url = "postgres://user:password@host:5432/zeroclaw"
schema = "public"
table = "memories"
connect_timeout_secs = 15
```

### 环境变量

```bash
export DATABASE_URL="postgres://user:password@host:5432/zeroclaw"
```

### 数据库 Schema

```sql
-- 记忆表
CREATE TABLE memories (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    embedding vector(1536),  -- pgvector
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- FTS5 虚拟表（关键词搜索）
CREATE INDEX memories_content_idx ON memories USING GIN (to_tsvector('simple', content));
```

---

## Markdown 后端配置

### 适用场景

- 人工可读的记忆存储
- 与 Obsidian 等工具集成
- 版本控制友好

### 基础配置

```toml
[memory]
backend = "markdown"

# Markdown 文件目录
markdown_dir = "~/.zeroclaw/memory/markdown"

# 文件格式
date_format = "%Y-%m-%d"  # 日期格式
file_prefix = "memory-"   # 文件前缀
file_suffix = ".md"       # 文件后缀
```

### 文件结构

```
~/.zeroclaw/memory/markdown/
├── memory-2026-02-21.md
├── memory-2026-02-20.md
└── memory-2026-02-19.md
```

### 文件格式

```markdown
---
date: 2026-02-21
tags: [memory, auto-saved]
---

# 2026-02-21 记忆

## 用户输入

> 帮我分析一下 ZeroClaw 的架构

## 关键信息

- ZeroClaw 使用 Rust 编写
- 支持 Trait 驱动架构
- 内存占用 <5MB
```

---

## 无记忆模式

### 配置

```toml
[memory]
backend = "none"
```

### 适用场景

- 临时会话
- 隐私敏感场景
- 测试和调试

---

## 自动保存配置

### 基础配置

```toml
[memory]
# 启用自动保存
auto_save = true

# 仅保存用户输入（不保存助手响应）
# 这是默认行为，防止模型生成的内容被当作事实
```

### 保存策略

| 内容类型 | 是否保存 | 说明 |
|----------|----------|------|
| 用户输入 | ✅ | 用户明确陈述的信息 |
| 助手响应 | ❌ | 防止模型幻觉污染记忆 |
| 工具结果 | ✅ | 事实性数据 |
| 文件内容 | ❌ | 可能包含敏感信息 |

### 手动保存

```bash
# 通过 Agent 命令手动保存
zeroclaw agent -m "记住这个：我的生日是 3 月 15 日"
```

---

## 混合搜索配置

### 权重配置

```toml
[memory]
# 向量搜索权重（0.0 - 1.0）
vector_weight = 0.7

# 关键词搜索权重（0.0 - 1.0）
keyword_weight = 0.3

# 权重和应为 1.0
# vector_weight + keyword_weight = 1.0
```

### 权重调优

| 场景 | vector_weight | keyword_weight |
|------|---------------|----------------|
| 语义搜索为主 | 0.8-0.9 | 0.1-0.2 |
| 平衡搜索 | 0.7 | 0.3 |
| 关键词搜索为主 | 0.3-0.5 | 0.5-0.7 |
| 仅向量搜索 | 1.0 | 0.0 |
| 仅关键词搜索 | 0.0 | 1.0 |

### 搜索配置

```toml
[memory]
# 最大搜索结果
max_results = 10

# 最小相似度阈值
min_similarity = 0.5

# 最大 RAG 块数
rag_chunk_limit = 5
```

---

## 成本追踪配置

### 基础配置

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

# 成本追踪文件
cost_file = "~/.zeroclaw/cost.json"
```

### 嵌入成本追踪

```toml
[cost]
enabled = true

# 嵌入成本（每 1000 tokens）
embedding_cost_per_1k = 0.0001  # OpenAI text-embedding-3-small

# 搜索成本（每次）
search_cost_per_query = 0.001
```

### 成本报告

```bash
# 查看成本统计
zeroclaw status --verbose

# 导出成本报告
# 查看 ~/.zeroclaw/cost.json
```

---

## 记忆维护

### 重新索引

```bash
# 重建 FTS5 索引
zeroclaw memory rebuild

# 重新嵌入（添加新向量）
zeroclaw memory reindex

# 预览更改
zeroclaw memory reindex --dry-run
```

### 清理记忆

```bash
# 清理旧记忆（保留最近 30 天）
zeroclaw memory cleanup --retain-days 30

# 清理特定标签的记忆
zeroclaw memory cleanup --tag "temp"

# 预览删除
zeroclaw memory cleanup --dry-run
```

### 导出记忆

```bash
# 导出为 JSON
zeroclaw memory export --format json > memory.json

# 导出为 Markdown
zeroclaw memory export --format markdown > memory.md

# 导出特定日期范围
zeroclaw memory export --from 2026-01-01 --to 2026-02-21
```

### 导入记忆

```bash
# 从 JSON 导入
zeroclaw memory import --format json memory.json

# 从 Markdown 导入
zeroclaw memory import --format markdown memory.md
```

---

## 调试和监控

### 诊断命令

```bash
# 查看记忆状态
zeroclaw memory status

# 查看记忆统计
zeroclaw memory stats

# 测试搜索
zeroclaw memory search "测试查询"

# 查看嵌入缓存
zeroclaw memory cache stats
```

### 日志分析

```bash
# 启用详细日志
RUST_LOG=debug zeroclaw daemon

# 查看记忆操作日志
grep "memory" ~/.zeroclaw/zeroclaw.log

# 查看嵌入日志
grep "embedding" ~/.zeroclaw/zeroclaw.log

# 查看搜索日志
grep "search" ~/.zeroclaw/zeroclaw.log
```

---

## 最佳实践

### 1. 选择合适的后端

```toml
# ✅ 个人使用：SQLite
[memory]
backend = "sqlite"

# ✅ 企业部署：PostgreSQL
[storage.provider.config]
provider = "postgres"

# ✅ 人工审查：Markdown
[memory]
backend = "markdown"
```

### 2. 配置嵌入

```toml
# ✅ 推荐：使用嵌入增强搜索
[memory]
embedding_provider = "openai"
embedding_model = "text-embedding-3-small"
vector_weight = 0.7
keyword_weight = 0.3

# ❌ 避免：无嵌入（搜索质量差）
[memory]
embedding_provider = "none"
```

### 3. 定期维护

```bash
# 每周清理旧记忆
zeroclaw memory cleanup --retain-days 30

# 每月导出备份
zeroclaw memory export --format json > backup-$(date +%Y-%m).json
```

---

## 相关文档

- [[ZeroClaw-核心配置详解]] - 基础配置
- [[ZeroClaw-完整配置指南]] - 配置总览
- [[ZeroClaw-成本追踪配置]] - 成本管理

---

_最后更新：2026-02-21_
