---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, tools, integration, development, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 工具开发，ZeroClaw 自定义工具]
---

# ZeroClaw-工具集成

> [!summary] 核心概念
> ZeroClaw 通过 Rust Trait 系统支持自定义工具扩展，开发者可以实现任意功能作为 Agent 可调用的工具，包括 API 集成、数据库操作、外部服务调用等。

## 内置工具

### 文件操作

| 工具 | 描述 | 示例 |
|------|------|------|
| `file_read` | 读取文件内容 | 读取配置文件、日志分析 |
| `file_write` | 写入文件内容 | 生成报告、创建文件 |
| `file_delete` | 删除文件 | 清理临时文件 |
| `file_list` | 列出目录内容 | 浏览文件结构 |

### 系统命令

| 工具 | 描述 | 示例 |
|------|------|------|
| `shell` | 执行 Shell 命令 | 系统管理、进程控制 |
| `process_info` | 查看进程信息 | 监控系统状态 |
| `network_info` | 网络信息查询 | 检查连接、端口 |

### 网络工具

| 工具 | 描述 | 示例 |
|------|------|------|
| `web_search` | 网络搜索 | 查找信息、研究 |
| `web_fetch` | 获取网页内容 | 抓取文章、文档 |
| `http_request` | 发送 HTTP 请求 | API 调用、Webhook |

## 自定义工具开发

### Tool Trait 定义

```rust
use zeroclaw::tools::{Tool, ToolInput, ToolOutput, ToolSchema};
use async_trait::async_trait;

#[derive(Debug, Clone)]
pub struct MyCustomTool {
    name: String,
    description: String,
}

#[async_trait]
impl Tool for MyCustomTool {
    fn name(&self) -> &str {
        &self.name
    }

    fn description(&self) -> &str {
        &self.description
    }

    fn schema(&self) -> ToolSchema {
        ToolSchema {
            name: self.name.clone(),
            description: self.description.clone(),
            input_schema: json!({
                "type": "object",
                "properties": {
                    "param1": {
                        "type": "string",
                        "description": "参数 1 描述"
                    },
                    "param2": {
                        "type": "integer",
                        "description": "参数 2 描述"
                    }
                },
                "required": ["param1"]
            }),
        }
    }

    async fn execute(&self, input: ToolInput) -> Result<ToolOutput> {
        // 工具实现逻辑
        let param1 = input.get("param1").as_str().unwrap();
        let param2 = input.get("param2").as_i64().unwrap_or(0);

        // 执行操作
        let result = self.do_something(param1, param2).await?;

        Ok(ToolOutput {
            success: true,
            data: json!({ "result": result }),
            message: Some("操作成功".to_string()),
        })
    }
}

impl MyCustomTool {
    pub fn new() -> Self {
        Self {
            name: "my_custom_tool".to_string(),
            description: "我的自定义工具描述".to_string(),
        }
    }

    async fn do_something(&self, param1: &str, param2: i64) -> Result<String> {
        // 实现具体逻辑
        Ok(format!("处理了 {} 和 {}", param1, param2))
    }
}
```

### 注册自定义工具

```rust
// src/main.rs 或工具注册模块
use zeroclaw::tools::ToolRegistry;
use crate::tools::MyCustomTool;

fn register_tools(registry: &mut ToolRegistry) {
    // 注册内置工具
    registry.register(Box::new(zeroclaw::tools::ShellTool::new()));
    registry.register(Box::new(zeroclaw::tools::FileReadTool::new()));
    registry.register(Box::new(zeroclaw::tools::FileWriteTool::new()));

    // 注册自定义工具
    registry.register(Box::new(MyCustomTool::new()));
}
```

## 实用工具示例

### 示例 1：数据库查询工具

```rust
use sqlx::{PgPool, Row};

pub struct DatabaseQueryTool {
    pool: PgPool,
}

#[async_trait]
impl Tool for DatabaseQueryTool {
    fn schema(&self) -> ToolSchema {
        ToolSchema {
            name: "database_query".to_string(),
            description: "执行 SQL 查询".to_string(),
            input_schema: json!({
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "SQL 查询语句"
                    },
                    "params": {
                        "type": "array",
                        "items": {"type": "string"},
                        "description": "查询参数"
                    }
                },
                "required": ["query"]
            }),
        }
    }

    async fn execute(&self, input: ToolInput) -> Result<ToolOutput> {
        let query = input.get("query").as_str().ok_or("Missing query")?;
        
        let rows = sqlx::query(query)
            .fetch_all(&self.pool)
            .await?;

        let results: Vec<serde_json::Value> = rows
            .iter()
            .map(|row| {
                json!({
                    "id": row.get::<i64, _>("id"),
                    "name": row.get::<String, _>("name"),
                })
            })
            .collect();

        Ok(ToolOutput {
            success: true,
            data: json!({ "rows": results }),
            message: Some(format!("查询返回 {} 行", results.len())),
        })
    }
}
```

### 示例 2：API 调用工具

```rust
use reqwest::Client;

pub struct ApiCallTool {
    client: Client,
    base_url: String,
    api_key: String,
}

#[async_trait]
impl Tool for ApiCallTool {
    fn schema(&self) -> ToolSchema {
        ToolSchema {
            name: "api_call".to_string(),
            description: "调用外部 API".to_string(),
            input_schema: json!({
                "type": "object",
                "properties": {
                    "endpoint": {
                        "type": "string",
                        "description": "API 端点"
                    },
                    "method": {
                        "type": "string",
                        "enum": ["GET", "POST", "PUT", "DELETE"]
                    },
                    "body": {
                        "type": "object",
                        "description": "请求体"
                    }
                },
                "required": ["endpoint", "method"]
            }),
        }
    }

    async fn execute(&self, input: ToolInput) -> Result<ToolOutput> {
        let endpoint = input.get("endpoint").as_str().ok_or("Missing endpoint")?;
        let method = input.get("method").as_str().unwrap_or("GET");
        
        let url = format!("{}/{}", self.base_url, endpoint);
        let mut request = self.client.request(method.parse()?, &url);
        
        // 添加认证头
        request = request.header("Authorization", format!("Bearer {}", self.api_key));
        
        // 添加请求体
        if let Some(body) = input.get("body") {
            request = request.json(body);
        }
        
        let response = request.send().await?;
        let status = response.status();
        let body: serde_json::Value = response.json().await?;

        Ok(ToolOutput {
            success: status.is_success(),
            data: body,
            message: Some(format!("API 响应状态：{}", status)),
        })
    }
}
```

### 示例 3：Git 操作工具

```rust
use std::process::Command;

pub struct GitTool;

#[async_trait]
impl Tool for GitTool {
    fn schema(&self) -> ToolSchema {
        ToolSchema {
            name: "git_operation".to_string(),
            description: "执行 Git 操作".to_string(),
            input_schema: json!({
                "type": "object",
                "properties": {
                    "command": {
                        "type": "string",
                        "enum": ["status", "add", "commit", "push", "pull", "log"]
                    },
                    "args": {
                        "type": "array",
                        "items": {"type": "string"},
                        "description": "Git 命令参数"
                    },
                    "workdir": {
                        "type": "string",
                        "description": "工作目录"
                    }
                },
                "required": ["command"]
            }),
        }
    }

    async fn execute(&self, input: ToolInput) -> Result<ToolOutput> {
        let command = input.get("command").as_str().ok_or("Missing command")?;
        let args: Vec<&str> = input.get("args")
            .as_array()
            .map(|arr| arr.iter().filter_map(|v| v.as_str()).collect())
            .unwrap_or_default();
        let workdir = input.get("workdir").as_str().unwrap_or(".");

        let mut cmd = Command::new("git");
        cmd.arg(command).args(&args).current_dir(workdir);

        let output = cmd.output()?;
        
        Ok(ToolOutput {
            success: output.status.success(),
            data: json!({
                "stdout": String::from_utf8_lossy(&output.stdout),
                "stderr": String::from_utf8_lossy(&output.stderr),
                "status": output.status.code()
            }),
            message: Some(if output.status.success() {
                "Git 操作成功".to_string()
            } else {
                "Git 操作失败".to_string()
            }),
        })
    }
}
```

## 工具配置

### 启用/禁用工具

```toml
# ~/.zeroclaw/config.toml

[tools]
# 全局启用/禁用
enabled = true

# 启用特定工具
enabled_tools = ["shell", "file_read", "file_write", "database_query"]

# 禁用特定工具
disabled_tools = ["shell"]  # 安全考虑禁用 Shell

# 工具超时配置
[tools.timeout]
shell = 60  # Shell 命令 60 秒超时
http_request = 30  # HTTP 请求 30 秒超时
default = 120  # 默认 120 秒超时
```

### 工具权限

```toml
# 限制 Shell 命令
[tools.shell]
allowed_commands = ["ls", "cat", "grep", "find", "git"]
forbidden_commands = ["rm", "sudo", "chmod", "chown"]
allowed_paths = ["/home/user/workspace", "/tmp"]
```

## 工具测试

### 单元测试

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_my_custom_tool() {
        let tool = MyCustomTool::new();
        let input = ToolInput::from(json!({
            "param1": "test",
            "param2": 42
        }));

        let result = tool.execute(input).await.unwrap();
        
        assert!(result.success);
        assert!(result.data.get("result").is_some());
    }
}
```

### 手动测试

```bash
# 测试工具调用
zeroclaw agent -m "使用 my_custom_tool 处理数据" \
  --enable-tools my_custom_tool \
  --verbose
```

## 安全考虑

### 输入验证

```rust
async fn execute(&self, input: ToolInput) -> Result<ToolOutput> {
    // 验证输入
    let path = input.get("path").as_str().ok_or("Missing path")?;
    
    // 防止路径遍历攻击
    if path.contains("..") || path.starts_with("/") {
        return Err("Invalid path".into());
    }
    
    // 验证命令白名单
    let command = input.get("command").as_str().ok_or("Missing command")?;
    if !self.allowed_commands.contains(&command) {
        return Err("Command not allowed".into());
    }
    
    // ... 执行操作
}
```

### 输出过滤

```rust
// 过滤敏感信息
let mut output = self.execute_internal(input).await?;

// 移除密码、密钥等敏感字段
if let Some(data) = output.data.as_object_mut() {
    data.remove("password");
    data.remove("api_key");
    data.remove("secret");
}

Ok(output)
```

## 工具发布

### 创建工具包

```bash
# 工具目录结构
my-tool/
├── Cargo.toml
├── src/
│   └── lib.rs
├── README.md
└── examples/
    └── basic.rs
```

### 文档要求

- 工具名称和描述
- 输入参数 schema
- 输出格式说明
- 使用示例
- 安全注意事项

## 下一步

- [[ZeroClaw-Agent 模式]] - Agent 工具调用
- [[ZeroClaw-沙箱运行时]] - 工具执行安全
- [[ZeroClaw-架构设计]] - Trait 系统详解

---

_最后更新：2026-02-21_
