---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, channels, messaging, configuration, reference]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 通道配置，ZeroClaw Telegram Discord WhatsApp]
---

# ZeroClaw-通道配置详解

> [!summary] 本文档
> ZeroClaw 消息通道配置详解，涵盖 Telegram、Discord、WhatsApp、Nextcloud Talk 等通道的完整配置和使用指南。

---

## 通道概述

### 支持的通道

| 通道 | 状态 | 配置复杂度 | 需要 Bot |
|------|------|------------|----------|
| Telegram | ✅ | 简单 | ✅ |
| Discord | ✅ | 中等 | ✅ |
| Slack | ✅ | 中等 | ✅ |
| WhatsApp | ✅ | 复杂 | ✅ |
| Mattermost | ✅ | 简单 | ✅ |
| Nextcloud Talk | ✅ | 中等 | ✅ |
| Email | ✅ | 中等 | ❌ |
| IRC | ✅ | 简单 | ❌ |
| Matrix | 🚧 | 复杂 | ✅ |
| Signal | 🚧 | 复杂 | ❌ |
| Lark (飞书) | 🚧 | 中等 | ✅ |
| DingTalk (钉钉) | 🚧 | 中等 | ✅ |
| QQ | 🚧 | 复杂 | ✅ |

### 通道配置位置

```toml
# ~/.zeroclaw/config.toml

# 全局通道配置
[channels_config]
message_timeout_secs = 300

# 各通道独立配置
[channels_config.telegram]
[channels_config.discord]
[channels_config.whatsapp]
```

### 通道管理命令

```bash
# 列出所有通道
zeroclaw channel list

# 启动通道服务
zeroclaw channel start

# 检查通道健康
zeroclaw channel doctor

# 绑定 Telegram 用户
zeroclaw channel bind-telegram 123456789

# 添加通道
zeroclaw channel add telegram '{"bot_token": "..."}'

# 移除通道
zeroclaw channel remove telegram
```

---

## Telegram 配置

### 获取 Bot Token

1. 在 Telegram 搜索 `@BotFather`
2. 发送 `/newbot` 创建新 Bot
3. 设置 Bot 名称和用户名
4. 获取 Bot Token（格式：`123456789:ABCdefGHIjklMNOpqrsTUVwxyz`）

### 基础配置

```toml
[channels_config.telegram]
# 启用 Telegram
enabled = true

# Bot Token（从 @BotFather 获取）
bot_token = "123456789:ABCdefGHIjklMNOpqrsTUVwxyz"

# 允许的用户 ID（空数组 = 全部禁止）
allowed_users = ["123456789", "987654321"]

# 允许的群组 ID（可选）
allowed_groups = ["-1001234567890"]

# 消息超时（秒）
message_timeout_secs = 300

# 新消息中断处理
interrupt_on_new_message = false
```

### 获取用户 ID

**方法 1：使用 @userinfobot**
1. 在 Telegram 搜索 `@userinfobot`
2. 发送任意消息
3. Bot 会回复你的用户 ID

**方法 2：使用 ZeroClaw**
```bash
# 发送消息给 Bot，然后查看日志
zeroclaw daemon --log-level debug 2>&1 | grep "telegram"
```

**方法 3：使用 channel 命令**
```bash
# 绑定当前用户
zeroclaw channel bind-telegram <your_user_id>
```

### 高级配置

```toml
[channels_config.telegram]
enabled = true
bot_token = "YOUR_BOT_TOKEN"
allowed_users = ["123456789"]

# Webhook 模式（可选，默认使用长轮询）
# webhook_url = "https://your-domain.com/webhook/telegram"

# 最大消息长度
max_message_length = 4096

# 解析模式
parse_mode = "Markdown"  # Markdown | HTML

# 超时配置
timeout_secs = 30
read_timeout_secs = 30
write_timeout_secs = 30

# 重试配置
max_retries = 3
retry_delay_secs = 2

# 新消息中断（取消进行中的请求）
interrupt_on_new_message = false

# 回复引用
reply_to_message = true

# 发送打字状态
send_chat_action = true
```

### 群组支持

```toml
[channels_config.telegram]
enabled = true
bot_token = "YOUR_BOT_TOKEN"

# 允许的用户
allowed_users = ["123456789"]

# 允许的群组（需要将 Bot 添加到群组）
allowed_groups = [
    "-1001234567890",  # 超级群组
    "-123456789"       # 普通群组
]

# 仅响应提及（群组中）
reply_to_mentions_only = true

# 命令前缀（群组中）
command_prefix = "/"
```

### 内联命令

Telegram 支持内联命令（在聊天中输入）：

```
/models              # 查看可用模型
/models anthropic    # 查看特定提供者的模型
/model               # 查看当前模型
/model claude-sonnet # 切换模型
```

### 热配置更新

Telegram 通道运行时会自动热更新以下配置：
- `default_provider`
- `default_model`
- `default_temperature`
- `api_key`
- `api_url`
- `reliability.*` 重试配置

---

## Discord 配置

### 创建 Discord Bot

1. 访问 https://discord.com/developers/applications
2. 点击 "New Application"
3. 进入 "Bot" 页面
4. 点击 "Add Bot"
5. 复制 Bot Token
6. 启用 "Message Content Intent"

### 邀请 Bot 到服务器

1. 进入 "OAuth2" → "URL Generator"
2. 选择 scopes: `bot`
3. 选择 permissions:
   - Send Messages
   - Read Message History
   - Embed Links
4. 复制生成的 URL 并在浏览器打开
5. 选择服务器并授权

### 获取服务器和频道 ID

**启用开发者模式：**
1. Discord 设置 → 高级 → 开发者模式
2. 右键点击服务器/频道 → "复制 ID"

### 基础配置

```toml
[channels_config.discord]
# 启用 Discord
enabled = true

# Bot Token（从 Discord Developer Portal 获取）
bot_token = "MTIzNDU2Nzg5MDEyMzQ1Njc4OQ.ABCDEF.GHIjklMNOpqrsTUVwxyz123456"

# 允许的频道 ID（空数组 = 全部禁止）
allowed_channels = ["123456789012345678"]

# 允许的服务器 ID（可选）
allowed_guilds = ["987654321098765432"]

# 命令前缀
prefix = "!"

# 消息超时（秒）
message_timeout_secs = 300
```

### 高级配置

```toml
[channels_config.discord]
enabled = true
bot_token = "YOUR_BOT_TOKEN"
allowed_channels = ["123456789012345678"]
allowed_guilds = ["987654321098765432"]

# 命令前缀
prefix = "!"

# 最大消息长度
max_message_length = 2000

# 分块发送（超过最大长度时）
chunk_messages = true
chunk_size = 1900

# 嵌入消息
embed_messages = true

# 显示思考状态
show_typing = true

# 回复引用
reply_to_message = true

# 超时配置
timeout_secs = 30

# 重试配置
max_retries = 3
retry_delay_secs = 2
```

### 内联命令

```
!models              # 查看可用模型
!model               # 查看当前模型
!model claude-sonnet # 切换模型
!status              # 查看状态
```

---

## WhatsApp 配置

### 配置方式对比

WhatsApp 支持两种配置方式：

| 方式 | 优点 | 缺点 | 复杂度 |
|------|------|------|--------|
| Cloud API | 官方支持、稳定 | 需要 Meta 审核 | 中等 |
| WhatsApp Web | 无需审核、快速 | 非官方、可能被封 | 简单 |

### Cloud API 配置（推荐）

#### 准备工作

1. 访问 https://developers.facebook.com/
2. 创建 Meta 开发者账号
3. 创建应用（类型：Other → Business）
4. 添加 WhatsApp 产品
5. 创建测试号码或使用已验证号码

#### 获取凭证

1. **Access Token**: WhatsApp → API Setup → Temporary Access Token
2. **Phone Number ID**: WhatsApp → API Setup → Phone number ID
3. **App Secret**: 应用设置 → 基本设置 → App Secret

#### 基础配置

```toml
[channels_config.whatsapp]
# 启用 WhatsApp（Cloud API 模式）
enabled = true

# Meta Cloud API 访问令牌
access_token = "EAABsbCS1iHgBO..."

# 电话号码 ID
phone_number_id = "123456789012345"

# Webhook 验证令牌（自定义）
verify_token = "your-verify-token"

# 应用密钥（用于签名验证）
app_secret = "1234567890abcdef"

# 允许的号码（空数组 = 全部禁止，"*" = 允许所有）
allowed_numbers = ["+861234567890"]

# 消息超时（秒）
message_timeout_secs = 300
```

#### Webhook 配置

1. **设置 Webhook URL**:
   - WhatsApp → Configuration → Webhook
   - URL: `https://your-domain.com/webhook/whatsapp`
   - Verify Token: 与配置一致

2. **订阅字段**:
   - messages
   - message_reactions

3. **验证 Webhook**:
```bash
# ZeroClaw 会自动处理验证请求
# 确保网关可公开访问（使用隧道）
```

### WhatsApp Web 配置

#### 构建要求

```bash
# 需要启用 build flag
cargo build --features whatsapp-web
```

#### 基础配置

```toml
[channels_config.whatsapp]
# 启用 WhatsApp（Web 模式）
enabled = true

# 会话存储路径
session_path = "~/.zeroclaw/whatsapp-session"

# 配对手机号（可选）
pair_phone = "861234567890"

# 自定义配对码（可选，否则自动生成）
pair_code = "ABC123"

# 允许的号码
allowed_numbers = ["+861234567890"]

# 消息超时（秒）
message_timeout_secs = 300
```

#### 配对流程

1. 启动 ZeroClaw
2. 扫描终端显示的二维码
3. 或使用配对码：
   - 发送消息到指定手机号
   - 包含显示的配对码

---

## Nextcloud Talk 配置

### 准备工作

1. Nextcloud 服务器（版本 25+）
2. 安装 "Talk" 应用
3. 创建 Bot 用户

### 创建 Bot

1. Nextcloud → 设置 → 管理 → Talk
2. 添加 Bot
3. 记录 Bot Token 和 Webhook Secret

### 基础配置

```toml
[channels_config.nextcloud_talk]
# 启用 Nextcloud Talk
enabled = true

# Nextcloud 基础 URL
base_url = "https://cloud.example.com"

# Bot App Token
app_token = "YOUR_APP_TOKEN"

# Webhook Secret（可选，用于签名验证）
webhook_secret = "YOUR_SECRET"

# 允许的用户 ID（空数组 = 全部禁止）
allowed_users = ["admin", "user1", "user2"]

# 消息超时（秒）
message_timeout_secs = 300
```

### Webhook 配置

1. Nextcloud → Talk → Bot 设置
2. 添加 Webhook URL:
   - `https://your-domain.com/nextcloud-talk`
3. 设置 Secret（与配置一致）

### 环境变量覆盖

```bash
export ZEROCLAW_NEXTCLOUD_TALK_WEBHOOK_SECRET="secret"
```

---

## Slack 配置

### 创建 Slack App

1. 访问 https://api.slack.com/apps
2. 创建新 App
3. 添加 Bot User
4. 安装到工作区

### 权限配置

需要以下权限：
- `chat:write` - 发送消息
- `channels:read` - 读取频道
- `im:read` - 读取私信
- `im:history` - 读取历史消息

### 基础配置

```toml
[channels_config.slack]
# 启用 Slack
enabled = true

# Bot Token（以 xoxb- 开头）
bot_token = "xoxb-YOUR-BOT-TOKEN"

# 允许的频道 ID
allowed_channels = ["C0123456789"]

# 允许的服务器 ID
allowed_teams = ["T0123456789"]

# 消息超时（秒）
message_timeout_secs = 300
```

---

## 全局通道配置

### 超时配置

```toml
[channels_config]
# 基础消息超时（秒）
message_timeout_secs = 300

# 实际超时 = message_timeout_secs × scale
# scale = min(max_tool_iterations, 4)
# 最小值 = 30 秒
```

### 建议配置

| 场景 | message_timeout_secs |
|------|---------------------|
| 云 API（快速） | 60-120 |
| 本地模型（Ollama） | 300-600 |
| 复杂任务 | 300+ |

---

## 通道调试

### 诊断命令

```bash
# 查看通道状态
zeroclaw channel list

# 检查通道健康
zeroclaw channel doctor

# 查看通道详情
zeroclaw integrations info Telegram
zeroclaw integrations info Discord
```

### 日志分析

```bash
# 启用详细日志
RUST_LOG=debug zeroclaw daemon

# 查看 Telegram 日志
grep "telegram" ~/.zeroclaw/zeroclaw.log

# 查看 Discord 日志
grep "discord" ~/.zeroclaw/zeroclaw.log

# 查看错误
grep -i "error.*channel" ~/.zeroclaw/zeroclaw.log
```

### 常见问题

#### 问题 1：Telegram 无响应

```bash
# 验证 Bot Token
curl -X GET "https://api.telegram.org/bot<TOKEN>/getMe"

# 检查用户 ID 是否在允许列表
zeroclaw channel list

# 重新绑定用户
zeroclaw channel bind-telegram <user_id>
```

#### 问题 2：Discord 连接失败

```bash
# 验证 Bot Token
curl -X GET https://discord.com/api/users/@me \
  -H "Authorization: Bot <TOKEN>"

# 检查 Intent 设置
# Discord Developer Portal → Bot → Privileged Gateway Intents
# 启用 "Message Content Intent"
```

#### 问题 3：WhatsApp Webhook 失败

```bash
# 检查 Webhook URL 可访问性
curl -X GET "https://your-domain.com/webhook/whatsapp?hub.mode=subscribe&hub.verify_token=YOUR_TOKEN"

# 检查隧道状态
zeroclaw status

# 验证签名
grep "webhook" ~/.zeroclaw/zeroclaw.log
```

---

## 相关文档

- [[ZeroClaw-核心配置详解]] - 基础配置
- [[ZeroClaw-完整配置指南]] - 配置总览
- [[ZeroClaw-故障排除]] - 常见问题解决

---

_最后更新：2026-02-21_
