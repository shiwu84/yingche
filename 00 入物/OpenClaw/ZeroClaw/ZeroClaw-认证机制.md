---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, authentication, oauth, api-key, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 认证，ZeroClaw OAuth]
---

# ZeroClaw-认证机制

> [!summary] 核心概念
> ZeroClaw 支持多种认证方式，包括 API Key、OAuth（OpenAI Codex）和订阅令牌（Anthropic）。认证配置文件加密存储在 `~/.zeroclaw/auth-profiles.json`。

## 认证存储

| 文件 | 描述 |
|------|------|
| `~/.zeroclaw/auth-profiles.json` | 认证配置文件（加密存储） |
| `~/.zeroclaw/.secret_key` | 加密密钥 |

## OpenAI Codex OAuth

### 设备代码认证（推荐服务器/无头环境）

```bash
zeroclaw auth login --provider openai-codex --device-code
```

**流程：**
1. 运行命令获取设备代码
2. 在浏览器访问验证 URL
3. 输入设备代码
4. 授权 ZeroClaw 访问

### 浏览器回调认证

```bash
# 使用默认配置
zeroclaw auth login --provider openai-codex --profile default

# 使用粘贴重定向（无浏览器环境）
zeroclaw auth paste-redirect --provider openai-codex --profile default
```

### 管理认证配置

```bash
# 查看认证状态
zeroclaw auth status

# 刷新令牌
zeroclaw auth refresh --provider openai-codex --profile default

# 切换配置
zeroclaw auth use --provider openai-codex --profile work
```

### 使用认证运行 Agent

```bash
# 使用默认配置
zeroclaw agent --provider openai-codex -m "hello"

# 指定配置
zeroclaw agent --provider openai-codex --auth-profile openai-codex:work -m "hello"
```

## Anthropic 认证

### 使用订阅/设置令牌

```bash
# 粘贴订阅令牌（Authorization header 模式）
zeroclaw auth paste-token --provider anthropic --profile default --auth-kind authorization

# 别名命令
zeroclaw auth setup-token --provider anthropic --profile default
```

### 环境变量方式

```bash
# 使用 API Key
export ANTHROPIC_API_KEY="sk-ant-..."

# 使用认证令牌
export ANTHROPIC_AUTH="..."
```

### 运行 Agent

```bash
# 使用 API Key
zeroclaw agent --provider anthropic -m "hello"

# 使用认证令牌
zeroclaw agent --provider anthropic --auth-profile anthropic:default -m "hello"
```

## 多账户管理

### 配置文件 ID 格式

```
<provider>:<profile-name>
```

**示例：**
- `openai-codex:default`
- `openai-codex:work`
- `anthropic:personal`
- `anthropic:work`

### 创建多配置

```bash
# 创建第一个配置
zeroclaw auth login --provider openai-codex --profile default

# 创建工作配置
zeroclaw auth login --provider openai-codex --profile work

# 查看配置列表
zeroclaw auth status --list
```

### 切换配置

```bash
# 切换到工作配置
zeroclaw auth use --provider openai-codex --profile work

# 临时使用特定配置运行
zeroclaw agent --provider openai-codex --auth-profile openai-codex:work -m "task"
```

## 安全最佳实践

### 1. 加密存储

认证信息自动加密存储：
- 加密算法：AES-256-GCM
- 密钥存储：`~/.zeroclaw/.secret_key`
- 文件权限：600（仅所有者可读）

### 2. 权限控制

```bash
# 确保认证文件权限正确
chmod 600 ~/.zeroclaw/auth-profiles.json
chmod 600 ~/.zeroclaw/.secret_key
```

### 3. 定期刷新

OAuth 令牌会过期，建议定期刷新：

```bash
# 刷新所有配置
zeroclaw auth refresh --all

# 刷新特定配置
zeroclaw auth refresh --provider openai-codex --profile default
```

### 4. 撤销访问

如需撤销 ZeroClaw 的访问权限：

1. 访问提供者控制台
2. 找到已授权应用
3. 撤销 ZeroClaw 权限
4. 本地删除配置：
```bash
rm ~/.zeroclaw/auth-profiles.json
rm ~/.zeroclaw/.secret_key
```

## 认证故障排除

### 问题：OAuth 令牌过期

```bash
# 刷新令牌
zeroclaw auth refresh --provider openai-codex --profile default

# 重新认证
zeroclaw auth login --provider openai-codex --profile default
```

### 问题：认证文件损坏

```bash
# 备份并重新创建
mv ~/.zeroclaw/auth-profiles.json ~/.zeroclaw/auth-profiles.json.bak
zeroclaw auth login --provider openai-codex --profile default
```

### 问题：加密密钥丢失

```bash
# 删除并重新生成
rm ~/.zeroclaw/.secret_key
rm ~/.zeroclaw/auth-profiles.json
zeroclaw auth login --provider openai-codex --profile default
```

## 支持的服务

| 服务 | 认证方式 | 状态 |
|------|----------|------|
| OpenAI Codex | OAuth | ✅ |
| Anthropic | API Key / 令牌 | ✅ |
| OpenRouter | API Key | ✅ |
| Ollama | 无（本地） | ✅ |
| LM Studio | 无（本地） | ✅ |

## 重要通知

> [!warning] Anthropic 认证更新 (2026-02-19)
> Anthropic 更新了认证和凭证使用条款。OAuth 认证（Free、Pro、Max）仅适用于 Claude Code 和 Claude.ai。
> 
> 暂时避免使用 Claude Code OAuth 集成，以防止潜在的服务条款违规。

## 下一步

- [[ZeroClaw-快速开始]] - 安装和初始化
- [[ZeroClaw-提供者配置]] - 配置不同 AI 提供者
- [[ZeroClaw-通道配置]] - 配置消息通道

---

_最后更新：2026-02-21_
