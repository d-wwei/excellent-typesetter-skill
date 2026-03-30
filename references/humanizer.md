# Humanizer Module

Read this file when `--humanize` is specified. Activates de-AI processing on the text.

Source: md2wechat humanizer.

---

## 24 AI Pattern Detection (5 Categories)

### 内容类
1. **过度强调** — 不必要的「非常」「极其」「令人惊叹」
2. **夸大其词** — 将普通事物描述得过于重大
3. **推销语气** — 像广告文案而非正常表达
4. **模糊归因** — 「研究表明」「专家认为」无具体来源

### 语言类
5. **AI 词汇** — 「赋能」「助力」「深入探讨」「至关重要」「不可或缺」
6. **否定并行** — 「不是 X，而是 Y」反复使用
7. **三段式结构** — 每次都是三个并列（三个好处、三个方面、三个建议）
8. **同义词轮换** — 不自然地交替使用近义词避免重复

### 风格类
9. **破折号滥用** — 过多使用「——」作为插入语
10. **加粗滥用** — 大量 **粗体** 标记，失去强调效果
11. **Emoji 装饰** — 不必要的表情符号点缀

### 填充类
12. **垫话短语** — 「值得注意的是」「需要指出的是」「毫无疑问」
13. **过度限定** — 每个观点都加「当然也要考虑」「但这并不意味着」
14. **万能结尾** — 「总之」「综上所述」+ 空洞的展望

### 协作类
15. **对话式填充** — 「这是个好问题」「让我来解释」
16. **知识截止声明** — 「截至我所知」「根据我的训练数据」

---

## Three Intensity Levels

| Level | Detection Threshold | Edit Force | Use Case |
|---|---|---|---|
| gentle | Only obvious traces | Minimal edits | Already fairly natural text |
| medium (default) | Medium traces | Balanced edits | General purpose |
| aggressive | Subtle traces | Heavy rewrite | Text with strong AI smell |

---

## Processing Flow

1. Scan text, detect each of the 24 patterns by category
2. Mark location and severity for each detection
3. Decide whether to edit based on current intensity level
4. Execute: replace AI vocabulary, simplify structure, remove filler
5. Output change report (optional `--show-changes`)

---

## Five-Dimension Quality Score (/10 each, total /50)

| Dimension | Description |
|---|---|
| 直接性 (Directness) | Gets to the point without detours |
| 节奏感 (Rhythm) | Sentence length varies, not monotone |
| 信任度 (Trustworthiness) | Reads like a human wrote it |
| 真实性 (Authenticity) | Has personal perspective and specific details |
| 精炼度 (Conciseness) | No filler, no padding |
