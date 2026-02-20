---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, unit-file]
source: https://wiki.archlinux.org/title/Systemd
aliases: [unit 文件，systemd 服务文件]
---

# Systemd Unit 文件

> [!summary] 核心概念
> Unit 文件是 systemd 的配置文件，定义服务、挂载点、设备等资源的行为。

## Unit 文件格式

Systemd 的 unit 文件格式灵感来自：
- XDG Desktop Entry Specification .desktop 文件
- Microsoft Windows .ini 文件

## 基本结构

Unit 文件由多个部分组成，每个部分包含键值对：

```ini
[Unit]
Description=My Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/my-service
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## Unit 文件位置

### 系统级 Units

**包提供的 Units：**
```
/usr/lib/systemd/system/
```

**管理员自定义 Units：**
```
/etc/systemd/system/
```

### 加载顺序

从低到高优先级：
1. `/usr/lib/systemd/system/` - 最低优先级
2. `/etc/systemd/system/` - 最高优先级

> 💡 **Note**: 用户模式（user mode）的加载路径完全不同。

## Unit 命名规则

### 允许的字符

Unit 名称只能包含：
- ASCII 字母数字字符
- 下划线 `_`
- 句点 `.`

### 特殊字符转义

其他字符必须使用 C 风格的 `\x2d` 转义。

**示例：**
```bash
# 将路径转换为 unit 名称
systemd-escape /mnt/my-disk
mnt-my\x2ddisk.mount
```

参考：[systemd-escape(1)](https://man.archlinux.org/man/systemd-escape.1)

## [Unit] 部分

### 基本选项

```ini
[Unit]
# 描述
Description=My Service

# 文档
Documentation=man:my-service(8)
Documentation=https://example.com/docs

# 依赖关系
Requires=network.target
Wants=postgresql.service
Before=network-online.target
After=network.target network-online.target
```

### 依赖关系选项

| 选项 | 说明 |
|------|------|
| `Requires=` | 硬依赖，必须运行 |
| `Wants=` | 软依赖，尝试启动 |
| `BindsTo=` | 绑定依赖，同时停止 |
| `Before=` | 在该 unit 之前启动 |
| `After=` | 在该 unit 之后启动 |
| `Conflicts=` | 冲突的 units |

> 💡 **Note**: `Wants=` 和 `Requires=` 不隐含 `After=`，需要显式指定。

## [Service] 部分

### 基本选项

```ini
[Service]
# 服务类型
Type=simple

# 启动命令
ExecStart=/usr/bin/my-service --config /etc/my-service.conf

# 停止命令
ExecStop=/usr/bin/my-service-stop

# 重新加载命令
ExecReload=/usr/bin/my-service-reload

# 运行用户
User=myuser
Group=mygroup

# 工作目录
WorkingDirectory=/var/lib/my-service

# 环境变量
Environment="KEY=value"
EnvironmentFile=/etc/my-service/env

# 重启策略
Restart=on-failure
RestartSec=5s
```

### 服务类型

| Type | 说明 |
|------|------|
| `simple` | 默认，立即启动，不 fork |
| `forking` | 传统 daemon，fork 后父进程退出 |
| `oneshot` | 一次性脚本，执行后退出 |
| `notify` | 发送通知表示就绪 |
| `dbus` | DBus 名称出现时表示就绪 |
| `idle` | 延迟执行，避免控制台输出混乱 |

参考：[[Systemd-服务类型]] 详细了解每种类型。

### PID 文件

对于 `forking` 类型，指定 PID 文件：

```ini
[Service]
Type=forking
PIDFile=/var/run/my-service.pid
```

### RemainAfterExit

对于 `oneshot` 类型，保持活动状态：

```ini
[Service]
Type=oneshot
RemainAfterExit=yes
```

适用于改变系统状态的脚本（如挂载分区）。

## [Install] 部分

定义安装（启用）时的行为：

```ini
[Install]
# 被哪个 target 包含
WantedBy=multi-user.target

# 别名
Alias=my-service.service

# 也被启用
Also=other-service.service
```

### 常用 WantedBy

| Target | 用途 |
|--------|------|
| `multi-user.target` | 多用户文本模式（最常用） |
| `graphical.target` | 图形界面模式 |
| `default.target` | 系统默认 target |

## 编辑提供的 Units

### 推荐方法：使用 edit

```bash
sudo systemctl edit unit_name
```

这会创建覆盖文件 `/etc/systemd/system/unit_name.d/override.conf`。

**优点：**
- 不与包管理冲突
- 更新包时保留自定义
- 只记录更改的部分

### 完整覆盖

```bash
sudo systemctl edit --full unit_name
```

创建完整的覆盖文件。

### 手动编辑

直接编辑 `/etc/systemd/system/` 中的文件。

> ⚠️ **Warning**: 不要修改 `/usr/lib/systemd/system/` 中的文件，包更新会覆盖。

## 示例：创建服务

### 简单服务

```ini
[Unit]
Description=My Web Application
After=network.target postgresql.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/myapp
ExecStart=/usr/bin/python3 /var/www/myapp/app.py
Restart=on-failure
Environment="FLASK_ENV=production"

[Install]
WantedBy=multi-user.target
```

### 一次性脚本

```ini
[Unit]
Description=Setup Database
After=postgresql.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/setup-db.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

### Forking Daemon

```ini
[Unit]
Description=Legacy Daemon

[Service]
Type=forking
PIDFile=/var/run/legacy.pid
ExecStart=/usr/bin/legacy-daemon start
ExecStop=/usr/bin/legacy-daemon stop

[Install]
WantedBy=multi-user.target
```

## 验证 Unit 文件

### 使用 systemd-analyze

```bash
# 验证语法
systemd-analyze verify /etc/systemd/system/my-service.service

# 查看依赖树
systemd-analyze dot my-service.service | dot -Tsvg > deps.svg

# 查看启动时间
systemd-analyze blame
```

## 注释规则

```ini
# 这是注释，只能在行首使用
# 不能使用行尾注释
ExecStart=/usr/bin/my-service  # 错误！
```

> ⚠️ **Warning**: 行尾注释会导致 unit 激活失败。

## 常用环境变量

```ini
[Service]
# 设置环境变量
Environment="PATH=/usr/bin:/bin"
Environment="NODE_ENV=production"

# 从文件加载
EnvironmentFile=/etc/sysconfig/my-service
```

## 资源限制

```ini
[Service]
# CPU 限制
CPUQuota=50%

# 内存限制
MemoryLimit=512M

# 文件描述符限制
LimitNOFILE=65535
```

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-服务类型]] - 服务类型详解
- [[Systemd-依赖管理]] - 依赖关系配置
- [[Systemd-服务管理]] - systemctl 命令
