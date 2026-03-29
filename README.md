# Excellent Typesetter

An AI agent skill that converts Markdown to publication-ready HTML for 7 platforms. Works with Claude Code, Codex, Gemini CLI, OpenCode, and any agent that can read prompt files. One command, inline CSS, paste and publish.

一个通用 AI Agent Skill，把 Markdown 转成可直接发布的 HTML。兼容 Claude Code、Codex、Gemini CLI、OpenCode 等所有主流 Agent。7 个平台，内联 CSS，粘贴即发。

---

[English](#english) | [中文](#中文)

---

## English

### The Problem

You wrote a great article in Markdown. Now you need to publish it on WeChat Official Account — but WeChat strips `<style>` tags, kills external links, resets list styles, and swallows `#000000` colors. You spend 30 minutes fighting the editor.

Then you need the same article on Medium. Different rules. LinkedIn? Different again.

**Excellent Typesetter** handles the platform quirks so you don't have to. Write once in Markdown, typeset for any platform.

### Quick Start

```bash
# In any AI agent, just say:
/typeset article.md --platform wechat --theme default

# Or for Medium:
/typeset article.md --platform medium --theme elegant

# Or from a PDF:
/typeset report.pdf --platform linkedin --theme minimal
```

The skill outputs an HTML file. Open it in your browser, Cmd+A, Cmd+C, paste into your editor. Done.

### What You Can Do

- **Typeset for 7 platforms** — WeChat, Zhihu, Juejin, Medium, LinkedIn, X Articles, and generic blog. Each platform has its own profile handling CSS constraints, link behavior, image rules, and element support.
- **Pick from 5 built-in themes** — default (professional blue), elegant (warm serif), tech (dark code blocks), minimal (Swiss whitespace), vibrant (green energy). Each theme includes exact HTML templates for consistent rendering.
- **Feed it 16+ file formats** — Markdown, PDF, DOCX, PPTX, XLSX, HTML, CSV, EPUB, images (OCR), and more. Non-Markdown files are auto-converted via [MarkItDown](https://github.com/microsoft/markitdown).
- **Syntax-highlighted code blocks** — 3 code themes (GitHub Light, One Dark, Kimbie Light) with per-token inline coloring. No JavaScript dependency — works in WeChat.
- **Remove AI writing patterns** — Optional `--humanize` flag detects 24 AI patterns across 5 categories and rewrites them. Three intensity levels: gentle, medium, aggressive.
- **Auto link-to-footnote conversion** — External links become numbered references with a bibliography section. WeChat-internal links stay clickable.

### Platforms

| Platform | Flag | Links | Code Blocks | Tables | Key Constraint |
|----------|------|-------|-------------|--------|----------------|
| WeChat | `wechat` | Footnotes | `<section>` (no `<pre>`) | Scroll container | All CSS must be inline |
| Zhihu | `zhihu` | Preserved | `<pre><code>` | Standard | Inline CSS |
| Juejin | `juejin` | Preserved | `<pre><code>` | Standard | Inline CSS |
| Medium | `medium` | Preserved | `<pre>` (no highlighting) | Not supported | Medium overrides styles |
| LinkedIn | `linkedin` | Preserved | Not supported | Not supported | Most CSS stripped |
| X Articles | `x` | Preserved | `<pre><code>` | Limited | X Premium required |
| Blog | `blog` | Preserved | highlight.js | Standard | Full HTML document |

### Themes

| Theme | Style | Best For |
|-------|-------|---------|
| `default` | Professional blue, vertical bar headings | General articles |
| `elegant` | Warm serif, dark badge headings | Essays, business writing |
| `tech` | Cool teal, dark code blocks (One Dark) | Technical tutorials |
| `minimal` | Swiss typography, maximum whitespace | Short-form, thought pieces |
| `vibrant` | Green energy, black label headings | Brand content, marketing |

Preview all 5 themes side by side: open `themes/preview.html` in your browser.

### 14 WeChat Compatibility Rules

Compiled from [mdnice](https://github.com/mdnice/markdown-nice) (4.6k stars), [wechat-format](https://github.com/lyricat/wechat-format) (4.5k stars), [baoyu-markdown-to-html](https://github.com/jimliu/baoyu-skills) (11.9k stars), and [md2wechat-skill](https://github.com/geekjourneyx/md2wechat-skill):

1. All CSS inline — no `<style>` tags
2. No `#000000` — use `#3f3f3f`
3. List items wrapped in `<section>`
4. No `<pre>` — code blocks use `<section>` containers
5. External links converted to footnotes
6. Tables in scroll containers
7. Sub-pixel borders via `transform: scale()`
8. Heading decoration via prefix/content/suffix spans
9. Math as SVG or images
10. Images must use WeChat CDN
11. Container namespace isolation (`#nice`)
12. Safe HTML tag whitelist only
13. H1 removed from body (WeChat uses title field)
14. `<em>` rendered as bold (italic CJK renders poorly)

### Installation

**Prerequisites:**
- Any AI agent (Claude Code, Codex, Gemini CLI, OpenCode, etc.)
- Python 3.12+ with [MarkItDown](https://github.com/microsoft/markitdown) (optional, for non-Markdown input)

```bash
# Clone the repo
git clone https://github.com/d-wwei/excellent-typesetter.git

# Optional: install MarkItDown for PDF/DOCX/PPTX support
pipx install 'markitdown[all]' --python python3.12
```

**Per-agent setup:**

| Agent | Setup |
|-------|-------|
| Claude Code | `ln -s /path/to/excellent-typesetter ~/.claude/skills/typeset` — auto-registers as `/typeset` |
| Codex | Copy `SKILL.md` to project root, or reference in `AGENTS.md` |
| Gemini CLI | Reference `SKILL.md` in `GEMINI.md` |
| OpenCode | Add to your skill directory or config |
| Other | Load `SKILL.md` as system prompt or context file |

### File Structure

```
excellent-typesetter/
├── SKILL.md                  # Core: workflow, platform rules, rendering spec
├── README.md                 # This file
├── remix-provenance.md       # Remix audit trail (7 source projects)
└── themes/
    ├── default.md            # CSS values + HTML templates
    ├── elegant.md
    ├── tech.md
    ├── minimal.md
    ├── vibrant.md
    └── preview.html          # Side-by-side theme comparison
```

### Provenance

Built using the [Remix](https://github.com/d-wwei/remix) methodology, synthesizing the best practices from 7 open-source projects:

| Source | Contribution |
|--------|-------------|
| mdnice/markdown-nice | 4-layer theme architecture, heading decoration, WeChat CSS tricks |
| baoyu-markdown-to-html | Extension architecture (callouts, footnotes, code highlighting, CJK) |
| lyricat/wechat-format | Inline style generation during rendering, list compatibility |
| geekjourneyx/md2wechat-skill | Humanizer module (24 patterns, 5 dimensions) |
| iamzifei/wechat-article-formatter | WeChat API publishing reference |
| kepano/obsidian-skills | Callout taxonomy (13 types) |
| microsoft/markitdown | Input format preprocessing (16+ formats) |

### License

MIT

---

## 中文

### 问题

你用 Markdown 写了一篇好文章。现在要发微信公众号——但微信会剥离 `<style>` 标签、外部链接不可点击、列表样式被重置、纯黑色 `#000000` 被吞掉。你花 30 分钟跟编辑器搏斗。

然后同一篇文章要发 Medium。规则不同。LinkedIn？又不一样。

**Excellent Typesetter** 帮你处理平台差异。Markdown 写一次，排版发任何平台。

### 快速开始

```bash
# 在任何 AI Agent 中直接说：
/typeset 文章.md --platform wechat --theme default

# 发 Medium：
/typeset 文章.md --platform medium --theme elegant

# 从 PDF 排版：
/typeset 报告.pdf --platform linkedin --theme minimal
```

输出一个 HTML 文件。浏览器打开，Cmd+A 全选，Cmd+C 复制，粘贴到编辑器。搞定。

### 核心能力

- **7 个平台** — 微信公众号、知乎、掘金、Medium、LinkedIn、X Articles、通用博客。每个平台有独立的 CSS 约束、链接处理、图片规则配置。
- **5 个内置主题** — default（专业蓝）、elegant（文艺暖）、tech（技术冷）、minimal（极简白）、vibrant（活力绿）。每个主题包含精确的 HTML 渲染模板。
- **16+ 输入格式** — Markdown、PDF、DOCX、PPTX、XLSX、HTML、CSV、EPUB、图片（OCR）等。非 Markdown 文件通过 [MarkItDown](https://github.com/microsoft/markitdown) 自动转换。
- **代码语法高亮** — 3 套代码主题（GitHub Light、One Dark、Kimbie Light），逐 token 内联着色。无 JS 依赖——微信里也能用。
- **去 AI 味** — 可选 `--humanize` 标志，检测 5 大类 24 种 AI 写作模式并重写。三个强度：gentle、medium、aggressive。
- **链接自动脚注化** — 外部链接转为编号引用 + 文末参考文献。微信域内链接保持可点击。

### 平台支持

| 平台 | 参数 | 链接 | 代码块 | 表格 | 核心约束 |
|------|------|------|--------|------|----------|
| 微信公众号 | `wechat` | 转脚注 | `<section>`（禁 `<pre>`） | 滚动容器 | 全部 CSS 内联 |
| 知乎 | `zhihu` | 保留 | `<pre><code>` | 正常 | CSS 内联 |
| 掘金 | `juejin` | 保留 | `<pre><code>` | 正常 | CSS 内联 |
| Medium | `medium` | 保留 | `<pre>`（无高亮） | 不支持 | 样式被覆盖 |
| LinkedIn | `linkedin` | 保留 | 不支持 | 不支持 | 大部分 CSS 被剥离 |
| X Articles | `x` | 保留 | `<pre><code>` | 有限 | 需 X Premium |
| 博客 | `blog` | 保留 | highlight.js | 正常 | 完整 HTML 文档 |

### 主题

| 主题 | 风格 | 适用场景 |
|------|------|----------|
| `default` | 专业蓝，竖条标题装饰 | 通用文章 |
| `elegant` | 宋体衬线，深色徽章标题 | 人文/商业/叙事 |
| `tech` | 冷青色调，深色代码块 (One Dark) | 技术教程 |
| `minimal` | 瑞士排版，极致留白 | 短文/随笔/思考 |
| `vibrant` | 活力绿，黑底标签标题 | 品牌文/营销 |

打开 `themes/preview.html` 可以在浏览器里对比 5 个主题的视觉效果。

### 14 条微信兼容规则

编译自 [mdnice](https://github.com/mdnice/markdown-nice)（4.6k stars）、[wechat-format](https://github.com/lyricat/wechat-format)（4.5k stars）、[baoyu-markdown-to-html](https://github.com/jimliu/baoyu-skills)（11.9k stars）、[md2wechat-skill](https://github.com/geekjourneyx/md2wechat-skill)：

1. 全部 CSS 内联——不用 `<style>` 标签
2. 禁止 `#000000`——用 `#3f3f3f` 代替
3. 列表项包裹 `<section>`
4. 禁止 `<pre>`——代码块用 `<section>` 容器
5. 外部链接转脚注
6. 表格加滚动容器
7. 亚像素边框用 `transform: scale()`
8. 标题装饰用 prefix/content/suffix 三段 span
9. 数学公式转 SVG 或图片
10. 图片必须微信 CDN
11. 容器命名空间隔离（`#nice`）
12. 仅使用安全 HTML 标签
13. H1 从正文移除（微信用标题字段）
14. `<em>` 渲染为粗体（斜体中文渲染差）

### 安装

**前置条件：**
- 任何 AI Agent（Claude Code、Codex、Gemini CLI、OpenCode 等）
- Python 3.12+ 和 [MarkItDown](https://github.com/microsoft/markitdown)（可选，用于非 Markdown 输入）

```bash
# 克隆仓库
git clone https://github.com/d-wwei/excellent-typesetter.git

# 可选：安装 MarkItDown 支持 PDF/DOCX/PPTX
pipx install 'markitdown[all]' --python python3.12
```

**各 Agent 配置方式：**

| Agent | 配置 |
|-------|------|
| Claude Code | `ln -s /path/to/excellent-typesetter ~/.claude/skills/typeset` — 自动注册为 `/typeset` |
| Codex | 将 `SKILL.md` 复制到项目根目录，或在 `AGENTS.md` 中引用 |
| Gemini CLI | 在 `GEMINI.md` 中引用 `SKILL.md` |
| OpenCode | 添加到 skill 目录或配置文件 |
| 其他 Agent | 将 `SKILL.md` 作为 system prompt 或 context 文件加载 |

### 来源

通过 [Remix](https://github.com/d-wwei/remix) 方法论构建，融合 7 个开源项目的最佳实践：

| 来源 | 贡献 |
|------|------|
| mdnice/markdown-nice | 四层主题架构、标题装饰、WeChat CSS 技巧 |
| baoyu-markdown-to-html | 扩展架构（Callout/脚注/代码高亮/CJK） |
| lyricat/wechat-format | 渲染时直接生成内联样式、列表兼容方案 |
| geekjourneyx/md2wechat-skill | 人性化模块（24 模式/5 维评分） |
| iamzifei/wechat-article-formatter | 微信 API 发布参考 |
| kepano/obsidian-skills | Callout 分类法（13 类型） |
| microsoft/markitdown | 输入格式预处理（16+ 格式） |

### 许可

MIT
