# Element Rendering Specs

Read this file when rendering Markdown elements to HTML. Each section defines the exact HTML structure and inline styles. **Specific color and font values come from the active theme** — use `{theme.xxx}` placeholders.

---

## 容器

```html
<section id="nice" style="max-width: {theme.container.max-width}; margin: 0 auto; padding: {theme.container.padding}; background: {theme.container.background}; color: {theme.text.color}; font-family: {theme.font.family}; font-size: {theme.font.size}; line-height: {theme.font.line-height}; word-break: break-word; overflow-wrap: break-word;">
  <!-- 内容 -->
</section>
```

## 段落

```html
<p style="margin: {theme.paragraph.margin}; padding: 0; color: {theme.text.color}; font-size: {theme.font.size}; line-height: {theme.font.line-height}; letter-spacing: {theme.text.letter-spacing};">
  段落文本
</p>
```

## 标题 (H1-H6)

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

## 强调

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

## 列表

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
```

**微信额外规则**: 如果原生列表样式被重置，改用自定义字符：
- 无序列表用 `•` 文本字符 + `padding-left`
- 有序列表用数字文本前缀 `1.` + `padding-left`

## 引用块

```html
<blockquote style="margin: {theme.blockquote.margin}; padding: {theme.blockquote.padding}; border-left: {theme.blockquote.border-left}; background: {theme.blockquote.background}; color: {theme.blockquote.color};">
  <p style="margin: 0; font-size: {theme.blockquote.font-size}; line-height: {theme.font.line-height};">
    引用内容
  </p>
</blockquote>
```

支持多层嵌套引用，通过递增缩进和递减透明度区分层级。

## 代码块

**微信平台**（禁止 `<pre>`）：

```html
<section style="margin: {theme.code-block.margin}; border-radius: {theme.code-block.border-radius}; overflow: hidden; background: {theme.code-block.background};">
  <!-- macOS 窗口装饰 -->
  <section style="padding: 8px 16px; display: flex; align-items: center; gap: 6px;">
    <span style="width: 10px; height: 10px; border-radius: 50%; background: #ff5f57; display: inline-block;"></span>
    <span style="width: 10px; height: 10px; border-radius: 50%; background: #febc2e; display: inline-block;"></span>
    <span style="width: 10px; height: 10px; border-radius: 50%; background: #28c840; display: inline-block;"></span>
    <span style="flex: 1;"></span>
    <span style="font-size: 12px; color: #999; font-family: monospace;">language</span>
  </section>
  <!-- 代码内容: 逐行逐 token 着色 -->
  <section style="padding: 12px 16px; overflow-x: auto; font-family: 'Menlo', 'Monaco', 'Consolas', 'Courier New', monospace; font-size: {theme.code-block.font-size}; line-height: 1.6;">
    <span style="color: {syntax.keyword};">const</span> <span style="color: {syntax.variable};">name</span> = <span style="color: {syntax.string};">"hello"</span><br>
  </section>
</section>
```

**博客平台**: 可使用标准 `<pre><code>` + highlight.js。

## 行内代码

```html
<code style="background: {theme.inline-code.background}; color: {theme.inline-code.color}; padding: 2px 6px; border-radius: {theme.inline-code.border-radius}; font-family: 'Menlo', 'Monaco', 'Consolas', monospace; font-size: 0.9em;">
  code
</code>
```

## 表格

```html
<!-- 微信: 滚动容器 -->
<section style="overflow-x: auto; -webkit-overflow-scrolling: touch; margin: 16px 0;">
  <table style="border-collapse: collapse; width: 100%; min-width: 400px; font-size: {theme.table.font-size};">
    <thead>
      <tr style="background: {theme.table.header-background};">
        <th style="border: 1px solid {theme.table.border-color}; padding: {theme.table.cell-padding}; text-align: left; font-weight: bold; color: {theme.table.header-color};">表头</th>
      </tr>
    </thead>
    <tbody>
      <tr style="background: {theme.table.row-even-background};">
        <td style="border: 1px solid {theme.table.border-color}; padding: {theme.table.cell-padding}; color: {theme.text.color};">内容</td>
      </tr>
      <!-- 交替行背景色: odd=白, even=浅灰 -->
    </tbody>
  </table>
</section>
```

## 图片

```html
<figure style="margin: 20px 0; text-align: center;">
  <img src="图片URL" alt="描述" style="max-width: 100%; border-radius: {theme.image.border-radius}; display: block; margin: 0 auto;">
  <figcaption style="margin-top: 8px; font-size: 13px; color: {theme.text.secondary-color}; text-align: center;">图注文字</figcaption>
</figure>
```

## 水平线

```html
<hr style="border: none; height: 1px; background: {theme.hr.color}; margin: {theme.hr.margin}; transform: scale(1, 0.5);">
```

微信下用 `transform: scale(1, 0.5)` 实现 0.5px 视觉效果。

## 链接

### 微信公众号 — 纯文本脚注（无超链接）

```html
<!-- 正文中：纯文本 + 上标编号（无 <a> 标签） -->
<span style="color: {theme.text.color};">链接文字</span><sup style="font-size: 0.7em; color: {theme.footnote.color}; vertical-align: super;">[1]</sup>

<!-- 文末参考文献区 -->
<section style="margin-top: 40px; padding-top: 16px; border-top: 1px solid {theme.hr.color};">
  <p style="font-size: 14px; font-weight: bold; color: {theme.text.secondary-color}; margin-bottom: 8px;">参考文献</p>
  <p style="font-size: 13px; color: {theme.text.secondary-color}; margin: 4px 0; word-break: break-all;">
    [1] 链接文字: https://example.com/url
  </p>
</section>
```

微信域内链接（`mp.weixin.qq.com`）是唯一例外，可保留 `<a>` 标签。

### 其他平台

链接保留为正常 `<a>` 标签。

## Callout / 提示框

支持 GFM (`> [!NOTE]`) 和 Obsidian (`:::note`) 语法。13 种类型。

```html
<section style="margin: 16px 0; padding: 12px 16px; border-left: 4px solid {callout.border-color}; background: {callout.background}; border-radius: 0 4px 4px 0;">
  <p style="margin: 0 0 8px 0; font-weight: bold; color: {callout.title-color}; font-size: 15px;">
    {callout.icon} {callout.title}
  </p>
  <section style="color: {theme.text.color}; font-size: {theme.font.size};">内容</section>
</section>
```

**Callout 颜色映射**:

| 类型 | 边框/标题色 | 背景色 |
|---|---|---|
| note | #448aff | #e8f0fe |
| tip / important | #00bfa5 | #e6f9f0 |
| warning / caution | #ff9100 | #fff8e1 |
| danger / error | #ff5252 | #fde8e8 |
| info | #2196f3 | #e3f2fd |
| success | #4caf50 | #e8f5e9 |
| question | #9c27b0 | #f3e5f5 |
| bug | #f44336 | #fde8e8 |
| example | #7c4dff | #ede7f6 |
| quote | #9e9e9e | #f5f5f5 |

微信平台下 Callout 图标（emoji）可能被过滤，使用文字标签替代。

## 数学公式

**微信**: 公式必须转为 SVG 或图片。行内用 `<span>` 包裹 SVG，块级居中展示。

**实际策略**: 简单公式用 HTML 上标/下标近似；复杂公式保留 LaTeX 源码标注手动渲染；博客平台引入 KaTeX。

## Mermaid / PlantUML

**微信**: 不支持 JS。标注需手动截图替换，或用 PlantUML 服务器生成图片 URL。

**博客**: 保留 Mermaid 代码块，引入 Mermaid.js。

## 脚注

**微信**（纯文本，无 `<a>`）：
```html
<sup style="font-size: 0.7em; color: {theme.footnote.color};">[1]</sup>
<!-- 文末 -->
<p style="font-size: 13px; color: {theme.text.secondary-color};">[1] 脚注内容</p>
```

**博客/其他**（双向链接）：
```html
<sup id="fnRef-1"><a href="#fnDef-1" style="color: {theme.footnote.color};">[1]</a></sup>
<!-- 文末 -->
<p id="fnDef-1"><a href="#fnRef-1">[1]</a> 脚注内容</p>
```
