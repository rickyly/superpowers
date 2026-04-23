---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# 请求代码审查

派发 superpowers:code-reviewer subagent，在问题扩散前捕获它们。审查员会得到精心构造的上下文用于评估 —— 而不是你这次会话的历史。这样审查员专注于工作成果而非你的思考过程，同时保留你自己的上下文以便继续工作。

**核心原则：** 尽早审查，频繁审查。

## 何时请求审查

**强制：**
- subagent 驱动开发中的每个任务之后
- 完成主要特性之后
- 合并到 main 之前

**非强制但有价值：**
- 卡住时（获得新视角）
- 重构之前（基线检查）
- 修复复杂 bug 之后

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发 code-reviewer subagent：**

使用 Task 工具，类型为 superpowers:code-reviewer，填写 `code-reviewer.md` 的模板

**占位符：**
- `{WHAT_WAS_IMPLEMENTED}` —— 你刚刚构建了什么
- `{PLAN_OR_REQUIREMENTS}` —— 应该做什么
- `{BASE_SHA}` —— 起始 commit
- `{HEAD_SHA}` —— 结束 commit
- `{DESCRIPTION}` —— 简要摘要

**3. 对反馈采取行动：**
- 立即修复 Critical 问题
- 继续之前修复 Important 问题
- 记录 Minor 问题供稍后处理
- 如果审查员错了就反驳（附带理由）

## 示例

```
[Just completed Task 2: Add verification function]

You: Let me request code review before proceeding.

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[Dispatch superpowers:code-reviewer subagent]
  WHAT_WAS_IMPLEMENTED: Verification and repair functions for conversation index
  PLAN_OR_REQUIREMENTS: Task 2 from docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: Added verifyIndex() and repairIndex() with 4 issue types

[Subagent returns]:
  Strengths: Clean architecture, real tests
  Issues:
    Important: Missing progress indicators
    Minor: Magic number (100) for reporting interval
  Assessment: Ready to proceed

You: [Fix progress indicators]
[Continue to Task 3]
```

## 与工作流的集成

**subagent 驱动开发：**
- 每个任务之后审查
- 在问题累积之前捕获
- 进入下一个任务之前修复

**执行计划：**
- 每批次（3 个任务）之后审查
- 获取反馈，应用，继续

**临时开发：**
- 合并前审查
- 卡住时审查

## 红旗

**永远不要：**
- 因为"简单"而跳过审查
- 忽略 Critical 问题
- 带着未修复的 Important 问题继续
- 与有效的技术反馈争辩

**如果审查员错了：**
- 用技术推理反驳
- 展示证明其可行的代码或测试
- 请求澄清

参见模板：requesting-code-review/code-reviewer.md
