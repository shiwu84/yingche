---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, sandbox, docker, security, runtime, atomic]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 沙箱，ZeroClaw Docker 运行时]
---

# ZeroClaw-沙箱运行时

> [!summary] 核心概念
> ZeroClaw 支持 Docker 沙箱运行时，将工具执行隔离在容器中，提供额外的安全层，适合执行不受信任的代码或命令。

## 运行时模式

### 本地运行时（默认）

```toml
# ~/.zeroclaw/config.toml

[runtime]
kind = "local"  # 默认，直接在主机执行
```

**特点：**
- ✅ 性能最佳
- ✅ 无额外依赖
- ⚠️ 安全性依赖系统权限

### Docker 沙箱运行时

```toml
[runtime]
kind = "docker"
image = "zeroclaw/sandbox:latest"
workdir = "/workspace"
```

**特点：**
- ✅ 高度隔离
- ✅ 可限制资源
- ✅ 自动清理
- ⚠️ 需要 Docker
- ⚠️ 轻微性能开销

## Docker 配置

### 安装 Docker

```bash
# Ubuntu/Debian
sudo apt install docker.io
sudo systemctl enable docker
sudo systemctl start docker

# Fedora/RHEL
sudo dnf install docker
sudo systemctl enable docker
sudo systemctl start docker

# macOS
brew install --cask docker

# 验证安装
docker --version
docker run hello-world
```

### 配置 Docker 权限

```bash
# 创建 docker 组
sudo groupadd docker

# 添加用户到 docker 组
sudo usermod -aG docker $USER

# 重新登录或运行
newgrp docker

# 验证
docker run hello-world
```

## 沙箱镜像

### 使用官方镜像

```bash
# 拉取最新镜像
docker pull zeroclaw/sandbox:latest

# 拉取特定版本
docker pull zeroclaw/sandbox:v0.1.0
```

### 自定义镜像

```dockerfile
# Dockerfile
FROM zeroclaw/sandbox:latest

# 安装额外工具
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/*

# 设置工作目录
WORKDIR /workspace

# 设置环境变量
ENV PYTHONUNBUFFERED=1
ENV NODE_ENV=production
```

**构建镜像：**
```bash
docker build -t my-zeroclaw-sandbox .
```

### 镜像配置

```toml
[runtime.docker]
image = "my-zeroclaw-sandbox:latest"
pull_policy = "if_not_present"  # always, never, if_not_present

# 资源限制
memory_limit = "512m"
cpu_limit = 0.5  # CPU 核心数

# 网络配置
network = "none"  # none, bridge, host
dns = ["8.8.8.8", "8.8.4.4"]

# 挂载配置
volumes = [
    "/home/user/workspace:/workspace:rw",
    "/tmp/zeroclaw-cache:/cache:rw"
]

# 环境变量
env = { PYTHONPATH = "/workspace", NODE_ENV = "production" }
```

## 安全配置

### 容器隔离

```toml
[runtime.docker.security]
# 禁止特权模式
privileged = false

# 禁止_capabilities
drop_capabilities = ["ALL"]

# 只读根文件系统
read_only_rootfs = true

# 临时文件系统
tmpfs = ["/tmp", "/var/tmp"]

# 禁止新权限
no_new_privileges = true

# seccomp 配置
seccomp_profile = "default"  # default, unconfined, /path/to/profile.json

# AppArmor 配置
apparmor_profile = "docker-default"
```

### 网络隔离

```toml
[runtime.docker.network]
# 禁用网络（最安全）
mode = "none"

# 或使用自定义网络
# mode = "custom"
# network_name = "zeroclaw-isolated"

# 禁止端口映射
port_mapping = false

# DNS 配置
dns = ["8.8.8.8"]
dns_search = []
```

### 文件系统隔离

```toml
[runtime.docker.filesystem]
# 只读挂载
read_only_paths = ["/etc", "/usr", "/bin"]

# 可写挂载
writable_paths = ["/workspace", "/tmp"]

# 隐藏路径
masked_paths = ["/proc/kcore", "/proc/timer_list"]

# 只读路径
readonly_paths = ["/proc", "/sys"]
```

## 使用示例

### 执行 Shell 命令

```bash
# 启用 Docker 沙箱
zeroclaw agent -m "执行系统命令" \
  --enable-tools shell \
  --runtime docker
```

**配置：**
```toml
[runtime]
kind = "docker"

[tools.shell]
# 沙箱中允许的命令
allowed_commands = ["ls", "cat", "grep", "find", "python3", "node"]
```

### 代码执行

```bash
# 执行 Python 代码
zeroclaw agent -m "运行这个 Python 脚本" \
  --file script.py \
  --enable-tools shell \
  --runtime docker
```

**Python 沙箱配置：**
```toml
[runtime.docker]
image = "zeroclaw/sandbox-python:latest"

[tools.shell]
allowed_commands = ["python3", "pip3"]
allowed_paths = ["/workspace"]
```

### 网络请求

```bash
# 执行需要网络的代码
zeroclaw agent -m "调用这个 API" \
  --enable-tools http_request \
  --runtime docker
```

**网络沙箱配置：**
```toml
[runtime.docker]
network = "bridge"

# 限制可访问的域名
allowed_hosts = ["api.example.com", "github.com"]
```

## 资源管理

### 内存限制

```toml
[runtime.docker]
memory_limit = "256m"  # 硬限制
memory_reservation = "128m"  # 软限制
memory_swap = "512m"  # 内存 + 交换
```

### CPU 限制

```toml
[runtime.docker]
cpu_limit = 0.5  # 0.5 个 CPU 核心
cpu_shares = 512  # 相对权重
cpu_quota = 50000  # 微秒
cpu_period = 100000  # 微秒
```

### 磁盘限制

```toml
[runtime.docker]
# 容器大小限制
storage_limit = "1g"

# 临时文件限制
tmpfs_size = "100m"
```

## 监控和日志

### 容器监控

```bash
# 查看运行中的容器
docker ps --filter "name=zeroclaw"

# 查看容器资源使用
docker stats zeroclaw-sandbox-*

# 查看容器日志
docker logs zeroclaw-sandbox-abc123
```

### 详细日志

```toml
[runtime.docker.logging]
# 容器日志
stdout = true
stderr = true

# 日志驱动
driver = "json-file"

# 日志选项
options = { max-size = "10m", max-file = "3" }
```

### 调试模式

```bash
# 启用调试日志
RUST_LOG=debug zeroclaw daemon

# 查看沙箱事件
docker events --filter "container=zeroclaw*"
```

## 故障排除

### 问题：Docker 无法启动容器

```bash
# 检查 Docker 状态
sudo systemctl status docker

# 检查 Docker 权限
docker run hello-world

# 查看详细错误
zeroclaw daemon --log-level debug
```

### 问题：沙箱中命令不可用

```bash
# 检查镜像内容
docker run --rm -it zeroclaw/sandbox:latest which python3

# 进入容器调试
docker run --rm -it zeroclaw/sandbox:latest bash
```

### 问题：资源限制不生效

```bash
# 检查 cgroup 配置
cat /sys/fs/cgroup/memory/memory.limit_in_bytes

# 验证容器限制
docker inspect zeroclaw-sandbox-* | grep -A 10 HostConfig
```

## 最佳实践

### 1. 最小权限原则

```toml
# 仅启用必要的工具
enabled_tools = ["file_read", "file_write"]
disabled_tools = ["shell"]  # 除非必要

# 限制命令白名单
allowed_commands = ["ls", "cat", "grep"]
```

### 2. 定期更新镜像

```bash
# 每周更新沙箱镜像
docker pull zeroclaw/sandbox:latest
docker image prune -f
```

### 3. 监控异常

```toml
# 启用安全审计
[security.audit]
enabled = true
log_file = "/var/log/zeroclaw/audit.log"
alert_on = ["privilege_escalation", "network_escape"]
```

### 4. 备份和恢复

```bash
# 备份容器配置
docker inspect zeroclaw/sandbox:latest > sandbox-config.json

# 导出镜像
docker save zeroclaw/sandbox:latest > sandbox-backup.tar
```

## 性能优化

### 镜像预热

```bash
# 预拉取镜像
docker pull zeroclaw/sandbox:latest

# 创建镜像快照
docker commit zeroclaw-sandbox-template zeroclaw/sandbox:warmed
```

### 容器复用

```toml
[runtime.docker]
# 复用容器（更快但安全性略低）
reuse_containers = true
max_reuse_count = 10
idle_timeout = 300  # 5 分钟
```

## 下一步

- [[ZeroClaw-工具集成]] - 自定义工具开发
- [[ZeroClaw-安全配置]] - 安全最佳实践
- [[ZeroClaw-Daemon 模式]] - 生产环境部署

---

_最后更新：2026-02-21_
