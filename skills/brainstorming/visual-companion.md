# Visual Companion 指南

基于浏览器的视觉头脑风暴伴侣工具，用于展示 mockup、图示和选项。

## 何时使用

按问题决策，而不是按会话决策。判断标准：**相比阅读，用户能通过观看更好地理解这个问题吗？**

**使用浏览器** 当内容本身是视觉性的：

- **UI mockup** —— 线框图、布局、导航结构、组件设计
- **架构图** —— 系统组件、数据流、关系图
- **并排视觉对比** —— 对比两种布局、两种配色方案、两种设计方向
- **设计打磨** —— 当问题是关于观感、间距、视觉层次时
- **空间关系** —— 以图示呈现的状态机、流程图、实体关系

**使用终端** 当内容是文本或表格：

- **需求和范围问题** —— "X 是什么意思？""哪些特性在范围内？"
- **概念性的 A/B/C 选择** —— 在以文字描述的方案之间挑选
- **权衡取舍列表** —— 优缺点、对比表格
- **技术决策** —— API 设计、数据建模、架构方案选择
- **澄清问题** —— 任何答案是文字、而非视觉偏好的问题

一个*关于* UI 话题的问题并不自动就是视觉问题。"你想要哪种向导？"是概念性的——用终端。"这几种向导布局中哪一种感觉更合适？"是视觉性的——用浏览器。

## 工作原理

服务器监听一个目录中的 HTML 文件，并将最新的一个提供给浏览器。你将 HTML 内容写入 `screen_dir`，用户在浏览器中看到它，并可点击来选择选项。选择结果会被记录到 `state_dir/events`，你在下一轮读取。

**内容片段 vs 完整文档：** 如果你的 HTML 文件以 `<!DOCTYPE` 或 `<html` 开头，服务器会原样提供它（仅注入 helper 脚本）。否则，服务器会自动把你的内容包裹进 frame 模板——添加头部、CSS 主题、选择指示器以及所有交互基础设施。**默认写内容片段。** 只在你需要对页面做完全控制时才写完整文档。

## 启动一个会话

```bash
# Start server with persistence (mockups saved to project)
scripts/start-server.sh --project-dir /path/to/project

# Returns: {"type":"server-started","port":52341,"url":"http://localhost:52341",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#           "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

保存响应中的 `screen_dir` 和 `state_dir`。告诉用户打开该 URL。

**查找连接信息：** 服务器会把启动 JSON 写入 `$STATE_DIR/server-info`。如果你在后台启动了服务器且没有捕获 stdout，可读取该文件获取 URL 和端口。使用 `--project-dir` 时，到 `<project>/.superpowers/brainstorm/` 下查找会话目录。

**注意：** 把项目根目录作为 `--project-dir` 传入，这样 mockup 会持久化在 `.superpowers/brainstorm/`，服务器重启后依然保留。不传的话，文件会写到 `/tmp` 并被清理掉。提醒用户，如果 `.superpowers/` 还不在 `.gitignore` 里，就把它加进去。

**按平台启动服务器：**

**Claude Code (macOS / Linux)：**
```bash
# Default mode works — the script backgrounds the server itself
scripts/start-server.sh --project-dir /path/to/project
```

**Claude Code (Windows)：**
```bash
# Windows auto-detects and uses foreground mode, which blocks the tool call.
# Use run_in_background: true on the Bash tool call so the server survives
# across conversation turns.
scripts/start-server.sh --project-dir /path/to/project
```
通过 Bash tool 调用时，在调用上设置 `run_in_background: true`。然后在下一轮读取 `$STATE_DIR/server-info` 以获取 URL 和端口。

**Codex：**
```bash
# Codex reaps background processes. The script auto-detects CODEX_CI and
# switches to foreground mode. Run it normally — no extra flags needed.
scripts/start-server.sh --project-dir /path/to/project
```

**Gemini CLI：**
```bash
# Use --foreground and set is_background: true on your shell tool call
# so the process survives across turns
scripts/start-server.sh --project-dir /path/to/project --foreground
```

**其他环境：** 服务器 MUST 在多轮对话之间保持在后台运行。如果你的环境会回收分离进程，使用 `--foreground`，并用该平台的后台执行机制来启动这条命令。

如果该 URL 从你的浏览器无法访问（在远程/容器化环境中常见），绑定一个非回环主机：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

用 `--url-host` 控制返回的 URL JSON 中打印的主机名。

## 循环流程

1. **检查服务器存活**，然后**将 HTML 写入** `screen_dir` 中的一个新文件：
   - 每次写入前，检查 `$STATE_DIR/server-info` 是否存在。如果不存在（或 `$STATE_DIR/server-stopped` 存在），说明服务器已关闭——继续之前用 `start-server.sh` 重新启动。服务器在闲置 30 分钟后会自动退出。
   - 使用语义化的文件名：`platform.html`、`visual-style.html`、`layout.html`
   - **绝不复用文件名** —— 每屏用一个新文件
   - 使用 Write tool —— **绝不使用 cat/heredoc**（会把噪声倾入终端）
   - 服务器会自动提供最新的文件

2. **告诉用户要期待什么，然后结束你的回合：**
   - 提醒他们 URL（每一步都要，而不仅是第一步）
   - 给出屏幕上内容的简短文本摘要（例如 "Showing 3 layout options for the homepage"）
   - 请他们在终端里回应："Take a look and let me know what you think. Click to select an option if you'd like."

3. **在你的下一轮** —— 在用户在终端中回应之后：
   - 如果 `$STATE_DIR/events` 存在，读取它——其中以 JSON 行的形式记录了用户在浏览器中的交互（点击、选择）
   - 将其与用户的终端文本合并起来，得到完整信息
   - 终端消息是主要反馈；`state_dir/events` 提供结构化的交互数据

4. **迭代或推进** —— 如果反馈改变了当前屏，写一个新文件（例如 `layout-v2.html`）。只有当前步骤得到验证后才推进到下一个问题。

5. **返回终端时卸载** —— 当下一步不需要浏览器（例如一个澄清问题、一次权衡讨论）时，推送一个等待画面以清除过时内容：

   ```html
   <!-- filename: waiting.html (or waiting-2.html, etc.) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   这可以防止用户盯着一个已经定下来的选择，而对话早已继续前行。当下一个视觉问题出现时，照常推送一个新的内容文件。

6. 重复直到结束。

## 编写内容片段

只写要放进页面里的内容。服务器会自动把它包裹进 frame 模板（头部、主题 CSS、选择指示器，以及所有交互基础设施）。

**最小示例：**

```html
<h2>Which layout works better?</h2>
<p class="subtitle">Consider readability and visual hierarchy</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single Column</h3>
      <p>Clean, focused reading experience</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Two Column</h3>
      <p>Sidebar navigation with main content</p>
    </div>
  </div>
</div>
```

就这样。不需要 `<html>`、不需要 CSS、不需要 `<script>` 标签。服务器全部提供。

## 可用的 CSS 类

frame 模板为你的内容提供以下 CSS 类：

### 选项（A/B/C 选择）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Title</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

**多选：** 向容器添加 `data-multiselect` 让用户可选择多个选项。每次点击切换该项。指示条会显示计数。

```html
<div class="options" data-multiselect>
  <!-- same option markup — users can select/deselect multiple -->
</div>
```

### 卡片（视觉设计）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup content --></div>
    <div class="card-body">
      <h3>Name</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

### Mockup 容器

```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard Layout</div>
  <div class="mockup-body"><!-- your mockup HTML --></div>
</div>
```

### 分屏视图（并排）

```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### 优缺点

```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>Benefit</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>Drawback</li></ul></div>
</div>
```

### 模拟元素（线框图构件）

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">Navigation</div>
  <div class="mock-content">Main content area</div>
</div>
<button class="mock-button">Action Button</button>
<input class="mock-input" placeholder="Input field">
<div class="placeholder">Placeholder area</div>
```

### 排版和段落

- `h2` —— 页面标题
- `h3` —— 区块标题
- `.subtitle` —— 标题下方的次级文本
- `.section` —— 带底部外边距的内容块
- `.label` —— 小号大写字母的标签文本

## 浏览器事件格式

当用户在浏览器中点击选项时，他们的交互会被记录到 `$STATE_DIR/events`（每行一个 JSON 对象）。当你推送新屏时该文件会被自动清空。

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

完整的事件流展示了用户的探索路径——他们可能在定下最终选择前点击多个选项。最后一个 `choice` 事件通常就是最终选择，但点击模式可以揭示犹豫或值得追问的偏好。

如果 `$STATE_DIR/events` 不存在，说明用户没有与浏览器交互——只使用他们的终端文本即可。

## 设计贴士

- **保真度与问题匹配** —— 布局问题用线框图，打磨类问题用精致样式
- **在每一页上解释问题** —— "Which layout feels more professional?" 而不仅是 "Pick one"
- **先迭代再推进** —— 如果反馈改变了当前屏，写一个新版本
- 每屏 **最多 2-4 个选项**
- **在重要时使用真实内容** —— 做一个摄影作品集时，使用真实图片（Unsplash）。占位内容会掩盖设计问题。
- **保持 mockup 简洁** —— 聚焦布局和结构，不追求像素级完美设计

## 文件命名

- 使用语义化名字：`platform.html`、`visual-style.html`、`layout.html`
- 绝不复用文件名——每屏 MUST 是一个新文件
- 迭代时追加版本后缀，如 `layout-v2.html`、`layout-v3.html`
- 服务器按修改时间提供最新文件

## 清理

```bash
scripts/stop-server.sh $SESSION_DIR
```

如果该会话使用了 `--project-dir`，mockup 文件会持久化在 `.superpowers/brainstorm/` 中以便日后参考。只有 `/tmp` 会话在停止时才会被删除。

## 参考

- Frame 模板（CSS 参考）：`scripts/frame-template.html`
- Helper 脚本（客户端）：`scripts/helper.js`
