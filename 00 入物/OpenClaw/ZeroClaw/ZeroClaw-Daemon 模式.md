---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, daemon, background, service, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 守护进程，ZeroClaw 后台服务]
---

# ZeroClaw-Daemon 模式

> [!summary] 核心概念
> ZeroClaw Daemon 模式将 AI 助手作为后台守护进程运行，支持 webhook 接收消息、定时任务、自动响应等功能，适合生产环境部署。

## 启动守护进程

### 基本启动

```bash
# 前台运行（调试用）
zeroclaw daemon

# 后台运行
zeroclaw daemon --background

# 后台运行并指定 PID 文件
zeroclaw daemon --background --pid-file /var/run/zeroclaw.pid
```

### 配置选项

```bash
# 指定配置文件
zeroclaw daemon --config ~/.zeroclaw/prod-config.toml

# 指定工作目录
zeroclaw daemon --workdir /opt/zeroclaw/workspace

# 设置日志级别
zeroclaw daemon --log-level debug

# 指定日志文件
zeroclaw daemon --log-file /var/log/zeroclaw/daemon.log
```

## 系统服务集成

### systemd（现代 Linux）

**创建服务文件：**
```bash
sudo tee /etc/systemd/system/zeroclaw.service > /dev/null <<'EOF'
[Unit]
Description=ZeroClaw AI Assistant Daemon
After=network.target

[Service]
Type=simple
User=zeroclaw
Group=zeroclaw
WorkingDirectory=/opt/zeroclaw
ExecStart=/usr/local/bin/zeroclaw daemon --config /opt/zeroclaw/config.toml
Restart=on-failure
RestartSec=10
Environment=RUST_LOG=info

# 安全加固
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=read-only
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

**管理服务：**
```bash
# 重新加载 systemd 配置
sudo systemctl daemon-reload

# 启用开机自启
sudo systemctl enable zeroclaw

# 启动服务
sudo systemctl start zeroclaw

# 查看状态
sudo systemctl status zeroclaw

# 查看日志
sudo journalctl -u zeroclaw -f

# 重启服务
sudo systemctl restart zeroclaw

# 停止服务
sudo systemctl stop zeroclaw
```

### OpenRC（Alpine Linux）

**创建服务脚本：**
```bash
sudo tee /etc/init.d/zeroclaw > /dev/null <<'EOF'
#!/sbin/openrc-run

description="ZeroClaw AI Assistant Daemon"
command="/usr/local/bin/zeroclaw"
command_args="daemon --config /etc/zeroclaw/config.toml"
command_background=true
pidfile="/var/run/zeroclaw.pid"
logfile="/var/log/zeroclaw/daemon.log"

depend() {
    need net
    use logger
}

start_pre() {
    checkpath --directory --owner zeroclaw:zeroclaw --mode 0755 /var/log/zeroclaw
}
EOF

sudo chmod +x /etc/init.d/zeroclaw
```

**管理服务：**
```bash
# 启用开机自启
sudo rc-update add zeroclaw default

# 启动服务
sudo service zeroclaw start

# 查看状态
sudo service zeroclaw status

# 查看日志
sudo tail -f /var/log/zeroclaw/daemon.log
```

## 网关配置

### HTTP Webhook

```toml
# ~/.zeroclaw/config.toml

[gateway]
enabled = true
host = "127.0.0.1"  # 仅本地访问（安全）
# host = "0.0.0.0"  # 所有接口（需防火墙）
port = 3000

# HTTPS 配置（可选）
# tls_cert = "/etc/ssl/certs/zeroclaw.crt"
# tls_key = "/etc/ssl/private/zeroclaw.key"

# 认证配置
auth_token = "your-secret-token"

# 速率限制
rate_limit = 100  # 每分钟请求数
```

### 启动网关

```bash
# 仅启动网关（不启动完整 daemon）
zeroclaw gateway

# 指定端口
zeroclaw gateway --port 8080

# 随机端口（安全）
zeroclaw gateway --port 0

# 绑定到特定接口
zeroclaw gateway --host 192.168.1.100
```

## 通道配置

### Telegram

```toml
# ~/.zeroclaw/config.toml

[channel.telegram]
enabled = true
bot_token = "BOT_TOKEN_FROM_BOTFATHER"
allowed_users = ["123456789", "987654321"]  # Telegram 用户 ID

# 可选配置
webhook_url = "https://your-domain.com/webhook/telegram"
max_message_length = 4096
```

**获取 Telegram 用户 ID：**
```bash
# 发送消息给 @userinfobot
# 或使用 zeroclaw 命令
zeroclaw channel bind-telegram 123456789
```

### Discord

```toml
[channel.discord]
enabled = true
bot_token = "DISCORD_BOT_TOKEN"
allowed_channels = ["channel_id_1", "channel_id_2"]
allowed_guilds = ["guild_id_1"]

# 可选配置
prefix = "!"  # 命令前缀
```

### Slack

```toml
[channel.slack]
enabled = true
bot_token = "xoxb-YOUR-BOT-TOKEN"
allowed_channels = ["C0123456789"]
```

## 定时任务

### Cron 配置

```toml
# ~/.zeroclaw/config.toml

[[cron.jobs]]
name = "每日新闻推送"
schedule = "0 9 * * *"  # 每天 9:00
enabled = true
message = "获取今日技术新闻并推送给用户"
provider = "anthropic"
model = "claude-sonnet-4-5-20250929"

[[cron.jobs]]
name = "系统健康检查"
schedule = "0 3 * * *"  # 每天 3:00
enabled = true
message = "检查系统状态：CPU、内存、磁盘、服务状态"
tools = ["shell"]

[[cron.jobs]]
name = "备份提醒"
schedule = "0 18 * * 5"  # 每周五 18:00
enabled = true
message = "提醒用户进行数据备份"
```

### 管理定时任务

```bash
# 列出所有任务
zeroclaw cron list

# 查看任务状态
zeroclaw cron status

# 禁用任务
zeroclaw cron disable "每日新闻推送"

# 启用任务
zeroclaw cron enable "每日新闻推送"

# 手动运行任务
zeroclaw cron run "每日新闻推送"

# 查看运行历史
zeroclaw cron runs --id "任务 ID"
```

## 监控和日志

### 日志配置

```toml
# ~/.zeroclaw/config.toml

[logging]
level = "info"  # debug, info, warn, error
file = "/var/log/zeroclaw/daemon.log"
max_size = "100MB"
max_files = 5
rotate = true
```

### 查看日志

```bash
# 实时查看日志
tail -f /var/log/zeroclaw/daemon.log

# 查看最近 100 行
tail -n 100 /var/log/zeroclaw/daemon.log

# 搜索错误
grep -i error /var/log/zeroclaw/daemon.log

# 使用 journalctl（systemd）
journalctl -u zeroclaw -f
journalctl -u zeroclaw --since "2026-02-21 00:00:00"
```

### 健康检查

```bash
# 检查守护进程状态
zeroclaw status

# 检查网关健康
curl http://127.0.0.1:3000/health

# 检查通道健康
zeroclaw channel doctor

# 完整诊断
zeroclaw doctor
```

## 安全配置

### 网络隔离

```toml
# 仅绑定本地接口（推荐）
[gateway]
host = "127.0.0.1"
port = 3000

# 如需远程访问，使用反向代理
# host = "127.0.0.1"  # 仍然只绑定本地
```

### 防火墙配置

```bash
# UFW（Ubuntu）
sudo ufw allow from 192.168.1.0/24 to any port 3000

# firewalld（Fedora/RHEL）
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port port="3000" protocol="tcp" accept'

# iptables
sudo iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 3000 -j ACCEPT
```

### 认证令牌

```toml
[gateway]
auth_token = "使用强随机令牌"

# 生成安全令牌
openssl rand -hex 32
```

## 性能调优

### 资源限制

```bash
# 使用 systemd 限制资源
# /etc/systemd/system/zeroclaw.service

[Service]
MemoryLimit=512M
CPUQuota=50%
```

### 连接池配置

```toml
[gateway]
max_connections = 100
connection_timeout = 30
read_timeout = 60
write_timeout = 60
```

## 高可用部署

### 多实例配置

```bash
# 实例 1（主）
zeroclaw daemon --config /etc/zeroclaw/primary.toml

# 实例 2（备）
zeroclaw daemon --config /etc/zeroclaw/standby.toml
```

### 负载均衡

```nginx
# nginx 配置
upstream zeroclaw {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    listen 443 ssl;
    server_name zeroclaw.example.com;
    
    location / {
        proxy_pass http://zeroclaw;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 故障排除

### 问题：守护进程无法启动

```bash
# 检查配置文件
zeroclaw config show

# 检查端口占用
sudo lsof -i :3000

# 查看详细日志
zeroclaw daemon --log-level debug
```

### 问题：Webhook 不接收消息

```bash
# 检查网关状态
curl http://127.0.0.1:3000/health

# 检查防火墙
sudo ufw status

# 测试 webhook
curl -X POST http://127.0.0.1:3000/webhook/telegram \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'
```

### 问题：通道无响应

```bash
# 检查通道配置
zeroclaw channel list

# 重新绑定通道
zeroclaw channel bind-telegram <user_id>

# 检查认证
zeroclaw auth status
```

## 下一步

- [[ZeroClaw-Agent 模式]] - 交互式 Agent 使用
- [[ZeroClaw-常用命令]] - CLI 命令速查
- [[ZeroClaw-沙箱运行时]] - Docker 沙箱配置

---

_最后更新：2026-02-21_
