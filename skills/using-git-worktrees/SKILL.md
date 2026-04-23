---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated git worktrees with smart directory selection and safety verification
---

# 使用 git worktree

## 概述

git worktree 创建共享同一仓库的隔离工作区，让你可以在多个 branch 上同时工作而不用切换。

**核心原则：** 系统化的目录选择 + 安全性校验 = 可靠的隔离。

**开始时宣布：** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## 目录选择流程

按以下优先级：

### 1. 检查已存在的目录

```bash
# Check in priority order
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

**找到了：** 使用该目录。如果两个都存在，`.worktrees` 胜出。

### 2. 检查 CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**如果指定了偏好：** 直接使用，不再询问。

### 3. 询问用户

如果没有已存在的目录，CLAUDE.md 也没有偏好：

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## 安全性校验

### 对于项目内目录（.worktrees 或 worktrees）

**MUST 在创建 worktree 前确认目录被忽略：**

```bash
# Check if directory is ignored (respects local, global, and system gitignore)
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果没有被忽略：**

按照 Jesse 的规则"立即修复损坏的东西"：
1. 在 .gitignore 中加入相应的一行
2. 提交该变更
3. 继续创建 worktree

**为什么关键：** 防止意外把 worktree 内容提交进仓库。

### 对于全局目录（~/.config/superpowers/worktrees）

不需要 .gitignore 校验 —— 完全在项目之外。

## 创建步骤

### 1. 检测项目名

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 创建 worktree

```bash
# Determine full path
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 运行项目初始化

自动检测并运行相应的初始化：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 验证干净的基线

跑测试确保 worktree 从干净状态开始：

```bash
# Examples - use project-appropriate command
npm test
cargo test
pytest
go test ./...
```

**测试失败：** 报告失败，询问是继续还是排查。

**测试通过：** 报告就绪。

### 5. 报告位置

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## 速查

| 情形 | 动作 |
|-----------|--------|
| `.worktrees/` 存在 | 使用它（确认被忽略） |
| `worktrees/` 存在 | 使用它（确认被忽略） |
| 两者都存在 | 使用 `.worktrees/` |
| 都不存在 | 检查 CLAUDE.md → 询问用户 |
| 目录未被忽略 | 加入 .gitignore 并提交 |
| 基线测试失败 | 报告失败并询问 |
| 没有 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

### 跳过忽略校验

- **问题：** worktree 内容被纳入跟踪，污染 git status
- **修法：** 创建项目内 worktree 前始终使用 `git check-ignore`

### 臆断目录位置

- **问题：** 造成不一致，违反项目约定
- **修法：** 按优先级：已存在 > CLAUDE.md > 询问

### 在测试失败时继续

- **问题：** 无法区分新 bug 和已有问题
- **修法：** 报告失败，获得明确许可再继续

### 硬编码初始化命令

- **问题：** 在使用不同工具的项目上会失败
- **修法：** 从项目文件（package.json 等）自动检测

## 示例流程

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## 危险信号

**永远不要：**
- 在没确认被忽略的情况下创建 worktree（项目内）
- 跳过基线测试校验
- 在测试失败的情况下不询问就继续
- 在有歧义时臆断目录位置
- 跳过 CLAUDE.md 检查

**始终：**
- 遵循目录优先级：已存在 > CLAUDE.md > 询问
- 对项目内目录确认它被忽略
- 自动检测并运行项目初始化
- 验证测试基线干净

## 集成

**被以下调用：**
- **brainstorming**（第 4 阶段）—— 当设计获批、接下来要实现时 REQUIRED
- **subagent-driven-development** —— 在执行任何任务之前 REQUIRED
- **executing-plans** —— 在执行任何任务之前 REQUIRED
- 任何需要隔离工作区的 skill

**搭配：**
- **finishing-a-development-branch** —— 工作完成后做清理时 REQUIRED
