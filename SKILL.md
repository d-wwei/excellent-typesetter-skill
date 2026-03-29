---
name: typeset
description: 全能编辑器排版 Skill — 将 Markdown 转换为多平台兼容的精美排版 HTML，支持微信公众号/知乎/掘金/博客/X Articles/Medium/LinkedIn。适用于任何 AI Agent。
version: 1.2.0
compatible_agents:
  - Claude Code
  - Codex (OpenAI)
  - Gemini CLI (Google)
  - OpenCode
  - Any agent supporting skill/prompt files
triggers:
  - /typeset
  - 排版
  - 格式化文章
  - 转公众号
  - WeChat format
  - typeset
  - 微信排版
  - 知乎排版
  - 掘金排版
  - Medium 排版
  - LinkedIn 排版
  - X article
---

# Typeset — 全能编辑器排版 Skill

将 Markdown 转换为多平台兼容的精美排版 HTML。适用于任何 AI Agent（Claude Code、Codex、Gemini CLI、OpenCode 等）。融合 mdnice、baoyu、wechat-format、md2wechat 等最佳实践。

## Agent 兼容性

本 Skill 是纯 Markdown 指令集，不依赖任何特定 Agent 的专有 API。

| Agent | 加载方式 |
|-------|---------|
| **Claude Code** | 放入 `~/.claude/skills/` 目录，自动注册为 `/typeset` |
| **Codex (OpenAI)** | 放入项目根目录或通过 `AGENTS.md` 引用 SKILL.md |
| **Gemini CLI** | 通过 `GEMINI.md` 引用或作为 context 文件加载 |
| **OpenCode** | 放入 skill 目录或通过配置文件引用 |
| **其他 Agent** | 将 SKILL.md 内容作为 system prompt 或 context 加载即可 |

**依赖的通用能力**（所有主流 Agent 均具备）：
- 读写文件（读取 Markdown 输入，写出 HTML 输出）
- 运行 shell 命令（可选，用于 MarkItDown 格式转换）
- 读取本 Skill 目录下的主题文件

## 调用格式

```
/typeset <文件路径或内容> [--platform wechat|zhihu|juejin|medium|linkedin|x|blog] [--theme default|elegant|tech|minimal|vibrant] [--humanize] [--code-theme github|one-dark|kimbie]
```

**默认值**: platform=wechat, theme=minimal, code-theme=github

---

## 工作流

### Step 1: 确认输入

1. 如果用户提供了文件路径，检测文件格式：
   - **Markdown (.md)** → 直接使用
   - **其他格式** → 通过 MarkItDown 预处理转换为 Markdown（见下方支持格式表）
2. 如果用户提供了内联文本（Markdown 或纯文本），直接使用
3. 确认目标平台（默认微信公众号）
4. 确认主题（默认 default）
5. 如果用户没有明确指定平台或主题，根据上下文推断或简短确认

#### 输入格式预处理（MarkItDown）

当输入文件不是 Markdown 时，使用 `markitdown` CLI 自动转换：

```bash
markitdown <input_file> -o <output.md>
```

**支持的输入格式**:

| 类别 | 格式 |
|---|---|
| 文档 | PDF, DOCX, PPTX, XLSX, XLS, EPUB |
| 网页 | HTML, RSS |
| 数据 | CSV, JSON, XML |
| 媒体 | 图片 (EXIF + OCR), 音频 (转录) |
| 其他 | Jupyter Notebook (.ipynb), Outlook MSG, ZIP |

**预处理流程**:
1. 检测文件扩展名
2. 运行 `markitdown <file>` 获取 Markdown 输出
3. 将输出保存为临时 `.md` 文件（与源文件同目录，命名 `<原名>.converted.md`）
4. 后续步骤使用转换后的 Markdown 继续

**注意**:
- MarkItDown 需要已安装: `pipx install 'markitdown[all]' --python python3.12`
- 复杂表格布局和数学公式转换精度有限，建议转换后人工检查
- 如果 `markitdown` 命令不可用，提示用户安装并回退到 Claude 的内置文件读取能力（Read 工具支持 PDF 和图片）

### Step 2: 预处理

1. **提取 Frontmatter**: 解析 YAML frontmatter（`---` 分隔），提取 title、author、description、tags
2. **CJK 空格规范化**: 在中文与英文/数字之间添加空格（盘古之白）
3. **标题清理**: 如果目标平台是微信，从正文中移除 H1 标题（微信用 title 字段单独展示）
4. **链接预处理**: 收集所有外部链接。微信平台下移除所有非微信域名的 `<a>` 标签，转为纯文本脚注（上标编号 + 文末 URL 文字）。其他平台保留超链接

### Step 3: 可选 — 人性化处理

如果用户指定 `--humanize` 或明确要求「去 AI 味」，执行人性化处理。详见 [人性化处理](#人性化处理) 章节。

### Step 4: 渲染 HTML

这是核心步骤。按以下顺序生成 HTML：

1. **读取主题定义**: 从 `{{SKILL_DIR}}/themes/<theme>.md` 读取主题样式规范
2. **生成 HTML 结构**: 按 [元素渲染规范](#元素渲染规范) 将每个 Markdown 元素转换为带内联样式的 HTML
3. **应用平台规则**: 按 [平台配置](#平台配置) 调整 HTML 以适配目标平台
4. **组装完整文档**: 生成可在浏览器打开的 HTML 文件

### Step 5: 输出

1. 将 HTML 写入文件（与输入同目录，扩展名 `.html`，或用户指定路径）
2. 报告输出路径
3. 提示用户：「在浏览器中打开 → 全选 → 复制 → 粘贴到编辑器」
4. 如果平台是微信，额外提示图片需要单独上传

---

## 主题系统

### 四层架构（灵感来源: mdnice）

| 层 | 作用 | 说明 |
|---|---|---|
| **基础层 (Base)** | 容器、排版基础 | 所有主题共享，定义 max-width、字体栈、行高 |
| **内容层 (Content)** | 标题、段落、列表、引用等 | 每个主题独立定义，决定视觉风格 |
| **代码层 (Code)** | 代码块、行内代码高亮 | 可独立于内容层切换 |
| **字体层 (Font)** | 字体族覆盖 | 可选，用于微调字体偏好 |

四层分离意味着：切换代码高亮主题不影响内容样式，切换字体不影响配色。

### 内置主题

| 主题 | 风格 | 适用场景 |
|---|---|---|
| `minimal` | 极简、大留白、注重节奏（**默认**） | 通用文章、短文、随笔 |
| `default` | 专业、均衡、蓝色调 | 正式报告、技术文章 |
| `elegant` | 文艺、精致、暖色调 | 人文/商业/叙事类 |
| `tech` | 现代、冷色调、代码友好 | 技术教程、开发文档 |
| `vibrant` | 活力、绿色调、视觉丰富 | 公众号品牌文/营销 |
| `dark` | 深色背景、护眼、沉浸阅读 | 技术社区、夜间阅读 |
| `notion` | Notion 风格、干净利落 | 知识库、内部文档 |
| `newspaper` | 报纸印刷风、衬线经典 | 深度报道、长文评论 |
| `gradient` | 现代渐变色、视觉冲击 | 产品发布、发布会风格 |

主题文件路径: `{{SKILL_DIR}}/themes/<name>.md`

每个主题文件包含两部分：
1. **样式值定义** — 颜色、字体、间距等抽象值
2. **HTML 渲染模板** — 关键元素的精确内联 HTML 模板，渲染时直接复制替换内容

主题对比预览页: `{{SKILL_DIR}}/themes/preview.html`（浏览器打开可查看 5 个主题的视觉效果对比）

### 自定义主题

用户可在 `{{SKILL_DIR}}/themes/` 创建自定义主题 Markdown 文件，格式与内置主题一致。SKILL 按用户指定名称查找主题文件。

---

## 平台配置

### 微信公众号 (wechat)

**核心约束**（编译自 mdnice + wechat-format + baoyu 全量研究）：

1. **所有 CSS 必须内联** — 微信编辑器会剥离 `<style>` 标签和外部样式表。每个 HTML 元素都必须用 `style="..."` 属性承载样式
2. **禁止纯黑色 #000000** — 微信编辑器会吞掉纯黑的 `color` 属性。使用 `rgb(1,1,1)` 或 `#3f3f3f` 代替
3. **列表项包裹 `<section>`** — 微信会重置 `<ul>/<ol>` 子元素样式。每个 `<li>` 内容必须包在 `<section>` 中：`<li><section style="...">内容</section></li>`
4. **禁止 `<pre>` 标签** — 微信对 `<pre>` 渲染异常。代码块改用 `<section>` 容器 + 逐行 `<span>` 标记
5. **禁止非微信域名超链接** — 微信编辑器不接收非 `weixin.qq.com` 域名的 `<a href>` 标签。所有外部链接必须转为纯文本展示（上标编号 `[n]` + 文末参考文献区列出 URL 文字），**不使用任何 `<a>` 标签包裹外部 URL**。仅微信域内链接可保留 `<a>` 标签
6. **表格需滚动容器** — 用 `<section style="overflow-x: auto; -webkit-overflow-scrolling: touch;">` 包裹 `<table>`
7. **亚像素边框用 transform** — 微信不支持小数边框宽度，用 `transform: scale(0.5)` 实现 0.5px 线
8. **标题装饰用 span 三段式** — 每个标题内注入 `<span class="prefix">` + `<span class="content">` + `<span class="suffix">`，通过内联样式实现装饰效果（编号、下划线、图标），无需自定义 Markdown 语法
9. **数学公式必须 SVG 或图片** — 微信无 MathJax/KaTeX 支持。公式转为内联 SVG，设置 `width` 和 `height` 为内联样式（非属性）
10. **图片必须微信 CDN** — 非微信 CDN 的图片会被替换为占位符。正文图片需上传到微信素材库，URL 会过期需每次重新上传
11. **容器命名空间隔离** — 所有内容包在 `<section id="nice" style="...">` 中，防止样式泄漏
12. **微信安全 HTML 标签白名单**: `section`, `span`, `p`, `strong`, `em`, `a`, `img`, `table`, `thead`, `tbody`, `tr`, `th`, `td`, `ul`, `ol`, `li`, `blockquote`, `h1`-`h6`, `br`, `hr`, `sup`, `sub`, `figcaption`, `figure`
13. **H1 标题从正文移除** — 微信用自己的标题字段，正文 H1 会重复
14. **`<em>` 渲染为粗体** — 微信中斜体中文字体渲染差，`<em>` 也用 `font-weight: bold` + 颜色区分

### 知乎 (zhihu)

1. **CSS 内联** — 与微信类似，知乎也会剥离外部样式
2. **链接可保留** — 知乎支持可点击外部链接，不需要脚注化
3. **公式用 `<img data-eeimg>`** — 知乎有专用公式标签，LaTeX 作为 alt 文本
4. **代码块保留 `<pre><code>`** — 知乎正常支持 `<pre>` 标签
5. **图片可用外部链接** — 知乎会自动代理外部图片

### 掘金 (juejin)

1. **CSS 内联** — 掘金也剥离外部样式
2. **链接可保留** — 掘金支持外部链接
3. **公式用图片 URL** — 掘金不支持 MathJax，公式转为外部渲染服务图片
4. **代码块**: `<pre><code>` + `display: block` + 换行符规范化
5. **图片**: 掘金有自己的图片上传服务，外部图片可能被替换

### Medium (medium)

1. **内联 CSS 为主** — Medium 粘贴时会保留部分内联样式，但会用自己的字体和间距覆盖。重点放在结构正确而非精确样式控制
2. **链接保留** — Medium 完整支持外部链接，不需要脚注化
3. **标题层级限制** — Medium 实际只支持两级标题（大标题 + 小标题），H1 映射为 Medium 大标题，H2 映射为小标题，H3+ 用粗体段落模拟
4. **代码块用 `<pre>`** — Medium 支持代码块，但**无语法高亮**。代码块以等宽字体灰底显示，不要内联着色
5. **图片可用外部链接** — Medium 会自动代理并重新托管外部图片
6. **引用块样式由 Medium 控制** — `<blockquote>` 保持标准结构即可，Medium 有自己的左边框样式
7. **无表格支持** — Medium 不支持 `<table>`。表格转为格式化文本列表，或提示用户截图替代
8. **分隔线** — 使用 `<hr>`，Medium 会渲染为三个居中圆点
9. **建议主题** — `minimal` 或 `elegant`（Medium 自带排版很干净，过度装饰的主题会被覆盖）
10. **粘贴方式** — 在浏览器打开 HTML → 全选复制 → 在 Medium 编辑器里粘贴（Medium 会解析 HTML 并转为自己的格式）

### LinkedIn (linkedin)

1. **内联 CSS 大部分被剥离** — LinkedIn 文章编辑器比较激进地剥离样式，重点放在语义结构正确
2. **链接保留** — LinkedIn 支持外部链接
3. **标题** — LinkedIn 支持 H1 和 H2，H3+ 用粗体段落替代
4. **无代码块支持** — LinkedIn 不支持 `<pre><code>`，代码会变成纯文本。长代码建议截图替代，短代码用行内等宽字体
5. **图片需手动上传** — LinkedIn 不支持从 HTML 粘贴图片，需在编辑器中手动插入
6. **无表格支持** — 与 Medium 类似，表格转为格式化列表或截图
7. **列表支持** — `<ul>` 和 `<ol>` 正常工作
8. **引用块** — LinkedIn 支持 `<blockquote>`，会以灰色左边框显示
9. **字数参考** — LinkedIn 文章最多约 125,000 字符，但阅读体验最佳长度 1,000-2,000 字
10. **建议主题** — `minimal`（LinkedIn 编辑器自身样式很简洁，花哨主题无效）
11. **粘贴方式** — 在浏览器打开 HTML → 全选复制 → 在 LinkedIn 文章编辑器里粘贴

### X Articles (x)

1. **X Articles 是 Premium 功能** — 需要 X Premium 订阅才能发布长文
2. **富文本编辑器** — X Articles 有自己的渲染引擎，粘贴 HTML 后会转换为内部格式
3. **链接保留** — 外部链接完整可点击
4. **标题** — 支持 H1、H2、H3
5. **代码块** — 支持等宽字体代码块，无语法高亮，保持 `<pre><code>` 结构
6. **图片** — 支持从 HTML 粘贴图片，也可手动上传。外部图片 URL 可能需要重新上传
7. **表格** — 有限支持，复杂表格建议截图
8. **引用块** — 标准 `<blockquote>` 支持
9. **列表** — `<ul>` 和 `<ol>` 正常工作
10. **内联 CSS** — 大部分被覆盖，X 使用自己的排版。关注结构而非样式
11. **建议主题** — `default` 或 `tech`（X Articles 的渲染风格偏科技感）
12. **粘贴方式** — 在浏览器打开 HTML → 全选复制 → 在 X Articles 编辑器里粘贴

### 通用博客 (blog)

1. **完整 HTML 文档** — 生成带 `<!DOCTYPE>`, `<head>`, `<body>` 的完整 HTML
2. **可用 `<style>` 标签** — 不需要全部内联，可保留嵌入式样式表
3. **链接正常** — 保留所有超链接
4. **公式用 KaTeX** — 引入 KaTeX CSS/JS，使用标准 LaTeX 语法
5. **Mermaid 支持** — 引入 Mermaid.js，代码块保留为 `<div class="mermaid">`
6. **语法高亮**: 可引入 highlight.js 或 Prism.js

---

## 元素渲染规范

以下规范定义了每个 Markdown 元素转换为 HTML 时的结构和内联样式。**具体颜色和字体值从当前主题读取**，此处用 `{theme.xxx}` 占位。

### 容器

```html
<section id="nice" style="max-width: {theme.container.max-width}; margin: 0 auto; padding: {theme.container.padding}; background: {theme.container.background}; color: {theme.text.color}; font-family: {theme.font.family}; font-size: {theme.font.size}; line-height: {theme.font.line-height}; word-break: break-word; overflow-wrap: break-word;">
  <!-- 内容 -->
</section>
```

### 段落

```html
<p style="margin: {theme.paragraph.margin}; padding: 0; color: {theme.text.color}; font-size: {theme.font.size}; line-height: {theme.font.line-height}; letter-spacing: {theme.text.letter-spacing};">
  段落文本
</p>
```

### 标题 (H1-H6)

微信公众号使用三段式装饰结构（来源: mdnice）：

```html
<h2 style="{theme.h2.style}">
  <span style="{theme.h2.prefix-style}"></span>
  <span style="{theme.h2.content-style}">标题文本</span>
  <span style="{theme.h2.suffix-style}"></span>
</h2>
```

- H1: 仅用于博客平台；微信平台下从正文移除
- H2: 主要分节标题，视觉最强
- H3: 二级分节
- H4-H6: 递减，简化处理

### 强调

```html
<!-- 粗体 -->
<strong style="color: {theme.strong.color}; font-weight: bold;">粗体文本</strong>

<!-- 斜体（微信平台下也渲染为粗体+颜色区分） -->
<em style="font-style: italic; color: {theme.em.color};">
  <!-- 微信: font-style: normal; font-weight: bold; color: {theme.em.color}; -->
  斜体文本
</em>

<!-- 高亮 ==文本== -->
<span style="background: {theme.highlight.background}; padding: 2px 4px; border-radius: 2px;">高亮文本</span>

<!-- 删除线 -->
<del style="text-decoration: line-through; color: {theme.text.secondary-color};">删除文本</del>
```

### 列表

微信平台下列表项必须包裹 `<section>`（来源: mdnice + wechat-format）：

```html
<!-- 无序列表 -->
<ul style="margin: {theme.list.margin}; padding-left: {theme.list.padding-left}; list-style-type: disc;">
  <li style="margin-bottom: {theme.list.item-spacing};">
    <section style="color: {theme.text.color}; font-size: {theme.font.size}; line-height: {theme.font.line-height};">
      列表项内容
    </section>
  </li>
</ul>

<!-- 有序列表 -->
<ol style="margin: {theme.list.margin}; padding-left: {theme.list.padding-left}; list-style-type: decimal;">
  <li style="margin-bottom: {theme.list.item-spacing};">
    <section style="color: {theme.text.color}; font-size: {theme.font.size}; line-height: {theme.font.line-height};">
      列表项内容
    </section>
  </li>
</ol>
```

**微信额外规则**: 如果原生列表样式被重置（常见），改用自定义字符方案：
- 无序列表用 `•` 文本字符 + `padding-left`
- 有序列表用数字文本前缀 `1.` + `padding-left`

### 引用块

```html
<blockquote style="margin: {theme.blockquote.margin}; padding: {theme.blockquote.padding}; border-left: {theme.blockquote.border-left}; background: {theme.blockquote.background}; color: {theme.blockquote.color};">
  <p style="margin: 0; font-size: {theme.blockquote.font-size}; line-height: {theme.font.line-height};">
    引用内容
  </p>
</blockquote>
```

支持多层嵌套引用（来源: mdnice），通过递增缩进和递减透明度区分层级。

### 代码块

**微信平台**（来源: mdnice + wechat-format，禁止 `<pre>`）：

```html
<section style="margin: {theme.code-block.margin}; border-radius: {theme.code-block.border-radius}; overflow: hidden; background: {theme.code-block.background};">
  <!-- 可选: macOS 窗口装饰（来源: baoyu mac-sign） -->
  <section style="padding: 8px 16px; display: flex; align-items: center; gap: 6px;">
    <span style="width: 10px; height: 10px; border-radius: 50%; background: #ff5f57; display: inline-block;"></span>
    <span style="width: 10px; height: 10px; border-radius: 50%; background: #febc2e; display: inline-block;"></span>
    <span style="width: 10px; height: 10px; border-radius: 50%; background: #28c840; display: inline-block;"></span>
    <span style="flex: 1;"></span>
    <span style="font-size: 12px; color: #999; font-family: monospace;">language</span>
  </section>
  <!-- 代码内容 -->
  <section style="padding: 12px 16px; overflow-x: auto; font-family: 'Menlo', 'Monaco', 'Consolas', 'Courier New', monospace; font-size: {theme.code-block.font-size}; line-height: 1.6;">
    <span style="color: {syntax.keyword};">const</span> <span style="color: {syntax.variable};">name</span> <span style="color: {syntax.operator};">=</span> <span style="color: {syntax.string};">"hello"</span><br>
    <!-- 逐行逐 token 着色 -->
  </section>
</section>
```

**博客平台**: 可使用标准 `<pre><code>` + highlight.js。

### 行内代码

```html
<code style="background: {theme.inline-code.background}; color: {theme.inline-code.color}; padding: 2px 6px; border-radius: {theme.inline-code.border-radius}; font-family: 'Menlo', 'Monaco', 'Consolas', monospace; font-size: 0.9em;">
  code
</code>
```

### 表格

```html
<!-- 微信: 滚动容器 -->
<section style="overflow-x: auto; -webkit-overflow-scrolling: touch; margin: 16px 0;">
  <table style="border-collapse: collapse; width: 100%; min-width: 400px; font-size: {theme.table.font-size};">
    <thead>
      <tr style="background: {theme.table.header-background};">
        <th style="border: 1px solid {theme.table.border-color}; padding: {theme.table.cell-padding}; text-align: left; font-weight: bold; color: {theme.table.header-color};">
          表头
        </th>
      </tr>
    </thead>
    <tbody>
      <tr style="background: {theme.table.row-even-background};">
        <td style="border: 1px solid {theme.table.border-color}; padding: {theme.table.cell-padding}; color: {theme.text.color};">
          内容
        </td>
      </tr>
      <!-- 交替行背景色: odd=白, even=浅灰 -->
    </tbody>
  </table>
</section>
```

### 图片

```html
<figure style="margin: 20px 0; text-align: center;">
  <img src="图片URL" alt="描述" style="max-width: 100%; border-radius: {theme.image.border-radius}; display: block; margin: 0 auto;">
  <!-- 可选图注 -->
  <figcaption style="margin-top: 8px; font-size: 13px; color: {theme.text.secondary-color}; text-align: center;">
    图注文字
  </figcaption>
</figure>
```

### 水平线

```html
<hr style="border: none; height: 1px; background: {theme.hr.color}; margin: {theme.hr.margin}; transform: scale(1, 0.5);">
```

微信下用 `transform: scale(1, 0.5)` 实现 0.5px 视觉效果（来源: wechat-format）。

### 链接

#### 微信公众号 — 纯文本脚注（无超链接）

**核心规则**: 微信编辑器不接收非 `weixin.qq.com` 域名的超链接。**所有外部 URL 不得使用 `<a>` 标签**，必须以纯文本形式展示。

渲染方式：正文中用上标编号标记，文末参考文献区以纯文本列出 URL。

```html
<!-- 正文中：纯文本 + 上标编号（无 <a> 标签） -->
<span style="color: {theme.text.color};">链接文字</span><sup style="font-size: 0.7em; color: {theme.footnote.color}; vertical-align: super;">[1]</sup>

<!-- 文末参考文献区：URL 为纯文本（无 <a> 标签） -->
<section style="margin-top: 40px; padding-top: 16px; border-top: 1px solid {theme.hr.color};">
  <p style="font-size: 14px; font-weight: bold; color: {theme.text.secondary-color}; margin-bottom: 8px;">参考文献</p>
  <p style="font-size: 13px; color: {theme.text.secondary-color}; margin: 4px 0; word-break: break-all;">
    [1] 链接文字: https://example.com/url
  </p>
</section>
```

**规则**:
- 微信域内链接（`mp.weixin.qq.com`、`weixin.qq.com`）是唯一例外，可保留 `<a>` 标签
- 所有其他域名的 URL：**禁止 `<a>` 标签**，以纯文本展示
- href 等于文字的链接直接显示为纯文本 URL
- 上标编号 `[n]` 也用纯文本 `<sup>` 而非 `<a>` 标签

#### 其他平台

- **知乎/掘金/Medium/LinkedIn/X**: 链接保留为正常 `<a>` 标签
- **博客**: 保留所有链接为正常 `<a>` 标签

### Callout / 提示框

支持两种语法（来源: baoyu + obsidian）：

**GFM 语法**: `> [!NOTE]`, `> [!TIP]`, `> [!WARNING]`, `> [!CAUTION]`, `> [!IMPORTANT]`

**Obsidian 语法**: `:::note`, `:::warning`, `:::tip` 等

13 种标准类型（来源: obsidian-markdown）：
`note`, `abstract`, `info`, `todo`, `tip`, `success`, `question`, `warning`, `failure`, `danger`, `bug`, `example`, `quote`

**渲染结构**:

```html
<section style="margin: 16px 0; padding: 12px 16px; border-left: 4px solid {callout.border-color}; background: {callout.background}; border-radius: 0 4px 4px 0;">
  <p style="margin: 0 0 8px 0; font-weight: bold; color: {callout.title-color}; font-size: 15px;">
    {callout.icon} {callout.title}
  </p>
  <section style="color: {theme.text.color}; font-size: {theme.font.size};">
    内容
  </section>
</section>
```

**Callout 颜色映射**:

| 类型 | 边框/标题色 | 背景色 | 图标 |
|---|---|---|---|
| note | #448aff | #e8f0fe | |
| tip / important | #00bfa5 | #e6f9f0 | |
| warning / caution | #ff9100 | #fff8e1 | |
| danger / error | #ff5252 | #fde8e8 | |
| info | #2196f3 | #e3f2fd | |
| success | #4caf50 | #e8f5e9 | |
| question | #9c27b0 | #f3e5f5 | |
| bug | #f44336 | #fde8e8 | |
| example | #7c4dff | #ede7f6 | |
| quote | #9e9e9e | #f5f5f5 | |

**注意**: 微信平台下 Callout 图标（emoji）可能被过滤。使用文字标签替代：「注意」「提示」「警告」等。

### 数学公式

**微信平台**: 公式必须转为 SVG 或图片（来源: mdnice）。
- 行内公式 `$...$`: 生成小型 SVG，用 `<span>` 包裹
- 块级公式 `$$...$$`: 生成 SVG，居中展示

**实际操作**: 由于 Claude 无法直接运行 KaTeX/MathJax，处理策略：
1. 简单公式：用 HTML 上标/下标和特殊字符近似表达
2. 复杂公式：保留 LaTeX 源码并标注「此公式需要手动渲染」
3. 博客平台：保留 LaTeX 语法，引入 KaTeX JS/CSS

### Mermaid / PlantUML 图表

**微信平台**: 不支持 JS 运行时。处理策略：
1. 标注「此图表需要手动截图替换」
2. 如果有 PlantUML 服务器可用，生成图片 URL
3. 提供替代方案：将流程描述为文字列表

**博客平台**: 保留 Mermaid 代码块，引入 Mermaid.js。

### 脚注

```markdown
文本[^1]

[^1]: 脚注内容
```

**微信平台**渲染为纯文本脚注（无 `<a>` 标签）：

```html
<!-- 正文引用 -->
<sup style="font-size: 0.7em; color: {theme.footnote.color};">[1]</sup>

<!-- 文末定义 -->
<section style="margin-top: 40px; padding-top: 16px; border-top: 1px solid {theme.hr.color};">
  <p style="font-size: 13px; color: {theme.text.secondary-color}; margin: 4px 0;">
    [1] 脚注内容
  </p>
</section>
```

**博客/其他平台**渲染为双向链接（来源: baoyu）：

```html
<!-- 正文引用 -->
<sup id="fnRef-1" style="font-size: 0.7em;"><a href="#fnDef-1" style="color: {theme.footnote.color}; text-decoration: none;">[1]</a></sup>

<!-- 文末定义 -->
<section style="margin-top: 40px; padding-top: 16px; border-top: 1px solid {theme.hr.color};">
  <p id="fnDef-1" style="font-size: 13px; color: {theme.text.secondary-color}; margin: 4px 0;">
    <a href="#fnRef-1" style="color: {theme.footnote.color}; text-decoration: none;">[1]</a> 脚注内容
  </p>
</section>
```

---

## 代码高亮

### 内联语法着色方案

由于微信不支持外部 JS，代码高亮必须通过内联 `color` 样式实现。Claude 根据语言识别 token 类型并应用对应颜色。

### 内置代码主题

#### GitHub Light (默认)

| Token 类型 | 颜色 |
|---|---|
| keyword | #d73a49 |
| string | #032f62 |
| comment | #6a737d |
| function | #6f42c1 |
| number | #005cc5 |
| operator | #d73a49 |
| variable | #e36209 |
| type | #6f42c1 |
| property | #005cc5 |
| tag | #22863a |
| attribute | #6f42c1 |
| 背景色 | #f6f8fa |
| 文字色 | #24292e |

#### One Dark

| Token 类型 | 颜色 |
|---|---|
| keyword | #c678dd |
| string | #98c379 |
| comment | #5c6370 |
| function | #61afef |
| number | #d19a66 |
| operator | #56b6c2 |
| variable | #e06c75 |
| type | #e5c07b |
| property | #e06c75 |
| tag | #e06c75 |
| attribute | #d19a66 |
| 背景色 | #282c34 |
| 文字色 | #abb2bf |

#### Kimbie Light

| Token 类型 | 颜色 |
|---|---|
| keyword | #98676a |
| string | #889b4a |
| comment | #a57a4c |
| function | #f79a32 |
| number | #f79a32 |
| operator | #98676a |
| variable | #dc3958 |
| type | #f06431 |
| property | #f06431 |
| tag | #dc3958 |
| attribute | #f79a32 |
| 背景色 | #fbebd4 |
| 文字色 | #84613d |

### Token 识别规则

Claude 在生成代码块时按以下规则识别 token 类型：

- **keyword**: 语言保留字（`const`, `let`, `var`, `function`, `class`, `if`, `else`, `for`, `while`, `return`, `import`, `export`, `from`, `def`, `async`, `await` 等）
- **string**: 引号包裹的文本（单引号、双引号、反引号、三引号）
- **comment**: `//`, `/* */`, `#`, `<!-- -->`, `"""` 等
- **function**: 函数名（定义和调用）
- **number**: 数字字面量
- **operator**: `=`, `+`, `-`, `*`, `/`, `===`, `!==`, `=>`, `&&`, `||` 等
- **variable**: 变量名（非关键字、非函数）
- **type**: 类型名（大写开头、类型注解）
- **property**: 对象属性（`.` 后的标识符）
- **tag**: HTML/JSX 标签名
- **attribute**: HTML/JSX 属性名

---

## 人性化处理

可选模块，来源: md2wechat humanizer。当用户指定 `--humanize` 时激活。

### 24 种 AI 模式检测（5 大类）

#### 内容类
1. **过度强调** — 不必要的「非常」「极其」「令人惊叹」
2. **夸大其词** — 将普通事物描述得过于重大
3. **推销语气** — 像广告文案而非正常表达
4. **模糊归因** — 「研究表明」「专家认为」无具体来源

#### 语言类
5. **AI 词汇** — 「赋能」「助力」「深入探讨」「至关重要」「不可或缺」
6. **否定并行** — 「不是 X，而是 Y」反复使用
7. **三段式结构** — 每次都是三个并列（三个好处、三个方面、三个建议）
8. **同义词轮换** — 不自然地交替使用近义词避免重复

#### 风格类
9. **破折号滥用** — 过多使用「——」作为插入语
10. **加粗滥用** — 大量 **粗体** 标记，失去强调效果
11. **Emoji 装饰** — 不必要的表情符号点缀

#### 填充类
12. **垫话短语** — 「值得注意的是」「需要指出的是」「毫无疑问」
13. **过度限定** — 每个观点都加「当然也要考虑」「但这并不意味着」
14. **万能结尾** — 「总之」「综上所述」+ 空洞的展望

#### 协作类
15. **对话式填充** — 「这是个好问题」「让我来解释」
16. **知识截止声明** — 「截至我所知」「根据我的训练数据」

### 三个强度级别

| 级别 | 检测阈值 | 修改力度 | 适用场景 |
|---|---|---|---|
| gentle | 仅明显痕迹 | 最小改动 | 已经较自然的文本 |
| medium (默认) | 中等痕迹 | 平衡改动 | 通用 |
| aggressive | 细微痕迹 | 大幅重写 | AI 痕迹很重的文本 |

### 处理流程

1. 扫描文本，按 5 大类 24 种模式逐一检测
2. 对每个检测到的模式标记位置和严重程度
3. 按当前强度级别决定是否修改
4. 执行修改：替换 AI 词汇、简化结构、移除填充
5. 输出修改报告（可选 `--show-changes`）

### 五维质量评分（各 /10，总分 /50）

| 维度 | 说明 |
|---|---|
| 直接性 | 是否直入主题，不绕弯子 |
| 节奏感 | 句子长短交替，不单调 |
| 信任度 | 读起来是否像真人写的 |
| 真实性 | 是否有个人视角和具体细节 |
| 精炼度 | 是否没有废话和水分 |

---

## 输出格式

### HTML 文件结构

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{文章标题}</title>
  <style>
    /* 仅博客平台使用嵌入式样式 */
    body { margin: 0; padding: 20px; background: #f5f5f5; }
    #nice { /* 容器样式 */ }
  </style>
</head>
<body>
  <section id="nice" style="...">
    <!-- 渲染后的 HTML 内容 -->
  </section>
</body>
</html>
```

**微信/知乎/掘金平台**: `<style>` 标签仅用于本地预览。实际粘贴到编辑器时只有 `<section id="nice">` 内的内联样式生效。

### 输出文件命名

- 输入: `article.md` → 输出: `article.html`
- 输入: 内联文本 → 输出: `typeset-output.html`（写入当前目录）

### 完成提示模板

```
排版完成！

- 输出文件: {path}
- 主题: {theme}
- 平台: {platform}
- 字数: {word_count}

下一步:
1. 在浏览器中打开 HTML 文件预览效果
2. 全选 (Cmd+A) → 复制 (Cmd+C) → 粘贴到{platform}编辑器
{3. 注意: 文章中的图片需要在{platform}编辑器中重新上传 (if applicable)}
```

---

## 最佳实践

1. **先确认平台再选主题** — 不同平台的约束不同，主题效果也不同
2. **技术文章用 tech 主题** — 代码块视觉效果最优
3. **CJK 空格自动处理** — 不需要用户手动添加半角空格
4. **代码块限制宽度** — 超长代码行会导致横向滚动，建议 80 字符换行
5. **图片在微信中会独占一行** — 行内图文混排在微信中不可靠
6. **人性化处理建议在排版前执行** — 修改文本后再渲染样式
7. **微信链接处理** — 微信平台下所有非微信域名的 URL 禁止使用 `<a>` 标签，自动转为纯文本脚注（上标编号 + 文末 URL 文字）

---

## 来源声明

本 Skill 通过 Remix 方法论融合以下项目的最佳实践：

| 来源 | 主要贡献 |
|---|---|
| mdnice/markdown-nice | 四层主题架构、标题三段式、WeChat CSS 技巧、多平台支持 |
| baoyu-markdown-to-html | 扩展架构（Callout/脚注/代码高亮）、CJK 预处理、CSS 内联管道 |
| wechat-format | 渲染时直接内联样式、列表兼容方案、亚像素边框技巧 |
| md2wechat-skill | 人性化模块（24 模式/5 维评分）、图片处理流程 |
| wechat-article-formatter | 微信 API 发布流程、bm.md 渲染参考 |
| obsidian-markdown | Callout 分类法（13 类型）、Frontmatter 规范 |
