---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, agent, interactive, cli, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw Agent 模式，ZeroClaw 交互模式]
---

# ZeroClaw-Agent 模式

> [!summary] 核心概念
> ZeroClaw Agent 模式支持与 AI 助手进行交互式对话，支持单条消息、多轮对话、文件上下文、工具调用等功能。

## 基本使用

### 单条消息

```bash
# 发送单条消息并等待响应
zeroclaw agent -m "Hello, ZeroClaw!"

# 指定模型
zeroclaw agent -m "解释一下量子计算" --model "claude-sonnet-4-5-20250929"

# 指定提供者
zeroclaw agent -m "写一个 Python 脚本" --provider anthropic
```

### 交互式模式（REPL）

```bash
# 启动交互式会话
zeroclaw agent

# 启动时加载系统提示
zeroclaw agent --system-prompt "你是一个专业的 Linux 系统管理员"
```

**交互式会话示例：**
```
$ zeroclaw agent

🦀 ZeroClaw Agent (claude-sonnet-4-5-20250929)
类型 /help 查看命令，/exit 退出

> 你好，帮我分析一下系统负载
[Agent 思考中...]
当前系统负载正常，CPU 使用率 15%，内存使用率 42%...

> 能帮我查看一下磁盘空间吗？
[Agent 调用工具：shell df -h]
文件系统      容量  已用  可用  使用% 挂载点
/dev/sda1     500G  30G  470G   6%    /

> /exit
再见！
```

## 高级选项

### 文件上下文

```bash
# 附加单个文件
zeroclaw agent -m "分析这个日志文件" --file ./app.log

# 附加多个文件
zeroclaw agent -m "比较这两个配置文件" \
  --file ./config.dev.toml \
  --file ./config.prod.toml

# 附加目录（递归）
zeroclaw agent -m "总结这个项目" --file ./src/
```

**支持的文件类型：**
- 文本文件（.txt, .md, .json, .toml, .yaml）
- 代码文件（.py, .rs, .ts, .js, .go）
- 日志文件（.log）
- 配置文件（.*rc）

### 超时控制

```bash
# 设置最大思考时间（秒）
zeroclaw agent -m "复杂任务" --timeout 120

# 设置最大执行时间
zeroclaw agent -m "长时间任务" --execution-timeout 300
```

### 详细输出

```bash
# 启用详细模式（显示思考过程）
zeroclaw agent -m "任务" --verbose

# 显示工具调用详情
zeroclaw agent -m "任务" --show-tool-calls

# JSON 输出（便于脚本处理）
zeroclaw agent -m "任务" --output json
```

## 会话管理

### 会话持久化

```bash
# 指定会话 ID（继续之前的对话）
zeroclaw agent --session "abc123" -m "继续之前的话题"

# 创建新会话
zeroclaw agent --new-session -m "开始新话题"

# 列出会话
zeroclaw agent --list-sessions

# 删除会话
zeroclaw agent --delete-session "abc123"
```

### 上下文窗口

```bash
# 限制上下文 token 数
zeroclaw agent -m "任务" --max-tokens 4096

# 保留最近 N 条消息
zeroclaw agent -m "任务" --max-messages 20
```

## 工具调用

### 启用/禁用工具

```bash
# 禁用所有工具（仅对话）
zeroclaw agent -m "任务" --no-tools

# 启用特定工具
zeroclaw agent -m "任务" --enable-tools shell,file_read

# 查看可用工具
zeroclaw agent --list-tools
```

### 工具调用示例

```bash
# 执行 Shell 命令
zeroclaw agent -m "查看当前目录下最大的 5 个文件" \
  --enable-tools shell

# 读取文件内容
zeroclaw agent -m "分析这个配置文件的问题" \
  --file ./config.toml \
  --enable-tools file_read

# 写入文件
zeroclaw agent -m "创建一个 README 文件" \
  --enable-tools file_write
```

## 交互式命令

在交互式模式中，可以使用以下命令：

| 命令 | 描述 |
|------|------|
| `/help` | 显示帮助信息 |
| `/exit` | 退出会话 |
| `/quit` | 退出会话（同 /exit） |
| `/clear` | 清除当前会话历史 |
| `/status` | 显示当前状态 |
| `/model` | 切换模型 |
| `/provider` | 切换提供者 |
| `/tools` | 列出可用工具 |
| `/session` | 显示会话信息 |
| `/history` | 显示对话历史 |
| `/retry` | 重试最后一条消息 |

## 编程模式

### 代码生成

```bash
# 生成代码并保存到文件
zeroclaw agent -m "写一个 Python 脚本，计算斐波那契数列" \
  --enable-tools file_write \
  --output-file fibonacci.py

# 代码审查
zeroclaw agent -m "审查这个代码的安全问题" \
  --file ./src/main.rs \
  --enable-tools file_read
```

### 批量处理

```bash
# 处理多个文件
for file in *.log; do
  zeroclaw agent -m "分析日志：$file" \
    --file "$file" \
    --output "analysis-${file%.log}.md"
done
```

## 管道和重定向

### 输入管道

```bash
# 从标准输入读取
echo "分析这段文本" | zeroclaw agent

# 从文件读取提示
cat prompt.txt | zeroclaw agent

# 组合命令
git diff HEAD~1 | zeroclaw agent -m "生成提交说明"
```

### 输出重定向

```bash
# 保存到文件
zeroclaw agent -m "生成报告" > report.md

# 追加到文件
zeroclaw agent -m "更新日志" >> changelog.md

# 同时显示和保存
zeroclaw agent -m "任务" | tee output.md
```

## 认证和配置

### 使用特定认证

```bash
# 使用工作配置
zeroclaw agent -m "任务" --auth-profile openai-codex:work

# 临时切换提供者
zeroclaw agent -m "任务" --provider anthropic --api-key "sk-ant-..."
```

### 环境变量

```bash
# 设置默认模型
export ZEROCLAW_MODEL="claude-sonnet-4-5-20250929"

# 设置默认提供者
export ZEROCLAW_PROVIDER="anthropic"

# 启用详细日志
export RUST_LOG=debug

# 使用自定义配置
export ZEROCLAW_CONFIG=~/.zeroclaw/work-config.toml
```

## 实用场景

### 场景 1：代码审查

```bash
zeroclaw agent \
  -m "请审查这个 Pull Request 的代码，关注：1. 安全问题 2. 性能问题 3. 代码风格" \
  --file ./src/ \
  --enable-tools file_read \
  --output review.md
```

### 场景 2：日志分析

```bash
zeroclaw agent \
  -m "分析这个日志文件，找出错误和警告，总结根本原因" \
  --file /var/log/app.log \
  --enable-tools file_read \
  --output error-summary.md
```

### 场景 3：文档生成

```bash
zeroclaw agent \
  -m "根据这个代码文件生成 API 文档" \
  --file ./src/api.rs \
  --enable-tools file_read,file_write \
  --output API.md
```

### 场景 4：系统诊断

```bash
zeroclaw agent \
  -m "诊断系统问题，检查：1. CPU 负载 2. 内存使用 3. 磁盘空间 4. 网络连接" \
  --enable-tools shell \
  --output diagnosis.md
```

## 性能优化

### 减少延迟

```bash
# 使用更快的模型
zeroclaw agent -m "简单任务" --model "haiku-3-5"

# 限制响应长度
zeroclaw agent -m "任务" --max-tokens 500

# 禁用不必要的工具
zeroclaw agent -m "简单对话" --no-tools
```

### 节省 Token

```bash
# 压缩上下文
zeroclaw agent -m "任务" --compress-context

# 限制历史消息
zeroclaw agent -m "任务" --max-messages 10
```

## 故障排除

### 问题：认证失败

```bash
# 检查认证状态
zeroclaw auth status

# 重新认证
zeroclaw auth login --provider anthropic
```

### 问题：工具调用失败

```bash
# 检查工具权限
zeroclaw agent --list-tools

# 查看详细错误
zeroclaw agent -m "任务" --verbose --enable-tools shell
```

### 问题：响应超时

```bash
# 增加超时时间
zeroclaw agent -m "复杂任务" --timeout 300

# 使用更快的模型
zeroclaw agent -m "任务" --model "haiku-3-5"
```

## 下一步

- [[ZeroClaw-常用命令]] - CLI 命令速查
- [[ZeroClaw-Daemon 模式]] - 后台守护进程配置
- [[ZeroClaw-工具集成]] - 自定义工具开发

---

_最后更新：2026-02-21_
