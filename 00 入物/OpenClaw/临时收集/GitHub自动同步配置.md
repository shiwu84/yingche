# GitHub 自动同步配置

_配置时间：2026-02-20 11:30_
_配置者：十五 (OpenClaw)_

---

## 📁 Git 仓库信息

- **仓库地址**: `git@github.com:shiwu84/yingche.git`
- **分支**: `main`
- **Vault 路径**: `~/Documents/yingche`

---

## 🔄 自动同步脚本

**脚本位置**: `~/.openclaw/scripts/yingche-git-sync.sh`

**功能**:
1. 检查文件变更
2. 自动暂存文件（遵循 .gitignore）
3. 提交变更（带时间戳）
4. 推送到 GitHub

**用法**:
```bash
# 手动执行
~/.openclaw/scripts/yingche-git-sync.sh "提交信息"

# 自动执行（定时任务调用）
~/.openclaw/scripts/yingche-git-sync.sh "📰 每日技术新闻 $(date +%Y-%m-%d)"
```

---

## 📋 .gitignore 配置

**位置**: `~/Documents/yingche/.gitignore`

**忽略内容**:
- Obsidian 插件和配置
- 工作区文件
- 临时文件 (.DS_Store, Thumbs.db)
- 日志文件
- 缓存文件

**保留内容**:
- ✅ 所有笔记文件 (.md)
- ✅ Canvas 文件 (.canvas)
- ✅ Excalidraw 文件 (.md)
- ✅ 附件文件 (图片、PDF 等)
- ✅ OpenClaw 生成的所有内容

---

## ⏰ 自动推送定时任务

### 1️⃣ 每日技术新闻推送 (9:00 AM)
```json
{
  "name": "每日技术新闻推送",
  "schedule": "0 9 * * *",
  "action": "生成新闻 + Mermaid 图表 + git push"
}
```

**提交格式**: `📰 每日技术新闻 2026-02-20`

### 2️⃣ Arch Linux 小知识 (12:00 PM)
```json
{
  "name": "Arch Linux 小知识",
  "schedule": "0 12 * * *",
  "action": "获取知识 + 保存笔记 + git push"
}
```

**提交格式**: `🐧 Arch Linux 小知识 2026-02-20`

### 3️⃣ 渗透测试知识 (6:00 PM)
```json
{
  "name": "渗透测试知识",
  "schedule": "0 18 * * *",
  "action": "获取知识 + 保存笔记 + git push"
}
```

**提交格式**: `🔒 渗透测试知识 2026-02-20`

---

## 📊 Git 提交流程

```
OpenClaw 生成内容
    ↓
保存到 Obsidian vault
    ↓
定时任务触发
    ↓
执行 yingche-git-sync.sh
    ↓
git add --all (遵循 .gitignore)
    ↓
git commit -m "提交信息"
    ↓
git push origin main
    ↓
GitHub 仓库更新 ✅
```

---

## 🔍 手动操作命令

### 查看状态
```bash
cd ~/Documents/yingche
git status
```

### 查看提交历史
```bash
git log --oneline -10
```

### 手动推送
```bash
~/.openclaw/scripts/yingche-git-sync.sh "手动提交说明"
```

### 查看远程仓库
```bash
git remote -v
```

### 拉取更新
```bash
cd ~/Documents/yingche
git pull origin main
```

---

## 📝 提交信息规范

**格式**: `[Emoji] 描述 日期`

**Emoji 使用**:
- 📰 - 每日技术新闻
- 🐧 - Arch Linux 小知识
- 🔒 - 渗透测试知识
- 🎨 - 可视化图表
- 🤖 - OpenClaw 自动更新
- 🔧 - 配置修改
- 📝 - 一般笔记

**示例**:
```
📰 每日技术新闻 2026-02-20
🐧 Arch Linux 小知识 2026-02-20
🔒 渗透测试知识 2026-02-20
🎨 添加 Mermaid 可视化图表
🤖 OpenClaw 自动更新 2026-02-20
```

---

## ✅ 验证同步

### 检查 GitHub 仓库
访问：https://github.com/shiwu84/yingche

### 查看最新提交
```bash
cd ~/Documents/yingche
git log -1 --stat
```

### 确认文件存在
```bash
ls -la ~/Documents/yingche/00\ 入物/OpenClaw/
```

---

## 🚨 故障处理

### 推送失败
```bash
# 检查 Git 配置
cd ~/Documents/yingche
git remote -v

# 检查 SSH key
ssh -T git@github.com

# 手动推送
git add .
git commit -m "手动修复"
git push origin main
```

### 冲突处理
```bash
# 拉取远程变更
git pull origin main

# 解决冲突后
git add .
git commit -m "解决冲突"
git push origin main
```

### 脚本执行失败
```bash
# 检查脚本权限
chmod +x ~/.openclaw/scripts/yingche-git-sync.sh

# 手动执行测试
~/.openclaw/scripts/yingche-git-sync.sh "测试"
```

---

## 📈 同步统计

### 查看提交频率
```bash
cd ~/Documents/yingche
git log --oneline --since="1 week ago" | wc -l
```

### 查看文件大小
```bash
du -sh ~/Documents/yingche
```

### 查看仓库大小
```bash
cd ~/Documents/yingche
git count-objects -vH
```

---

## 🔐 SSH Key 配置

**公钥位置**: `~/.ssh/id_ed25519.pub`

**添加到 GitHub**:
1. 复制公钥：`cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard`
2. GitHub → Settings → SSH and GPG keys → New SSH key
3. 粘贴公钥并保存

**验证连接**:
```bash
ssh -T git@github.com
# 输出：Hi shiwu84! You've successfully authenticated...
```

---

## 📚 相关文件

- `~/.openclaw/scripts/yingche-git-sync.sh` - 同步脚本
- `~/Documents/yingche/.gitignore` - Git 忽略配置
- `~/.openclaw/cron/jobs.json` - 定时任务配置

---

_配置完成时间：2026-02-20 11:30_
_状态：✅ 正常运行_
_下次同步：2026-02-21 09:00 (每日技术新闻)_
