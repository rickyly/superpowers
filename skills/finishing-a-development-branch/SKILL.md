---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# 收尾开发 branch

## 概览

通过呈现清晰的选项并处理所选工作流，引导开发工作收尾。

**核心原则：** 验证测试 → 呈现选项 → 执行选择 → 清理。

**开始时宣布：** "I'm using the finishing-a-development-branch skill to complete this work."

## 流程

### 第 1 步：验证测试

**在呈现选项前，先验证测试通过：**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**

```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

停止。不要进入第 2 步。

**如果测试通过：** 继续第 2 步。

### 第 2 步：确定基准 branch

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或询问："This branch split from main - is that correct?"

### 第 3 步：呈现选项

恰好呈现以下 4 个选项：

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

**不要添加解释** —— 保持选项简洁。

### 第 4 步：执行所选项

#### 选项 1：本地 merge

```bash
# Switch to base branch
git checkout <base-branch>

# Pull latest
git pull

# Merge feature branch
git merge <feature-branch>

# Verify tests on merged result
<test command>

# If tests pass
git branch -d <feature-branch>
```

然后：清理 worktree（第 5 步）

#### 选项 2：push 并创建 PR

```bash
# Push branch
git push -u origin <feature-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

然后：清理 worktree（第 5 步）

#### 选项 3：保持现状

汇报："Keeping branch <name>. Worktree preserved at <path>."

**不要清理 worktree。**

#### 选项 4：丢弃

**先确认：**

```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

等待确切的确认。

若已确认：

```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然后：清理 worktree（第 5 步）

### 第 5 步：清理 worktree

**针对选项 1、2、4：**

检查是否位于 worktree 中：

```bash
git worktree list | grep $(git branch --show-current)
```

若是：

```bash
git worktree remove <worktree-path>
```

**针对选项 3：** 保留 worktree。

## 快速参考

| 选项 | Merge | Push | 保留 worktree | 清理 branch |
|--------|-------|------|---------------|----------------|
| 1. 本地 merge | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持现状 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓（强制） |

## 常见错误

**跳过测试验证**

- **问题：** merge 了坏代码，创建了失败的 PR
- **修正：** 在提供选项前务必验证测试

**开放式提问**

- **问题：** "What should I do next?" → 含糊不清
- **修正：** 恰好呈现 4 个结构化选项

**自动清理 worktree**

- **问题：** 在可能仍需使用时移除了 worktree（选项 2、3）
- **修正：** 仅在选项 1 和 4 时清理

**丢弃时缺少确认**

- **问题：** 意外删除了工作成果
- **修正：** 要求输入 "discard" 作为确认

## 红旗

**永远不要：**

- 在测试失败的情况下继续
- 未验证 merge 结果上的测试就 merge
- 未经确认就删除工作成果
- 未经明确要求就 force-push

**始终：**

- 在提供选项前验证测试
- 恰好呈现 4 个选项
- 对选项 4 要求输入 "discard" 确认
- 仅对选项 1 和 4 清理 worktree

## 集成

**被以下调用：**

- **subagent-driven-development**（第 7 步）—— 所有任务完成后
- **executing-plans**（第 5 步）—— 所有批次完成后

**搭配使用：**

- **using-git-worktrees** —— 清理由该 skill 创建的 worktree
