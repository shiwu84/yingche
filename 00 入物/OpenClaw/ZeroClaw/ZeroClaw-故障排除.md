---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, troubleshooting, debugging, faq, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 故障排除，ZeroClaw 常见问题]
---

# ZeroClaw-故障排除

> [!summary] 核心概念
> 本笔记汇总 ZeroClaw 使用中的常见问题和解决方案，涵盖安装、认证、通道配置、工具执行、性能等各类问题。

## 诊断工具

### 内置诊断

```bash
# 完整系统诊断
zeroclaw doctor

# 检查状态
zeroclaw status

# 深度状态检查
zeroclaw status --deep

# 检查通道健康
zeroclaw channel doctor

# 检查认证状态
zeroclaw auth status
```

### 日志查看

```bash
# 实时查看日志
tail -f ~/.zeroclaw/zeroclaw.log

# 查看最近错误
grep -i error ~/.zeroclaw/zeroclaw.log | tail -20

# 查看特定时间日志
journalctl -u zeroclaw --since "2026-02-21 08:00:00" --until "2026-02-21 10:00:00"

# 启用调试日志
RUST_LOG=debug zeroclaw daemon
```

## 安装问题

### 问题 1：bootstrap.sh 执行失败

**症状：**
```
Permission denied: ./bootstrap.sh
```

**解决方案：**
```bash
# 添加执行权限
chmod +x bootstrap.sh
./bootstrap.sh

# 或直接使用 bash
bash bootstrap.sh
```

### 问题 2：Rust 安装失败

**症状：**
```
error: could not download rustup
```

**解决方案：**
```bash
# 使用国内镜像
export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
curl --proto '=https' --tlsv1.2 -sSf https://mirrors.ustc.edu.cn/misc/rustup-install.sh | sh

# 或手动下载
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --default-toolchain stable
```

### 问题 3：编译时内存不足

**症状：**
```
error: out of memory
fatal error: killed signal
```

**解决方案：**
```bash
# 使用预编译二进制
./bootstrap.sh --prefer-prebuilt

# 或增加交换空间
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 或使用更少的 codegen 单位
CARGO_CODEGEN_UNITS=1 cargo build --release
```

### 问题 4：依赖缺失

**症状：**
```
error: linker `cc` not found
package `pkg-config` not found
```

**解决方案：**
```bash
# Debian/Ubuntu
sudo apt install build-essential pkg-config libssl-dev

# Fedora/RHEL
sudo dnf group install "Development Tools"
sudo dnf install pkg-config openssl-devel

# Alpine
sudo apk add build-base pkgconfig openssl-dev

# macOS
xcode-select --install
brew install pkg-config openssl
```

## 认证问题

### 问题 1：OAuth 认证失败

**症状：**
```
Error: OAuth token exchange failed
```

**解决方案：**
```bash
# 检查网络连接
curl -I https://auth.openai.com

# 清除旧认证
rm ~/.zeroclaw/auth-profiles.json
rm ~/.zeroclaw/.secret_key

# 重新认证
zeroclaw auth login --provider openai-codex --device-code

# 检查系统时间（时间不同步会导致认证失败）
timedatectl status
sudo timedatectl set-ntp true
```

### 问题 2：API Key 无效

**症状：**
```
Error: Invalid API key
Error: 401 Unauthorized
```

**解决方案：**
```bash
# 验证 API Key 格式
# OpenAI: sk-... (32+ 字符)
# Anthropic: sk-ant-... (40+ 字符)

# 测试 API Key
curl -X POST https://api.anthropic.com/v1/messages \
  -H "Authorization: Bearer $ANTHROPIC_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-3-haiku-20240307","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'

# 重新配置
zeroclaw onboard --api-key "sk-..." --provider anthropic --force
```

### 问题 3：认证过期

**症状：**
```
Error: Token expired
Error: Session expired
```

**解决方案：**
```bash
# 刷新令牌
zeroclaw auth refresh --provider openai-codex --profile default

# 检查令牌过期时间
zeroclaw auth status --verbose

# 重新认证
zeroclaw auth login --provider openai-codex --profile default
```

## 通道问题

### 问题 1：Telegram 无响应

**症状：**
- 发送消息给机器人无响应
- 日志显示 webhook 接收失败

**解决方案：**
```bash
# 检查 Bot Token
curl -X GET "https://api.telegram.org/bot<BOT_TOKEN>/getMe"

# 检查用户 ID 是否在允许列表
zeroclaw channel list

# 重新绑定用户
zeroclaw channel bind-telegram <your_user_id>

# 检查网关状态
curl http://127.0.0.1:3000/health

# 查看 webhook 日志
grep telegram ~/.zeroclaw/zeroclaw.log
```

### 问题 2：Discord 连接失败

**症状：**
```
Error: Discord gateway connection failed
```

**解决方案：**
```bash
# 验证 Bot Token
curl -X GET https://discord.com/api/users/@me \
  -H "Authorization: Bot <BOT_TOKEN>"

# 检查 Bot 权限
# 确保 Bot 已邀请到服务器并有发送消息权限

# 检查通道 ID
zeroclaw channel list --verbose

# 重新配置
zeroclaw onboard --channels-only
```

### 问题 3：消息发送失败

**症状：**
```
Error: Failed to send message
Error: Rate limited
```

**解决方案：**
```bash
# 检查速率限制
# Telegram: 30 消息/秒
# Discord: 5 消息/5 秒

# 查看错误详情
zeroclaw agent -m "test" --verbose

# 临时禁用速率限制（不推荐生产环境）
# 在配置文件中调整
[channel.telegram]
rate_limit = 60  # 提高到 60/分钟
```

## 工具执行问题

### 问题 1：Shell 命令执行失败

**症状：**
```
Error: Command not found
Error: Permission denied
```

**解决方案：**
```bash
# 检查命令是否在 PATH 中
which ls
echo $PATH

# 检查工具配置
zeroclaw config show | grep -A 10 tools.shell

# 添加允许的命令
# ~/.zeroclaw/config.toml
[tools.shell]
allowed_commands = ["ls", "cat", "grep", "find", "git", "pwd"]

# 检查沙箱配置
zeroclaw daemon --log-level debug | grep sandbox
```

### 问题 2：文件读取失败

**症状：**
```
Error: Permission denied
Error: No such file or directory
```

**解决方案：**
```bash
# 检查文件权限
ls -la /path/to/file

# 检查允许的路径
# ~/.zeroclaw/config.toml
[tools.file_read]
allowed_paths = ["/home/user/workspace", "/tmp"]

# 使用绝对路径
zeroclaw agent -m "读取文件" --file /absolute/path/to/file
```

### 问题 3：Docker 沙箱启动失败

**症状：**
```
Error: Cannot connect to Docker daemon
Error: Permission denied
```

**解决方案：**
```bash
# 检查 Docker 服务
sudo systemctl status docker

# 检查用户权限
groups  # 确保包含 docker

# 添加用户到 docker 组
sudo usermod -aG docker $USER
newgrp docker  # 或重新登录

# 测试 Docker
docker run hello-world

# 检查镜像
docker images | grep zeroclaw

# 拉取镜像
docker pull zeroclaw/sandbox:latest
```

## 性能问题

### 问题 1：响应慢

**症状：**
- Agent 响应时间超过 30 秒
- 工具调用延迟高

**解决方案：**
```bash
# 检查模型选择
# 使用更快的模型
zeroclaw agent -m "任务" --model "haiku-3-5"

# 限制上下文
zeroclaw agent -m "任务" --max-tokens 2048 --max-messages 10

# 检查网络延迟
ping api.anthropic.com
curl -w "@curl-format.txt" -o /dev/null -s https://api.anthropic.com/v1/messages

# 启用本地缓存
# ~/.zeroclaw/config.toml
[cache]
enabled = true
ttl = 3600  # 1 小时
```

### 问题 2：内存占用高

**症状：**
```bash
# 检查内存使用
ps aux | grep zeroclaw
free -h
```

**解决方案：**
```bash
# 限制 Daemon 内存
# /etc/systemd/system/zeroclaw.service
[Service]
MemoryLimit=512M

# 重启服务
sudo systemctl daemon-reload
sudo systemctl restart zeroclaw

# 检查内存泄漏
zeroclaw status --verbose
```

### 问题 3：CPU 占用高

**解决方案：**
```bash
# 限制 CPU 使用
# /etc/systemd/system/zeroclaw.service
[Service]
CPUQuota=50%

# 减少并发任务
# ~/.zeroclaw/config.toml
[agent]
max_concurrent_tasks = 1

# 重启服务
sudo systemctl restart zeroclaw
```

## 配置问题

### 问题 1：配置文件语法错误

**症状：**
```
Error: Failed to parse config.toml
Error: Invalid TOML syntax
```

**解决方案：**
```bash
# 验证 TOML 语法
cat ~/.zeroclaw/config.toml | python3 -c "import toml, sys; toml.load(sys.stdin)"

# 使用默认配置
cp ~/.zeroclaw/config.toml ~/.zeroclaw/config.toml.bak
zeroclaw onboard --force

# 检查配置
zeroclaw config show
```

### 问题 2：配置不生效

**症状：**
- 修改配置后行为未改变

**解决方案：**
```bash
# 重启 Daemon
zeroclaw service restart

# 检查配置路径
zeroclaw status | grep config

# 强制使用特定配置
zeroclaw daemon --config /path/to/config.toml
```

## 网络问题

### 问题 1：无法连接 API

**症状：**
```
Error: Connection timed out
Error: Failed to establish connection
```

**解决方案：**
```bash
# 检查网络连接
ping 8.8.8.8
curl -I https://www.google.com

# 检查 DNS
cat /etc/resolv.conf
nslookup api.anthropic.com

# 使用备用 DNS
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# 检查防火墙
sudo ufw status
sudo iptables -L -n

# 使用代理（如需要）
export HTTPS_PROXY=http://proxy.example.com:8080
zeroclaw agent -m "test"
```

### 问题 2：SSL/TLS 错误

**症状：**
```
Error: SSL certificate verification failed
```

**解决方案：**
```bash
# 更新 CA 证书
sudo update-ca-certificates  # Debian/Ubuntu
sudo update-ca-trust  # RHEL/Fedora

# 检查系统时间
date
sudo timedatectl set-ntp true

# 临时禁用 SSL 验证（不推荐）
# 仅在测试环境使用
export SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt
```

## 崩溃恢复

### 问题：Daemon 频繁崩溃

**解决方案：**
```bash
# 查看崩溃日志
journalctl -u zeroclaw -n 100

# 检查核心转储
ls -la /var/crash/

# 启用自动重启
# /etc/systemd/system/zeroclaw.service
[Service]
Restart=on-failure
RestartSec=10

# 减少负载
# ~/.zeroclaw/config.toml
[agent]
max_concurrent_tasks = 1

# 联系支持
zeroclaw doctor > diagnosis.txt
# 附上日志文件提交 issue
```

## 获取帮助

### 官方资源

- GitHub Issues: https://github.com/zeroclaw-labs/zeroclaw/issues
- Telegram: https://t.me/zeroclawlabs
- Reddit: https://www.reddit.com/r/zeroclawlabs/
- 文档：https://github.com/zeroclaw-labs/zeroclaw/tree/main/docs

### 提交 Issue

```bash
# 收集诊断信息
zeroclaw doctor > diagnosis.txt
zeroclaw status --verbose >> diagnosis.txt
tail -100 ~/.zeroclaw/zeroclaw.log >> diagnosis.txt

# 提交到 GitHub
# https://github.com/zeroclaw-labs/zeroclaw/issues/new
```

## 下一步

- [[ZeroClaw-MOC]] - 返回知识地图
- [[ZeroClaw-常用命令]] - CLI 命令参考
- [[ZeroClaw-安全配置]] - 安全最佳实践

---

_最后更新：2026-02-21_
