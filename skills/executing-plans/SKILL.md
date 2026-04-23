---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# 执行计划

## 概览

加载计划、批判性审查、执行所有任务，完成后汇报。

**开始时宣布：** "I'm using the executing-plans skill to implement this plan."

**注意：** 告诉你的人类伙伴，Superpowers 在可以使用 subagent 时效果要好得多。如果运行在支持 subagent 的平台（例如 Claude Code 或 Codex），其工作质量会显著提升。若可用 subagent，请改用 superpowers:subagent-driven-development 而非本 skill。

## 流程

### 第 1 步：加载并审查计划

1. 读取计划文件
2. 批判性审查 —— 找出对计划的任何疑问或顾虑
3. 如有顾虑：在开始前与你的人类伙伴沟通
4. 如无顾虑：创建 TodoWrite 并继续

### 第 2 步：执行任务

对每个任务：

1. 标记为 in_progress
2. 严格按每个步骤执行（计划已拆成小步骤）
3. 按指定方式运行验证
4. 标记为 completed

### 第 3 步：完成开发

所有任务完成并验证后：

- 宣布："I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL：** 使用 superpowers:finishing-a-development-branch
- 按该 skill 执行：验证测试、呈现选项、执行选择

## 何时停下来求助

**遇到以下情况立即停止执行：**

- 遇到阻塞（缺少依赖、测试失败、指令不清晰）
- 计划存在关键缺口导致无法开始
- 你看不懂某条指令
- 验证反复失败

**应当寻求澄清，而不是猜测。**

## 何时回到更早的步骤

**以下情况返回审查（第 1 步）：**

- 伙伴根据你的反馈更新了计划
- 需要从根本上重新思考方案

**不要强行绕过阻塞** —— 停下来问。

## 谨记

- 先批判性地审查计划
- 严格按计划步骤执行
- 不要跳过验证
- 计划要求时引用 skill
- 被阻塞时停下，不要猜
- 未经用户明确同意，绝不在 main/master branch 上启动实现

## 集成

**必备工作流 skill：**

- **superpowers:using-git-worktrees** —— REQUIRED：开始前搭建隔离工作区
- **superpowers:writing-plans** —— 创建本 skill 所执行的计划
- **superpowers:finishing-a-development-branch** —— 所有任务完成后收尾开发
