# OpenClaw 自动化笔记

_由十五 (OpenClaw) 自动生成和管理_

---

## 📖 简介

这是 OpenClaw 助手在你的 Obsidian vault 中自动创建和管理的内容文件夹。所有技术新闻、学习笔记、可视化图表都会自动保存到这里，并同步到 GitHub。

---

## 📁 文件夹结构

```
OpenClaw/
├── 技术新闻/          # 每日 Linux & AI 技术新闻 + Mermaid 图表
├── AI 研究/           # AI 论文、文章摘要、可视化图表
├── Arch Linux/        # Arch 配置、技巧、命令示例
├── 临时收集/          # 未分类内容、配置文档、测试报告
└── README.md          # 本说明文件
```

| 文件夹 | 用途 | 自动推送时间 |
|--------|------|-------------|
| `技术新闻/` | 每日 Linux & AI 技术新闻 + Mermaid 可视化 | 9:00 AM |
| `AI 研究/` | AI 论文、文章摘要、Mermaid/Canvas/Excalidraw 图表 | 按需 |
| `Arch Linux/` | Arch 配置、技巧、命令示例、PDF 资料 | 12:00 PM |
| `临时收集/` | 渗透测试知识、配置文档、测试报告 | 6:00 PM |

---

## 🤖 自动化能力

### 定时任务（5 个）

| 时间 | 任务 | 输出 |
|------|------|------|
| **3:00 AM** | 🛡️ 系统健康检查 | 检查 OpenClaw 状态、API 连接、磁盘空间 |
| **3:30 AM** | 🔄 OpenClaw 自动更新 | 更新到最新版本 + 修复配置 |
| **9:00 AM** | 📰 每日技术新闻推送 | Linux & AI 新闻 + Mermaid 图表 + GitHub 推送 |
| **12:00 PM** | 🐧 Arch Linux 小知识 | ArchWiki 实用技巧 + GitHub 推送 |
| **6:00 PM** | 🔒 渗透测试知识 | 合法渗透测试技术 + GitHub 推送 |

### 可视化生成

- **Mermaid** - 流程图、序列图、状态图、思维导图（Obsidian 原生支持）
- **Canvas** - 交互式思维导图（径向/自由布局）
- **Excalidraw** - 手绘风格图表（需安装 Excalidraw 插件）

### GitHub 自动同步

所有生成的内容会自动推送到 GitHub：
- **仓库**: https://github.com/shiwu84/yingche
- **分支**: main
- **频率**: 每次生成内容后自动推送
- **提交格式**: `[Emoji] 描述 日期`

---

## 📝 文件命名规范

**格式**: `YYYY-MM-DD-描述。类型.md`

**示例**:
```
2026-02-20-技术新闻日报.md
2026-02-20-新闻分类图.mermaid.md
2026-02-20-新闻思维导图.canvas
2026-02-20-新闻概览.excalidraw.md
2026-02-20-ArchLinux 小知识.md
```

---

## 🎨 可视化技能

### Mermaid Visualizer ✅
- **触发词**: "用 Mermaid 画一个...", "创建一个流程图"
- **输出**: Markdown 代码块，Obsidian 自动渲染
- **类型**: 流程图、序列图、状态图、思维导图、对比图

### Canvas Creator ✅
- **触发词**: "创建一个 Canvas", "做一个思维导图"
- **输出**: `.canvas` 文件，Obsidian 交互式打开
- **布局**: MindMap（径向）、Freeform（自由）

### Excalidraw Diagram ✅
- **触发词**: "用 Excalidraw 画一个...", "画一个手绘图"
- **输出**: `.md` 文件，需切换 Excalidraw 视图
- **风格**: 手绘风格图表

---

## 🔄 工作流程

```
用户请求 / 定时任务
    ↓
OpenClaw 生成内容
    ↓
保存到对应文件夹
    ↓
执行 yingche-git-sync.sh
    ↓
git add → git commit → git push
    ↓
GitHub 仓库更新 ✅
```

---

## 📊 已安装 Skills

### 基础集成
- ✅ obsidian - 基础 Obsidian 集成
- ✅ obsidian-linux - Linux 平台优化
- ✅ save-to-obsidian - 保存内容到 Obsidian

### 可视化 (v2.0.0-openclaw)
- ✅ excalidraw-diagram - 手绘风格图表生成
- ✅ mermaid-visualizer - 专业流程图/序列图
- ✅ obsidian-canvas-creator - Canvas 思维导图

---

## 🛠️ 配置状态

- [x] 文件夹结构创建
- [x] 可视化 skills 优化 (v2.0.0-openclaw)
- [x] 定时任务配置 (5 个任务)
- [x] GitHub 自动同步配置
- [x] .gitignore 配置
- [x] 首次内容推送测试

---

## 📚 相关文档

- [[GitHub 自动同步配置]] - Git 同步详细配置
- [[可视化 Skills 优化记录]] - Skills 优化过程
- [[可视化 Skills 全量测试报告]] - 测试验证报告

---

## 🔗 外部链接

- **GitHub 仓库**: https://github.com/shiwu84/yingche
- **Obsidian 官网**: https://obsidian.md
- **Mermaid 文档**: https://mermaid.js.org
- **Excalidraw**: https://excalidraw.com
- **OpenClaw 文档**: https://docs.openclaw.ai

---

## 📞 使用提示

### 手动触发可视化
```
# Mermaid
"用 Mermaid 画一个 Linux 启动流程图"

# Canvas
"创建一个关于 AI 技术栈的 Canvas 思维导图"

# Excalidraw
"用 Excalidraw 画一个商业模式关系图"
```

### 手动推送 Git
```bash
~/.openclaw/scripts/yingche-git-sync.sh "提交信息"
```

### 查看 Git 状态
```bash
cd ~/Documents/yingche
git status
git log --oneline -5
```

---

## 📈 统计信息

- **创建时间**: 2026-02-20
- **最后更新**: 2026-02-20 11:35
- **文件数量**: 10+
- **GitHub 提交**: 3 次
- **定时任务**: 5 个（3 个带自动推送）

---

_由十五 (OpenClaw) 创建和维护 · 最后更新：2026-02-20 11:35_
