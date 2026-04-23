# Skill authoring best practices

> 学习如何编写 Claude 能够发现并成功使用的有效 skill。

优秀的 skill 简洁、结构良好，并经过真实使用的测试。本指南提供实用的写作决策，帮助你编写 Claude 能发现并有效使用的 skill。

关于 skill 工作原理的概念性背景，见 [Skills overview](/en/docs/agents-and-tools/agent-skills/overview)。

## Core principles

### Concise is key

[context window](https://platform.claude.com/docs/en/build-with-claude/context-windows) 是公共资源。你的 skill 与 Claude 需要知道的其他一切内容共享上下文窗口，包括：

* system prompt
* 对话历史
* 其他 skill 的元数据
* 你实际的请求

你 skill 中的每个 token 并非都会立刻产生成本。启动时，只有所有 skill 的元数据（name 和 description）会被预加载。只有当 skill 变得相关时，Claude 才会读取 SKILL.md；只有在需要时才读取其他文件。不过，SKILL.md 依然要简洁：一旦 Claude 加载它，每个 token 都会与对话历史和其他上下文竞争。

**默认假设**：Claude 已经很聪明了

只添加 Claude 还不知道的上下文。对每条信息都要发问：

* "Claude 真的需要这个解释吗？"
* "我能假设 Claude 已经知道这件事吗？"
* "这段话值得它所占用的 token 成本吗？"

**好例子：简洁**（约 50 个 token）：

````markdown  theme={null}
## Extract PDF text

Use pdfplumber for text extraction:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**坏例子：过于冗长**（约 150 个 token）：

```markdown  theme={null}
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but we
recommend pdfplumber because it's easy to use and handles most cases well.
First, you'll need to install it using pip. Then you can use the code below...
```

简洁版本假设 Claude 已经知道 PDF 是什么、库如何使用。

### Set appropriate degrees of freedom

让具体程度匹配任务的脆弱性和可变性。

**高自由度**（基于文本的指令）：

何时使用：

* 有多种有效方法
* 决策依赖于上下文
* 启发式指导方法选择

示例：

```markdown  theme={null}
## Code review process

1. Analyze the code structure and organization
2. Check for potential bugs or edge cases
3. Suggest improvements for readability and maintainability
4. Verify adherence to project conventions
```

**中等自由度**（伪代码或带参数的脚本）：

何时使用：

* 存在一个偏好的模式
* 可以有一定变化
* 配置影响行为

示例：

````markdown  theme={null}
## Generate report

Use this template and customize as needed:

```python
def generate_report(data, format="markdown", include_charts=True):
    # Process data
    # Generate output in specified format
    # Optionally include visualizations
```
````

**低自由度**（具体脚本、参数极少或没有）：

何时使用：

* 操作脆弱、容易出错
* 一致性至关重要
* 必须遵循特定顺序

示例：

````markdown  theme={null}
## Database migration

Run exactly this script:

```bash
python scripts/migrate.py --verify --backup
```

Do not modify the command or add additional flags.
````

**比喻**：把 Claude 想象成在路径上探索的机器人：

* **两侧悬崖的窄桥**：只有一条安全的前进路径。提供具体的护栏和确切的指令（低自由度）。例如：必须精确顺序执行的数据库迁移。
* **没有危险的开阔原野**：许多路径都能走到终点。给出总体方向并信任 Claude 找到最佳路线（高自由度）。例如：由上下文决定最佳做法的代码 review。

### Test with all models you plan to use

skill 是对模型的补充，因此其有效性依赖于底层模型。用你打算使用的所有模型测试你的 skill。

**按模型的测试考量：**

* **Claude Haiku**（快速、经济）：skill 提供的指导够不够？
* **Claude Sonnet**（均衡）：skill 是否清晰、高效？
* **Claude Opus**（强推理）：skill 是否避免了过度解释？

对 Opus 完美的内容，对 Haiku 可能需要更多细节。如果你打算在多个模型上使用 skill，应让指令在所有模型上都能良好工作。

## Skill structure

<Note>
  **YAML Frontmatter**：SKILL.md 的 frontmatter 需要两个字段：

  * `name` —— skill 的人类可读名称（最多 64 个字符）
  * `description` —— 一行描述，说明 skill 做什么以及何时使用（最多 1024 个字符）

  完整的 skill 结构细节见 [Skills overview](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)。
</Note>

### Naming conventions

使用一致的命名模式，让 skill 更容易引用和讨论。我们推荐对 skill 名称使用**动名词形式**（verb + -ing），因为这能清楚地描述 skill 提供的活动或能力。

**好的命名示例（动名词形式）：**

* "Processing PDFs"
* "Analyzing spreadsheets"
* "Managing databases"
* "Testing code"
* "Writing documentation"

**可接受的替代方式：**

* 名词短语："PDF Processing"、"Spreadsheet Analysis"
* 动作导向："Process PDFs"、"Analyze Spreadsheets"

**避免：**

* 模糊名称："Helper"、"Utils"、"Tools"
* 过于宽泛："Documents"、"Data"、"Files"
* 在 skill 集合内模式不一致

一致的命名让你更容易：

* 在文档和对话中引用 skill
* 一眼看出 skill 的用途
* 组织和搜索多个 skill
* 维护一个专业而连贯的 skill 库

### Writing effective descriptions

`description` 字段使 skill 可被发现，应同时包含 skill **做什么**以及**何时使用**。

<Warning>
  **始终使用第三人称**。description 会被注入到 system prompt 中，不一致的人称会导致发现问题。

  * **Good：** "Processes Excel files and generates reports"
  * **Avoid：** "I can help you process Excel files"
  * **Avoid：** "You can use this to process Excel files"
</Warning>

**要具体并包含关键词**。同时说明 skill 做什么以及使用的具体触发条件/上下文。

每个 skill 只有一个 description 字段。description 对 skill 的选择至关重要：Claude 要从可能 100+ 个可用 skill 中选出合适的那一个。你的 description 必须提供足够细节让 Claude 知道何时选用此 skill，而 SKILL.md 其余部分提供实现细节。

有效示例：

**PDF Processing skill：**

```yaml  theme={null}
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel Analysis skill：**

```yaml  theme={null}
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git Commit Helper skill：**

```yaml  theme={null}
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

避免像下面这样模糊的 description：

```yaml  theme={null}
description: Helps with documents
```

```yaml  theme={null}
description: Processes data
```

```yaml  theme={null}
description: Does stuff with files
```

### Progressive disclosure patterns

SKILL.md 起到概览的作用，按需把 Claude 指向详细材料，就像入职指南里的目录。关于渐进式披露的解释，见 overview 中的 [How Skills work](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

**实用指引：**

* SKILL.md 正文控制在 500 行以内以获得最佳性能
* 接近此上限时拆分到独立文件
* 使用下面的模式有效地组织指令、代码和资源

#### Visual overview: From simple to complex

基础的 skill 只有一个 SKILL.md 文件，包含元数据和指令：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=87782ff239b297d9a9e8e1b72ed72db9" alt="Simple SKILL.md file showing YAML frontmatter and markdown body" data-og-width="2048" width="2048" data-og-height="1153" height="1153" data-path="images/agent-skills-simple-file.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=c61cc33b6f5855809907f7fda94cd80e 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=90d2c0c1c76b36e8d485f49e0810dbfd 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=ad17d231ac7b0bea7e5b4d58fb4aeabb 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f5d0a7a3c668435bb0aee9a3a8f8c329 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0e927c1af9de5799cfe557d12249f6e6 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=46bbb1a51dd4c8202a470ac8c80a893d 2500w" />

当 skill 规模增长时，你可以打包额外内容，让 Claude 仅在需要时加载：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=a5e0aa41e3d53985a7e3e43668a33ea3" alt="Bundling additional reference files like reference.md and forms.md." data-og-width="2048" width="2048" data-og-height="1327" height="1327" data-path="images/agent-skills-bundling-content.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f8a0e73783e99b4a643d79eac86b70a2 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=dc510a2a9d3f14359416b706f067904a 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=82cd6286c966303f7dd914c28170e385 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=56f3be36c77e4fe4b523df209a6824c6 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=d22b5161b2075656417d56f41a74f3dd 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=3dd4bdd6850ffcc96c6c45fcb0acd6eb 2500w" />

完整的 skill 目录结构可能像这样：

```
pdf/
├── SKILL.md              # Main instructions (loaded when triggered)
├── FORMS.md              # Form-filling guide (loaded as needed)
├── reference.md          # API reference (loaded as needed)
├── examples.md           # Usage examples (loaded as needed)
└── scripts/
    ├── analyze_form.py   # Utility script (executed, not loaded)
    ├── fill_form.py      # Form filling script
    └── validate.py       # Validation script
```

#### Pattern 1: High-level guide with references

````markdown  theme={null}
---
name: PDF Processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

# PDF Processing

## Quick start

Extract text with pdfplumber:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced features

**Form filling**: See [FORMS.md](FORMS.md) for complete guide
**API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
**Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
````

Claude 只在需要时才加载 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

#### Pattern 2: Domain-specific organization

对于涉及多个领域的 skill，按领域组织内容以避免加载无关上下文。当用户询问销售指标时，Claude 只需读取销售相关的 schema，而不需要财务或市场数据。这样可保持 token 使用量低、上下文聚焦。

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

````markdown SKILL.md theme={null}
# BigQuery Data Analysis

## Available datasets

**Finance**: Revenue, ARR, billing → See [reference/finance.md](reference/finance.md)
**Sales**: Opportunities, pipeline, accounts → See [reference/sales.md](reference/sales.md)
**Product**: API usage, features, adoption → See [reference/product.md](reference/product.md)
**Marketing**: Campaigns, attribution, email → See [reference/marketing.md](reference/marketing.md)

## Quick search

Find specific metrics using grep:

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
````

#### Pattern 3: Conditional details

展示基础内容，链接到高级内容：

```markdown  theme={null}
# DOCX Processing

## Creating documents

Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

只有当用户需要这些功能时，Claude 才会读取 REDLINING.md 或 OOXML.md。

### Avoid deeply nested references

当文件从其他被引用文件中再次被引用时，Claude 可能只会部分读取这些文件。遇到嵌套引用时，Claude 可能会用 `head -100` 之类的命令预览内容，而不是读取整个文件，导致信息不完整。

**保持引用从 SKILL.md 出发只有一层深度**。所有参考文件都应从 SKILL.md 直接链接，以确保 Claude 在需要时读取完整文件。

**坏例子：过深**：

```markdown  theme={null}
# SKILL.md
See [advanced.md](advanced.md)...

# advanced.md
See [details.md](details.md)...

# details.md
Here's the actual information...
```

**好例子：一层深度**：

```markdown  theme={null}
# SKILL.md

**Basic usage**: [instructions in SKILL.md]
**Advanced features**: See [advanced.md](advanced.md)
**API reference**: See [reference.md](reference.md)
**Examples**: See [examples.md](examples.md)
```

### Structure longer reference files with table of contents

对超过 100 行的参考文件，在顶部加一个目录。这样即使 Claude 只预览部分内容，也能看到可用信息的全貌。

**示例：**

```markdown  theme={null}
# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...

## Core methods
...
```

然后 Claude 可以按需读取完整文件或跳转到特定章节。

关于这种基于文件系统的架构如何实现渐进式披露的细节，见下文 Advanced 中的 [Runtime environment](#runtime-environment) 章节。

## Workflows and feedback loops

### Use workflows for complex tasks

把复杂操作拆解为清晰的顺序步骤。对特别复杂的工作流，提供一个 Claude 可以复制到回复中并随进展勾选的清单。

**示例 1：研究综合工作流**（适用于不含代码的 skill）：

````markdown  theme={null}
## Research synthesis workflow

Copy this checklist and track your progress:

```
Research Progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**

Review each document in the `sources/` directory. Note the main arguments and supporting evidence.

**Step 2: Identify key themes**

Look for patterns across sources. What themes appear repeatedly? Where do sources agree or disagree?

**Step 3: Cross-reference claims**

For each major claim, verify it appears in the source material. Note which source supports each point.

**Step 4: Create structured summary**

Organize findings by theme. Include:
- Main claim
- Supporting evidence from sources
- Conflicting viewpoints (if any)

**Step 5: Verify citations**

Check that every claim references the correct source document. If citations are incomplete, return to Step 3.
````

此示例展示了工作流如何应用于不需要代码的分析任务。清单模式适用于任何复杂、多步骤的流程。

**示例 2：PDF 表单填写工作流**（适用于含代码的 skill）：

````markdown  theme={null}
## PDF form filling workflow

Copy this checklist and check off items as you complete them:

```
Task Progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify output (run verify_output.py)
```

**Step 1: Analyze the form**

Run: `python scripts/analyze_form.py input.pdf`

This extracts form fields and their locations, saving to `fields.json`.

**Step 2: Create field mapping**

Edit `fields.json` to add values for each field.

**Step 3: Validate mapping**

Run: `python scripts/validate_fields.py fields.json`

Fix any validation errors before continuing.

**Step 4: Fill the form**

Run: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify output**

Run: `python scripts/verify_output.py output.pdf`

If verification fails, return to Step 2.
````

清晰的步骤能防止 Claude 跳过关键校验。清单同时帮助 Claude 和你在多步工作流中跟踪进度。

### Implement feedback loops

**常见模式**：运行校验器 → 修复错误 → 重复

这种模式能极大提升输出质量。

**示例 1：风格指南合规**（适用于不含代码的 skill）：

```markdown  theme={null}
## Content review process

1. Draft your content following the guidelines in STYLE_GUIDE.md
2. Review against the checklist:
   - Check terminology consistency
   - Verify examples follow the standard format
   - Confirm all required sections are present
3. If issues found:
   - Note each issue with specific section reference
   - Revise the content
   - Review the checklist again
4. Only proceed when all requirements are met
5. Finalize and save the document
```

这展示了用参考文档而非脚本的校验循环模式。此处的"校验器"就是 STYLE\_GUIDE.md，Claude 通过阅读对照来执行检查。

**示例 2：文档编辑流程**（适用于含代码的 skill）：

```markdown  theme={null}
## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues in the XML
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. Test the output document
```

校验循环能尽早捕获错误。

## Content guidelines

### Avoid time-sensitive information

不要包含会过时的信息：

**坏例子：时间敏感**（将来会变错）：

```markdown  theme={null}
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
```

**好例子**（使用 "old patterns" 小节）：

```markdown  theme={null}
## Current method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>

The v1 API used: `api.example.com/v1/messages`

This endpoint is no longer supported.
</details>
```

"old patterns" 小节在不让主内容混乱的前提下提供历史上下文。

### Use consistent terminology

选一个术语并在整个 skill 中一致使用：

**好 —— 一致**：

* 始终用 "API endpoint"
* 始终用 "field"
* 始终用 "extract"

**坏 —— 不一致**：

* 混用 "API endpoint"、"URL"、"API route"、"path"
* 混用 "field"、"box"、"element"、"control"
* 混用 "extract"、"pull"、"get"、"retrieve"

一致性有助于 Claude 理解并遵循指令。

## Common patterns

### Template pattern

为输出格式提供模板。让严格程度匹配需要。

**对严格要求**（如 API 响应或数据格式）：

````markdown  theme={null}
## Report structure

ALWAYS use this exact template structure:

```markdown
# [Analysis Title]

## Executive summary
[One-paragraph overview of key findings]

## Key findings
- Finding 1 with supporting data
- Finding 2 with supporting data
- Finding 3 with supporting data

## Recommendations
1. Specific actionable recommendation
2. Specific actionable recommendation
```
````

**对灵活指引**（当适配有用时）：

````markdown  theme={null}
## Report structure

Here is a sensible default format, but use your best judgment based on the analysis:

```markdown
# [Analysis Title]

## Executive summary
[Overview]

## Key findings
[Adapt sections based on what you discover]

## Recommendations
[Tailor to the specific context]
```

Adjust sections as needed for the specific analysis type.
````

### Examples pattern

对于输出质量依赖于能看到示例的 skill，像常规 prompting 那样提供输入/输出对：

````markdown  theme={null}
## Commit message format

Generate commit messages following these examples:

**Example 1:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**Example 2:**
Input: Fixed bug where dates displayed incorrectly in reports
Output:
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

**Example 3:**
Input: Updated dependencies and refactored error handling
Output:
```
chore: update dependencies and refactor error handling

- Upgrade lodash to 4.17.21
- Standardize error response format across endpoints
```

Follow this style: type(scope): brief description, then detailed explanation.
````

相比仅有描述，示例能让 Claude 更清楚地理解期望的风格与细节程度。

### Conditional workflow pattern

引导 Claude 通过决策点：

```markdown  theme={null}
## Document modification workflow

1. Determine the modification type:

   **Creating new content?** → Follow "Creation workflow" below
   **Editing existing content?** → Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
   - Repack when complete
```

<Tip>
  如果工作流变得庞大或含有许多步骤，考虑把它们拆到独立文件，并告诉 Claude 根据当前任务读取对应文件。
</Tip>

## Evaluation and iteration

### Build evaluations first

**在编写大量文档之前先创建 evaluation。** 这能确保你的 skill 解决的是真实问题，而不是记录想象中的问题。

**evaluation 驱动的开发：**

1. **识别缺口**：在没有 skill 的情况下让 Claude 完成代表性任务。记录具体的失败或缺失上下文
2. **创建 evaluation**：构建三个测试这些缺口的场景
3. **建立基线**：测量 Claude 没有 skill 时的表现
4. **编写最小指令**：刚好创建足够的内容来应对缺口并通过 evaluation
5. **迭代**：执行 evaluation，与基线对比并改进

这种方法能保证你解决的是实际问题，而不是在预测可能永远不会出现的需求。

**evaluation 结构**：

```json  theme={null}
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF file using an appropriate PDF processing library or command-line tool",
    "Extracts text content from all pages in the document without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
  ]
}
```

<Note>
  此示例展示了一个数据驱动、带简易测试评分表的 evaluation。我们目前未提供内置方式来运行这些 evaluation。用户可以创建自己的 evaluation 系统。evaluation 是衡量 skill 有效性的真实来源。
</Note>

### Develop Skills iteratively with Claude

最有效的 skill 开发流程涉及 Claude 自身。与一个 Claude 实例（"Claude A"）合作创建 skill，而该 skill 将由其他实例（"Claude B"）使用。Claude A 帮助你设计和改进指令，Claude B 则在真实任务中测试它们。这种做法有效，是因为 Claude 模型既懂如何编写有效的 agent 指令，也懂 agent 需要什么信息。

**创建新 skill：**

1. **先不使用 skill 完成一个任务**：用常规 prompting 让 Claude A 解决一个问题。过程中你会自然地提供上下文、说明偏好、分享流程知识。留意哪些信息你反复提供。

2. **识别可复用模式**：完成任务后，识别你提供的上下文中哪些对类似未来任务有用。

   **例子**：如果你做了 BigQuery 分析，可能提供了表名、字段定义、过滤规则（如"始终排除测试账户"）以及常见查询模式。

3. **让 Claude A 创建 skill**："创建一个 skill 来捕获我们刚才用过的 BigQuery 分析模式。包含表 schema、命名约定，以及关于过滤测试账户的规则。"

   <Tip>
     Claude 模型原生理解 skill 的格式和结构。你不需要特殊的 system prompt 或 "writing skills" skill 就能让 Claude 帮你创建 skill。直接让 Claude 创建一个 skill，它就会生成结构正确的 SKILL.md 内容，带有合适的 frontmatter 和正文。
   </Tip>

4. **审视简洁性**：检查 Claude A 有没有加入不必要的解释。问："移除关于 win rate 含义的解释——Claude 已经知道了。"

5. **改进信息架构**：让 Claude A 把内容组织得更有效。例如："把它重新组织一下，让表 schema 放到独立参考文件里。以后可能还会加更多表。"

6. **在类似任务上测试**：用 Claude B（加载了此 skill 的新实例）在相关用例上使用该 skill。观察 Claude B 是否找到正确的信息、正确应用规则并成功完成任务。

7. **基于观察迭代**：如果 Claude B 挣扎或漏掉了什么，带着具体问题回到 Claude A："Claude 使用这个 skill 时，Q4 忘记按日期过滤了。我们是不是该加一节日期过滤模式？"

**迭代现有 skill：**

改进 skill 时，相同的层级模式继续。你会在以下两者间切换：

* **与 Claude A 合作**（帮助改进 skill 的专家）
* **用 Claude B 测试**（使用 skill 执行真实工作的 agent）
* **观察 Claude B 的行为**，并把洞察带回给 Claude A

1. **在真实工作流中使用 skill**：给 Claude B（已加载 skill）实际任务，而不是测试场景

2. **观察 Claude B 的行为**：记录它在哪里挣扎、成功或做出意外选择

   **观察示例**："当我让 Claude B 做区域销售报告时，它写出了查询但忘了过滤测试账户，尽管 skill 里提到了这条规则。"

3. **回到 Claude A 改进**：分享当前的 SKILL.md 并描述你观察到的情况。问："当我要求区域报告时，我注意到 Claude B 忘记过滤测试账户。skill 提到了过滤，但可能不够醒目？"

4. **审视 Claude A 的建议**：Claude A 可能建议重新组织以突出规则，使用更强的语言如 "MUST filter" 代替 "always filter"，或重构工作流章节。

5. **应用并测试改动**：用 Claude A 的改进更新 skill，然后在相似请求上再次用 Claude B 测试

6. **根据使用情况重复**：在遇到新场景时继续这个观察-改进-测试循环。每次迭代都基于真实 agent 行为改进 skill，而不是基于假设。

**收集团队反馈：**

1. 把 skill 分享给队友并观察他们的使用情况
2. 问：skill 是否在预期时触发？指令是否清楚？缺少什么？
3. 把反馈融入进来，以覆盖你自己使用模式中的盲点

**为什么此方法有效**：Claude A 理解 agent 需要什么，你提供领域专业知识，Claude B 在真实使用中暴露缺口，迭代改进基于观察到的行为而非假设。

### Observe how Claude navigates Skills

在迭代 skill 时，留意 Claude 在实践中如何实际使用它们。观察：

* **意料之外的探索路径**：Claude 是否按你没预料的顺序读文件？这可能表明你的结构没你以为的那样直观
* **错过的连接**：Claude 是否没能跟上对重要文件的引用？你的链接可能需要更明确或更醒目
* **对某些章节的过度依赖**：如果 Claude 反复读同一个文件，考虑是否应把该内容直接放进 SKILL.md
* **被忽略的内容**：如果 Claude 从不访问某个打包文件，它可能没必要，或在主指令中发出的信号太弱

基于这些观察而非假设来迭代。skill 元数据中的 'name' 和 'description' 尤其关键。Claude 在决定是否针对当前任务触发 skill 时会使用它们。确保它们清晰地描述 skill 做什么以及何时使用。

## Anti-patterns to avoid

### Avoid Windows-style paths

始终使用正斜杠作为文件路径分隔符，即便在 Windows 上：

* ✓ **Good**：`scripts/helper.py`、`reference/guide.md`
* ✗ **Avoid**：`scripts\helper.py`、`reference\guide.md`

Unix 风格路径在所有平台都能工作，而 Windows 风格路径在 Unix 系统上会导致错误。

### Avoid offering too many options

除非必要，不要给出多种方案：

````markdown  theme={null}
**Bad example: Too many choices** (confusing):
"You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or..."

**Good example: Provide a default** (with escape hatch):
"Use pdfplumber for text extraction:
```python
import pdfplumber
```

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."
````

## Advanced: Skills with executable code

下面的章节聚焦于包含可执行脚本的 skill。如果你的 skill 只包含 markdown 指令，可直接跳到 [Checklist for effective Skills](#checklist-for-effective-skills)。

### Solve, don't punt

为 skill 写脚本时，要处理错误情况，而不是把问题甩给 Claude。

**好例子：显式处理错误**：

```python  theme={null}
def process_file(path):
    """Process a file, creating it if it doesn't exist."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # Create file with default content instead of failing
        print(f"File {path} not found, creating default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # Provide alternative instead of failing
        print(f"Cannot access {path}, using default")
        return ''
```

**坏例子：甩给 Claude**：

```python  theme={null}
def process_file(path):
    # Just fail and let Claude figure it out
    return open(path).read()
```

配置参数也应证明其合理性并有文档，以避免「巫术常量」（Ousterhout 定律）。如果你自己不知道正确的值，Claude 又如何判断？

**好例子：自文档化**：

```python  theme={null}
# HTTP requests typically complete within 30 seconds
# Longer timeout accounts for slow connections
REQUEST_TIMEOUT = 30

# Three retries balances reliability vs speed
# Most intermittent failures resolve by the second retry
MAX_RETRIES = 3
```

**坏例子：魔法数字**：

```python  theme={null}
TIMEOUT = 47  # Why 47?
RETRIES = 5   # Why 5?
```

### Provide utility scripts

即使 Claude 能自己写脚本，预先准备好的脚本仍有优势：

**工具脚本的好处：**

* 比生成的代码更可靠
* 节省 token（无需在上下文中包含代码）
* 节省时间（无需生成代码）
* 使用时保持一致性

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=4bbc45f2c2e0bee9f2f0d5da669bad00" alt="Bundling executable scripts alongside instruction files" data-og-width="2048" width="2048" data-og-height="1154" height="1154" data-path="images/agent-skills-executable-scripts.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=9a04e6535a8467bfeea492e517de389f 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=e49333ad90141af17c0d7651cca7216b 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=954265a5df52223d6572b6214168c428 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=2ff7a2d8f2a83ee8af132b29f10150fd 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=48ab96245e04077f4d15e9170e081cfb 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0301a6c8b3ee879497cc5b5483177c90 2500w" />

上图展示了可执行脚本如何与指令文件协同工作。指令文件（forms.md）引用脚本，Claude 可以执行它，而无需把脚本内容加载到上下文中。

**重要区分**：在指令中说明 Claude 应该：

* **执行脚本**（最常见）："Run `analyze_form.py` to extract fields"
* **把它当作参考阅读**（适用于复杂逻辑）："See `analyze_form.py` for the field extraction algorithm"

对大多数工具脚本而言，执行更可取，因为更可靠、更高效。关于脚本执行如何工作的细节，见下文 [Runtime environment](#runtime-environment) 章节。

**示例：**

````markdown  theme={null}
## Utility scripts

**analyze_form.py**: Extract all form fields from PDF

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

Output format:
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**: Check for overlapping bounding boxes

```bash
python scripts/validate_boxes.py fields.json
# Returns: "OK" or lists conflicts
```

**fill_form.py**: Apply field values to PDF

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
````

### Use visual analysis

当输入可以被渲染为图像时，让 Claude 来分析它们：

````markdown  theme={null}
## Form layout analysis

1. Convert PDF to images:
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. Analyze each page image to identify form fields
3. Claude can see field locations and types visually
````

<Note>
  在这个例子中，你需要自行编写 `pdf_to_images.py` 脚本。
</Note>

Claude 的视觉能力有助于理解布局与结构。

### Create verifiable intermediate outputs

当 Claude 执行复杂的开放式任务时，它可能会犯错。"plan-validate-execute"模式通过先让 Claude 以结构化格式创建一个计划，再用脚本校验计划，然后执行，从而尽早捕获错误。

**示例**：设想让 Claude 基于电子表格更新 PDF 中的 50 个表单字段。没有校验时，Claude 可能引用不存在的字段、产生冲突值、遗漏必填字段或错误应用更新。

**解决方案**：使用上面展示的工作流模式（PDF 表单填写），但增加一个中间的 `changes.json` 文件，在应用改动前先校验它。工作流变为：分析 → **创建计划文件** → **校验计划** → 执行 → 验证。

**此模式为何有效：**

* **尽早捕获错误**：校验在改动应用前就发现问题
* **机器可验证**：脚本提供客观验证
* **可回退的计划**：Claude 可在不触碰原件的情况下迭代计划
* **清晰的调试**：错误信息指向具体问题

**何时使用**：批量操作、破坏性变更、复杂校验规则、高风险操作。

**实现提示**：让校验脚本冗长并给出具体错误信息，例如 "Field 'signature\_date' not found. Available fields: customer\_name, order\_total, signature\_date\_signed"，以帮助 Claude 修复问题。

### Package dependencies

skill 运行在代码执行环境中，该环境有平台相关的限制：

* **claude.ai**：可以从 npm 和 PyPI 安装包，并从 GitHub 仓库拉取
* **Anthropic API**：没有网络访问，也无法在运行时安装包

在 SKILL.md 中列出所需包，并到 [code execution tool documentation](/en/docs/agents-and-tools/tool-use/code-execution-tool) 确认它们可用。

### Runtime environment

skill 运行在一个代码执行环境中，具备文件系统访问、bash 命令以及代码执行能力。关于此架构的概念性解释，见 overview 中的 [The Skills architecture](/en/docs/agents-and-tools/agent-skills/overview#the-skills-architecture)。

**这对你的写作有何影响：**

**Claude 如何访问 skill：**

1. **元数据预加载**：启动时，所有 skill 的 YAML frontmatter 中的 name 和 description 都会加载到 system prompt
2. **按需读取文件**：Claude 在需要时用 bash Read 工具从文件系统访问 SKILL.md 和其他文件
3. **脚本高效执行**：工具脚本可通过 bash 执行，而无需将完整内容加载进上下文。只有脚本的输出会消耗 token
4. **大文件无上下文代价**：参考文件、数据或文档在实际被读取之前不会消耗上下文 token

* **文件路径很重要**：Claude 像浏览文件系统一样浏览你的 skill 目录。使用正斜杠（`reference/guide.md`），而非反斜杠
* **给文件起描述性名字**：名字应能表明内容：`form_validation_rules.md`，而非 `doc2.md`
* **组织结构便于发现**：按领域或功能组织目录
  * Good：`reference/finance.md`、`reference/sales.md`
  * Bad：`docs/file1.md`、`docs/file2.md`
* **打包全面的资源**：包含完整的 API 文档、大量示例、大数据集；在被访问之前没有上下文代价
* **对确定性操作优先使用脚本**：写 `validate_form.py`，而不是让 Claude 现场生成校验代码
* **明确执行意图**：
  * "Run `analyze_form.py` to extract fields"（执行）
  * "See `analyze_form.py` for the extraction algorithm"（当作参考阅读）
* **测试文件访问模式**：用真实请求来验证 Claude 能浏览你的目录结构

**示例：**

```
bigquery-skill/
├── SKILL.md (overview, points to reference files)
└── reference/
    ├── finance.md (revenue metrics)
    ├── sales.md (pipeline data)
    └── product.md (usage analytics)
```

当用户询问收入时，Claude 读取 SKILL.md，看到对 `reference/finance.md` 的引用，并调用 bash 只读那个文件。sales.md 和 product.md 仍在文件系统上，在需要之前消耗零上下文 token。这种基于文件系统的模型正是渐进式披露得以实现的基础。Claude 可以浏览并选择性地仅加载每个任务所需的内容。

关于技术架构的完整细节，见 Skills overview 中的 [How Skills work](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

### MCP tool references

如果你的 skill 使用 MCP（Model Context Protocol）工具，始终使用完全限定的工具名以避免 "tool not found" 错误。

**格式**：`ServerName:tool_name`

**示例**：

```markdown  theme={null}
Use the BigQuery:bigquery_schema tool to retrieve table schemas.
Use the GitHub:create_issue tool to create issues.
```

其中：

* `BigQuery` 和 `GitHub` 是 MCP 服务器名
* `bigquery_schema` 和 `create_issue` 是这些服务器内的工具名

没有服务器前缀时，Claude 可能找不到工具，特别是当有多个 MCP 服务器可用时。

### Avoid assuming tools are installed

不要假设包已可用：

````markdown  theme={null}
**Bad example: Assumes installation**:
"Use the pdf library to process the file."

**Good example: Explicit about dependencies**:
"Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
````

## Technical notes

### YAML frontmatter requirements

SKILL.md 的 frontmatter 需要 `name`（最多 64 字符）和 `description`（最多 1024 字符）字段。完整的结构细节见 [Skills overview](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)。

### Token budgets

SKILL.md 正文控制在 500 行以内以获得最佳性能。如果内容超出此限，用前面描述的渐进式披露模式拆分到独立文件。架构细节见 [Skills overview](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

## Checklist for effective Skills

发布 skill 之前，核对：

### Core quality

* [ ] Description is specific and includes key terms
* [ ] Description includes both what the Skill does and when to use it
* [ ] SKILL.md body is under 500 lines
* [ ] Additional details are in separate files (if needed)
* [ ] No time-sensitive information (or in "old patterns" section)
* [ ] Consistent terminology throughout
* [ ] Examples are concrete, not abstract
* [ ] File references are one level deep
* [ ] Progressive disclosure used appropriately
* [ ] Workflows have clear steps

### Code and scripts

* [ ] Scripts solve problems rather than punt to Claude
* [ ] Error handling is explicit and helpful
* [ ] No "voodoo constants" (all values justified)
* [ ] Required packages listed in instructions and verified as available
* [ ] Scripts have clear documentation
* [ ] No Windows-style paths (all forward slashes)
* [ ] Validation/verification steps for critical operations
* [ ] Feedback loops included for quality-critical tasks

### Testing

* [ ] At least three evaluations created
* [ ] Tested with Haiku, Sonnet, and Opus
* [ ] Tested with real usage scenarios
* [ ] Team feedback incorporated (if applicable)

## Next steps

<CardGroup cols={2}>
  <Card title="Get started with Agent Skills" icon="rocket" href="/en/docs/agents-and-tools/agent-skills/quickstart">
    Create your first Skill
  </Card>

  <Card title="Use Skills in Claude Code" icon="terminal" href="/en/docs/claude-code/skills">
    Create and manage Skills in Claude Code
  </Card>

  <Card title="Use Skills with the API" icon="code" href="/en/api/skills-guide">
    Upload and use Skills programmatically
  </Card>
</CardGroup>
