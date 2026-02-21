---
created: 2026-02-21
updated: 2026-02-21
tags: [zeroclaw, openclaw, ai-agent, rust, moc]
source: https://github.com/zeroclaw-labs/zeroclaw
aliases: [ZeroClaw 使用指南，ZeroClaw MOC]
---

# ZeroClaw MOC

> [!summary] 零开销 AI 助手基础设施
> ZeroClaw 是一个用 Rust 编写的轻量级、高性能 AI 助手基础设施，内存占用 <5MB，启动时间 <10ms，是 OpenClaw 的 Rust 替代方案。

## 核心特性

- 🏎️ **精益运行时**：常见 CLI 工作流仅需几 MB 内存
- 💰 **低成本部署**：可在 $10 硬件上运行
- ⚡ **快速冷启动**：单次二进制文件，启动近乎即时
- 🌍 **便携架构**：支持 ARM、x86、RISC-V

## 与 OpenClaw 对比

| 特性 | OpenClaw | ZeroClaw |
|------|----------|----------|
| 语言 | TypeScript | Rust |
| 内存占用 | >1GB | <5MB |
| 启动时间 | >500s | <10ms |
| 二进制大小 | ~28MB | ~8.8MB |
| 运行成本 | Mac Mini $599 | 任意硬件 $10 |

## 原子笔记

### 入门指南

- [[ZeroClaw-快速开始]] - 安装和初始化配置
- [[ZeroClaw-系统要求]] - 编译和运行环境
- [[ZeroClaw-一键安装]] - bootstrap.sh 使用指南

### 核心概念

- [[ZeroClaw-架构设计]] - Trait 驱动的架构解析
- [[ZeroClaw-认证机制]] - OAuth 和 API Key 配置
- [[ZeroClaw-通道配置]] - Telegram/Discord/Slack 集成

### 命令行使用

- [[ZeroClaw-常用命令]] - CLI 命令速查
- [[ZeroClaw-Agent 模式]] - 交互式 Agent 使用
- [[ZeroClaw-Daemon 模式]] - 后台守护进程配置

### 高级配置

- [[ZeroClaw-提供者配置]] - OpenAI/Anthropic/本地模型
- [[ZeroClaw-沙箱运行时]] - Docker 沙箱配置
- [[ZeroClaw-迁移指南]] - 从 OpenClaw 迁移

### 故障排除

- [[ZeroClaw-常见问题]] - FAQ 和解决方案
- [[ZeroClaw-性能优化]] - 性能调优指南
- [[ZeroClaw-安全配置]] - 安全最佳实践

## 外部资源

- [GitHub - zeroclaw-labs/zeroclaw](https://github.com/zeroclaw-labs/zeroclaw)
- [ZeroClaw 官方文档](https://github.com/zeroclaw-labs/zeroclaw/tree/main/docs)
- [ZeroClaw Telegram 频道](https://t.me/zeroclawlabs)
- [ZeroClaw Reddit](https://www.reddit.com/r/zeroclawlabs/)

## 相关项目

- [[OpenClaw 使用笔记]] - OpenClaw 使用指南
- [[AI 助手对比]] - 各 AI 助手框架对比分析

---

_最后更新：2026-02-21_
