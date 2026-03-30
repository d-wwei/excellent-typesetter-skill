# Code Syntax Highlighting Themes

Read this file when rendering code blocks. Choose the active code theme based on `--code-theme` flag.

Since WeChat doesn't support external JS, syntax highlighting is done via inline `color` styles per token.

---

## GitHub Light (default)

| Token | Color |
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
| background | #f6f8fa |
| text | #24292e |

## One Dark

| Token | Color |
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
| background | #282c34 |
| text | #abb2bf |

## Kimbie Light

| Token | Color |
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
| background | #fbebd4 |
| text | #84613d |

---

## Token Recognition Rules

Identify token types when generating code blocks:

- **keyword**: Language reserved words (`const`, `let`, `var`, `function`, `class`, `if`, `else`, `for`, `while`, `return`, `import`, `export`, `from`, `def`, `async`, `await`, etc.)
- **string**: Quote-wrapped text (single, double, backtick, triple quotes)
- **comment**: `//`, `/* */`, `#`, `<!-- -->`, `"""`, etc.
- **function**: Function names (definition and invocation)
- **number**: Numeric literals
- **operator**: `=`, `+`, `-`, `*`, `/`, `===`, `!==`, `=>`, `&&`, `||`, etc.
- **variable**: Variable names (not keywords, not functions)
- **type**: Type names (capitalized, type annotations)
- **property**: Object properties (identifier after `.`)
- **tag**: HTML/JSX tag names
- **attribute**: HTML/JSX attribute names
