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

将 Markdown 转换为多平台兼容的精美排版 HTML。适用于任何 AI Agent。融合 mdnice、baoyu、wechat-format、md2wechat 等最佳实践。

本 Skill 是纯 Markdown 指令集，不依赖任何特定 Agent 的专有 API。

| Agent | 加载方式 |
|-------|---------|
| **Claude Code** | `~/.claude/skills/` 目录，自动注册为 `/typeset` |
| **Codex** | 项目根目录或通过 `AGENTS.md` 引用 |
| **Gemini CLI** | 通过 `GEMINI.md` 引用或作为 context 加载 |
| **其他** | 将 SKILL.md 作为 system prompt 或 context 加载 |

**Reference files** (read on demand, not preloaded):
- `references/platform-rules.md` — 各平台详细约束规则
- `references/element-specs.md` — HTML 元素渲染模板
- `references/code-themes.md` — 代码高亮色表 + token 规则
- `references/humanizer.md` — 去 AI 味处理模块

## 调用格式

```
/typeset <文件路径或内容> [--platform wechat|zhihu|juejin|medium|linkedin|x|blog] [--theme default|elegant|tech|minimal|vibrant] [--humanize] [--code-theme github|one-dark|kimbie]
```

**默认值**: platform=wechat, theme=minimal, code-theme=github

---

## 工作流

### Step 1: 确认输入

1. 文件格式检测：
   - **Markdown (.md)** → 直接使用
   - **其他格式** → 通过 MarkItDown 转换（`markitdown <file> -o <output.md>`）
   - **内联文本** → 直接使用
2. 确认目标平台（默认微信）和主题（默认 minimal）

**MarkItDown 支持格式**: PDF, DOCX, PPTX, XLSX, HTML, CSV, JSON, XML, 图片(OCR), 音频(转录), Jupyter Notebook, ZIP
安装: `pipx install 'markitdown[all]' --python python3.12`

### Step 2: 预处理

1. **Frontmatter**: 解析 YAML，提取 title/author/description/tags
2. **CJK 空格**: 中英文之间自动加空格（盘古之白）
3. **标题清理**: 微信平台移除 H1（微信用独立标题字段）
4. **链接预处理**: 微信平台将外链转为纯文本脚注；其他平台保留

### Step 3: 可选 — 人性化处理

如果 `--humanize`，读取 `references/humanizer.md` 执行去 AI 味处理。

### Step 4: 渲染 HTML

核心步骤。按以下顺序：

1. **读主题**: `{{SKILL_DIR}}/themes/<theme>.md`
2. **渲染元素**: 读取 `references/element-specs.md`，按规范将每个 Markdown 元素转为内联样式 HTML
3. **应用平台规则**: 读取 `references/platform-rules.md` 中目标平台的约束
4. **代码高亮**: 读取 `references/code-themes.md` 中对应 code-theme 的色表
5. **组装文档**: 生成完整 HTML

### Step 5: 输出

1. 写 HTML 文件（与输入同目录，`.html` 扩展名）
2. 报告路径
3. 提示粘贴方式：浏览器打开 → 全选 → 复制 → 粘贴到编辑器
4. 微信平台额外提示图片需单独上传

---

## 主题系统

### 四层架构

| 层 | 作用 |
|---|---|
| **基础层** | 容器、排版基础（所有主题共享） |
| **内容层** | 标题、段落、列表等视觉风格 |
| **代码层** | 代码高亮（可独立切换） |
| **字体层** | 字体族覆盖（可选） |

### 内置主题

| 主题 | 风格 | 适用场景 |
|---|---|---|
| `minimal` (**默认**) | 极简、大留白 | 通用文章、短文 |
| `default` | 专业、蓝色调 | 正式报告、技术文章 |
| `elegant` | 文艺、暖色调 | 人文/商业/叙事 |
| `tech` | 现代、冷色调 | 技术教程、开发文档 |
| `vibrant` | 活力、绿色调 | 品牌文/营销 |
| `dark` | 深色、护眼 | 技术社区、夜间阅读 |
| `notion` | Notion 风格 | 知识库、内部文档 |
| `newspaper` | 报纸印刷风 | 深度报道、长文 |
| `gradient` | 现代渐变 | 产品发布 |

主题文件: `{{SKILL_DIR}}/themes/<name>.md`
主题预览: `{{SKILL_DIR}}/themes/preview.html`
自定义主题: 在 themes/ 下创建同格式 .md 文件

---

## 平台路由

渲染前读取 `references/platform-rules.md` 中对应平台的规则。

| 平台 | CSS 方式 | 链接 | 代码块 | 表格 | 关键约束 |
|------|---------|------|--------|------|---------|
| wechat | 全内联 | 纯文本脚注 | section+span | 滚动容器 | 禁 #000, 禁 pre, 禁外链 a 标签 |
| zhihu | 内联 | 保留 | pre/code | 正常 | 公式用 data-eeimg |
| juejin | 内联 | 保留 | pre/code | 正常 | — |
| medium | 内联 | 保留 | pre (无高亮) | 不支持 | 只 H1+H2, Medium 覆盖样式 |
| linkedin | 内联 | 保留 | 不支持 | 不支持 | 图片需手动上传 |
| x | 内联 | 保留 | pre (无高亮) | 有限 | Premium 功能 |
| blog | style 标签 | 保留 | pre/code+hljs | 正常 | 完整 HTML 文档 |

---

## 输出格式

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>{标题}</title>
</head>
<body>
  <section id="nice" style="...">
    <!-- 渲染后的内联样式 HTML -->
  </section>
</body>
</html>
```

文件命名: `article.md` → `article.html`

---

## 最佳实践

1. 先确认平台再选主题
2. 技术文章用 tech 主题
3. 代码块建议 80 字符换行
4. 人性化处理建议在排版前执行
5. 微信链接一律纯文本脚注，不使用 `<a>` 标签

---

## 来源声明

| 来源 | 贡献 |
|---|---|
| mdnice/markdown-nice | 四层主题、标题三段式、WeChat CSS |
| baoyu-markdown-to-html | Callout/脚注/代码高亮、CJK 预处理 |
| wechat-format | 内联样式、列表兼容、亚像素边框 |
| md2wechat-skill | 人性化模块、图片处理 |
| obsidian-markdown | Callout 分类法、Frontmatter 规范 |
