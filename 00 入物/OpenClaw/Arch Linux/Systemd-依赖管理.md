---
created: 2026-02-20
updated: 2026-02-20
tags: [arch-linux, systemd, atomic, dependencies]
source: https://wiki.archlinux.org/title/Systemd
aliases: [systemd 依赖，unit 依赖关系]
---

# Systemd 依赖管理

> [!summary] 核心概念
> 通过正确设计 unit 文件的依赖关系，systemd 可以正确解析启动顺序。

## 依赖类型

### Requires= 硬依赖

**语法：**
```ini
[Unit]
Requires=unit1.service unit2.service
```

**行为：**
- 如果 B 被 requires，启动 A 时必须启动 B
- 如果 B 启动失败，A 也会失败
- B 停止时，A 也会停止

**示例：**
```ini
[Unit]
Description=My Web App
Requires=postgresql.service
```

> 💡 **Note**: `Requires=` 不隐含 `After=`，需要显式指定顺序。

### Wants= 软依赖

**语法：**
```ini
[Unit]
Wants=unit1.service
```

**行为：**
- 如果 B 被 wants，启动 A 时会尝试启动 B
- 如果 B 启动失败，A 仍然继续
- B 停止时，A 不受影响

**示例：**
```ini
[Unit]
Description=My Web App
Wants=redis.service
```

> 💡 **Tip**: 对于可选依赖，使用 `Wants=` 而不是 `Requires=`。

### BindsTo= 绑定依赖

**语法：**
```ini
[Unit]
BindsTo=unit1.service
```

**行为：**
- 类似 `Requires=`，但更严格
- B 停止时，A 立即停止
- 适用于紧密耦合的服务

## 顺序控制

### After= 在...之后

**语法：**
```ini
[Unit]
After=network.target network-online.target
```

**行为：**
- 在指定的 unit 之后启动
- 不自动创建依赖关系
- 需要配合 `Requires=` 或 `Wants=`

### Before= 在...之前

**语法：**
```ini
[Unit]
Before=network-online.target
```

**行为：**
- 在指定的 unit 之前启动
- 不自动创建依赖关系

## 典型依赖模式

### 模式 1：需要网络

```ini
[Unit]
Description=My Web Service
After=network.target
Wants=network.target
```

### 模式 2：需要特定服务

```ini
[Unit]
Description=Database-dependent Service
Requires=postgresql.service
After=postgresql.service
```

> ⚠️ **Note**: `Requires=` 和 `After=` 都需要指定。

### 模式 3：可选依赖

```ini
[Unit]
Description=Service with Optional Dependencies
Wants=redis.service memcached.service
After=network.target
```

### 模式 4：多依赖关系

```ini
[Unit]
Description=Complex Service
Requires=postgresql.service redis.service
Wants=elasticsearch.service
After=network.target postgresql.service redis.service
```

## 依赖 Targets

### 常用 Targets

| Target | 说明 |
|--------|------|
| `network.target` | 网络已配置 |
| `network-online.target` | 网络已连接 |
| `multi-user.target` | 多用户文本模式 |
| `graphical.target` | 图形界面模式 |
| `local-fs.target` | 本地文件系统已挂载 |
| `remote-fs.target` | 远程文件系统已挂载 |

### 网络依赖

```ini
# 只需要网络配置完成
After=network.target

# 需要网络真正连接（更严格）
After=network-online.target
Wants=network-online.target
```

> 💡 **Tip**: 如果服务需要访问网络资源，使用 `network-online.target`。

## 依赖解析示例

### Web 应用示例

```ini
[Unit]
Description=My Web Application
Documentation=https://example.com/docs

# 依赖数据库和缓存
Requires=postgresql.service redis.service
After=network.target postgresql.service redis.service

# 可选依赖
Wants=elasticsearch.service
After=elasticsearch.service

[Service]
Type=simple
ExecStart=/usr/bin/webapp

[Install]
WantedBy=multi-user.target
```

### 数据库服务示例

```ini
[Unit]
Description=PostgreSQL Database
After=network.target local-fs.target

[Service]
Type=forking
ExecStart=/usr/bin/pg_ctl start
ExecStop=/usr/bin/pg_ctl stop

[Install]
WantedBy=multi-user.target
```

## 冲突处理

### Conflicts=

```ini
[Unit]
Conflicts=conflicting-service.service
```

**行为：**
- 如果启动 A，会停止 B
- 如果启动 B，会停止 A
- 两者不能同时运行

### 示例：邮件服务器

```ini
[Unit]
Description=Postfix MTA
Conflicts=sendmail.service exim.service
```

## 查看依赖关系

### 查看依赖树

```bash
# 正向依赖（该 unit 依赖什么）
systemctl list-dependencies unit_name

# 反向依赖（什么依赖该 unit）
systemctl list-dependencies --reverse unit_name
```

### 使用 pactree

```bash
# 查看依赖树
systemd-analyze dot unit_name | dot -Tsvg > deps.svg
```

### 查看启动顺序

```bash
systemd-analyze critical-chain unit_name
```

显示影响启动时间的关键路径。

## 条件启动

### ConditionPathExists

```ini
[Unit]
ConditionPathExists=/etc/my-service/enabled
```

仅在文件存在时启动。

### ConditionKernelCommandLine

```ini
[Unit]
ConditionKernelCommandLine=systemd.debug
```

仅在内核命令行包含特定参数时启动。

### ConditionEnvironment

```ini
[Unit]
ConditionEnvironment=MY_ENV_VAR
```

仅在环境变量存在时启动。

## 常见问题

### 问题 1：依赖循环

```
Circular dependency detected: A requires B, B requires A
```

**解决：** 重新设计依赖关系，避免循环。

### 问题 2：启动顺序错误

**症状：** 服务启动时依赖的服务还没准备好。

**解决：** 添加 `After=` 显式指定顺序。

### 问题 3：依赖服务失败

**症状：** 依赖的服务启动失败导致主服务失败。

**解决：** 如果是可选依赖，改用 `Wants=`。

## 最佳实践

### 1. 依赖服务而非 Targets

```ini
# ✅ 推荐：依赖具体服务
After=postgresql.service

# ❌ 避免：依赖 target
After=database.target
```

### 2. 显式指定顺序

```ini
# ✅ 推荐：同时指定依赖和顺序
Requires=postgresql.service
After=postgresql.service

# ❌ 避免：只指定依赖
Requires=postgresql.service
```

### 3. 使用 Wants 表示可选依赖

```ini
# ✅ 推荐：可选依赖
Wants=redis.service

# ❌ 避免：强制可选依赖
Requires=redis.service
```

## 相关链接

- [[Systemd-MOC]] - 返回 Systemd 知识地图
- [[Systemd-Unit 文件]] - Unit 文件配置
- [[Systemd-Targets]] - 启动目标
