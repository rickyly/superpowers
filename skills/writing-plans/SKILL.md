---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# 编写实施方案

## 概述

编写详尽的实施方案，假设工程师对我们的代码库毫无背景知识，且品味存疑。记录他们需要了解的一切：每个任务涉及哪些文件、代码内容、测试方法、可能需要查阅的文档，以及如何测试。把完整方案拆成一口大小的小任务交给他们。遵循 DRY、YAGNI、TDD 原则，频繁提交。

假设他们是熟练的开发者，但对我们的工具集和问题领域几乎一无所知。假设他们对良好的测试设计也了解不多。

**开始时声明：**"I'm using the writing-plans skill to create the implementation plan."

**上下文：** 这应当在专用的 worktree 中运行（由 brainstorming skill 创建）。

**方案保存至：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- （用户对方案位置的偏好会覆盖此默认值）

## 范围检查

如果规范涵盖多个独立子系统，它本应在 brainstorming 阶段被拆分为若干子项目规范。如果没有被拆分，建议把它拆成多个方案——每个子系统一个。每个方案都应该能独立产出可运行、可测试的软件。

## 文件结构

在定义任务之前，先规划出将要创建或修改哪些文件，以及每个文件各自负责什么。分解决策在此定型。

- 设计边界清晰、接口定义明确的单元。每个文件应该有一个明确的职责。
- 对一次能全部装进上下文的代码，你的推理最为准确；文件聚焦时，你的编辑也更可靠。宁可选小而聚焦的文件，也不要让一个文件承担过多。
- 一起变更的文件应当放在一起。按职责拆分，而不是按技术层次拆分。
- 在已有代码库中，遵循既定模式。如果代码库使用大文件，不要单方面重构——但如果你正在修改的文件已经臃肿不堪，把一次拆分纳入方案也是合理的。

这套结构会指导任务分解。每个任务都应产出自洽的变更，能够独立成立。

## 一口大小的任务粒度

**每一步是一个动作（2–5 分钟）：**
- "Write the failing test" —— 一步
- "Run it to make sure it fails" —— 一步
- "Implement the minimal code to make the test pass" —— 一步
- "Run the tests and make sure they pass" —— 一步
- "Commit" —— 一步

## 方案文档头部

**每份方案 MUST 以下述头部开始：**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## 任务结构

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 不允许占位符

每一步都必须包含工程师实际需要的内容。以下是**方案失败**的典型表现——永远不要这样写：
- "TBD"、"TODO"、"implement later"、"fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above"（没有实际测试代码）
- "Similar to Task N"（把代码重写一遍——工程师可能是乱序读任务的）
- 只描述做什么却不展示怎么做的步骤（涉及代码的步骤必须给出代码块）
- 引用了任何任务中都未定义的类型、函数或方法

## 记住
- 永远使用精确的文件路径
- 每一步都要有完整代码——如果一步涉及代码改动，就把代码展示出来
- 精确的命令以及预期输出
- DRY、YAGNI、TDD、频繁提交

## 自审

写完整份方案后，带着新鲜的眼光重新审视规范，并对照规范检查方案。这是你自己执行的检查清单——不要派发给 subagent。

**1. 规范覆盖：** 浏览规范中的每个小节或需求。你能指出哪个任务在实现它吗？列出所有缺口。

**2. 占位符扫描：** 在方案中搜索危险信号——上述"不允许占位符"一节列出的任何模式。修掉它们。

**3. 类型一致性：** 你在后续任务中使用的类型、方法签名、属性名是否与先前任务中定义的一致？在 Task 3 里叫 `clearLayers()`、在 Task 7 里却叫 `clearFullLayers()`，这就是 bug。

发现问题就就地修复。无需重新审阅——修好继续走。如果发现规范中某项需求没有对应任务，就把该任务补上。

## 执行交接

保存方案后，提供执行方式的选择：

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**如果选择 Subagent-Driven：**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- 每个任务派发一个全新的 subagent，外加两阶段审阅

**如果选择 Inline Execution：**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- 批量执行，并在检查点处进行审阅
