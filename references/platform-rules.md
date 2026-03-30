# Platform-Specific Rules

Read this file when you need the detailed constraints for a specific target platform.
Only read the section for the platform being typeset to.

---

## 微信公众号 (wechat)

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

---

## 知乎 (zhihu)

1. **CSS 内联** — 与微信类似，知乎也会剥离外部样式
2. **链接可保留** — 知乎支持可点击外部链接，不需要脚注化
3. **公式用 `<img data-eeimg>`** — 知乎有专用公式标签，LaTeX 作为 alt 文本
4. **代码块保留 `<pre><code>`** — 知乎正常支持 `<pre>` 标签
5. **图片可用外部链接** — 知乎会自动代理外部图片

---

## 掘金 (juejin)

1. **CSS 内联** — 掘金也剥离外部样式
2. **链接可保留** — 掘金支持外部链接
3. **公式用图片 URL** — 掘金不支持 MathJax，公式转为外部渲染服务图片
4. **代码块**: `<pre><code>` + `display: block` + 换行符规范化
5. **图片**: 掘金有自己的图片上传服务，外部图片可能被替换

---

## Medium (medium)

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

---

## LinkedIn (linkedin)

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

---

## X Articles (x)

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

---

## 通用博客 (blog)

1. **完整 HTML 文档** — 生成带 `<!DOCTYPE>`, `<head>`, `<body>` 的完整 HTML
2. **可用 `<style>` 标签** — 不需要全部内联，可保留嵌入式样式表
3. **链接正常** — 保留所有超链接
4. **公式用 KaTeX** — 引入 KaTeX CSS/JS，使用标准 LaTeX 语法
5. **Mermaid 支持** — 引入 Mermaid.js，代码块保留为 `<div class="mermaid">`
6. **语法高亮**: 可引入 highlight.js 或 Prism.js
