# Code Quality Reviewer Prompt Template

派发代码质量审查 subagent 时使用本模板。

**目的：** 验证实现是否扎实（整洁、已测试、易维护）

**仅在 spec 合规审查通过后派发。**

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [来自 implementer 的汇报]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [任务开始前的 commit]
  HEAD_SHA: [当前 commit]
  DESCRIPTION: [任务摘要]
```

**除标准代码质量关注点外，reviewer 还应检查：**
- 每个文件是否具有单一清晰的职责和定义良好的接口？
- 各单元是否被拆分为可独立理解和测试的形态？
- 实现是否遵循计划中的文件结构？
- 本次实现是否创建了已经很大的新文件，或显著增大了已有文件？（不要追究已存在的文件体量——只关注本次改动带来的增量。）

**Code reviewer 返回：** Strengths、Issues（Critical/Important/Minor）、Assessment
