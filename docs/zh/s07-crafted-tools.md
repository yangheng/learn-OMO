# S07：精心打造的工具集 — AST 搜索是 grep 的进化形态

> **格言**："AST 搜索是 grep 的进化形态"

[← 上一章](s06-background-agents.md) | [下一章 →](s08-prompt-engineering.md)

---

## 问题：Agent 需要什么级别的代码理解能力？

文本搜索 (grep) 只能找字符串。**AST 搜索** 理解代码结构。**LSP** 理解语义。OMO 三者兼备。

## 工具全景

```
┌──────────────────────────────────────────────────┐
│                 OMO 工具层                         │
├──────────────┬──────────────┬────────────────────┤
│  文本搜索     │  结构搜索     │  语义理解          │
│  grep/glob   │  ast-grep    │  LSP               │
├──────────────┼──────────────┼────────────────────┤
│ 关键词匹配   │ AST 模式匹配  │ 定义跳转           │
│ 文件查找     │ 元变量 $VAR   │ 引用查找           │
│ 正则表达式   │ 结构替换      │ 符号列表           │
│              │ 25 种语言     │ 诊断信息           │
│              │              │ 安全重命名          │
└──────────────┴──────────────┴────────────────────┘
```

## AST-Grep — 结构感知搜索

```typescript
// src/tools/ast-grep/tools.ts:L30-L45
export const ast_grep_search: ToolDefinition = tool({
  description:
    "Search code patterns across filesystem using AST-aware matching. " +
    "Use meta-variables: $VAR (single node), $$$ (multiple nodes). " +
    "Patterns must be complete AST nodes (valid code).",
  args: {
    pattern: tool.schema.string(),  // 例如 "console.log($MSG)"
    lang: tool.schema.enum(CLI_LANGUAGES),
    paths: tool.schema.array(tool.schema.string()).optional(),
    globs: tool.schema.array(tool.schema.string()).optional(),
  },
})
```

AST-grep 的**空结果提示**：

```typescript
// src/tools/ast-grep/tools.ts:L18-L28
function getEmptyResultHint(pattern: string, lang: CliLanguage): string | null {
  if (lang === "python") {
    if (src.startsWith("class ") && src.endsWith(":")) {
      return `Hint: Remove trailing colon. Try: "${withoutColon}"`
    }
  }
  if (["javascript", "typescript", "tsx"].includes(lang)) {
    if (/^(export\s+)?(async\s+)?function\s+\$[A-Z_]+\s*$/i.test(src)) {
      return `Hint: Function patterns need params and body.`
    }
  }
}
```

还有 `ast_grep_replace` 支持 dry-run 预览。

## LSP 工具集

```typescript
// src/tools/lsp/tools.ts
export const lsp_goto_definition      // 跳转到定义
export const lsp_find_references      // 查找所有引用
export const lsp_document_symbols     // 文件符号列表
export const lsp_workspace_symbols    // 工作区符号搜索
export const lsp_diagnostics          // 诊断信息 (错误/警告)
export const lsp_rename               // 安全重命名
export const lsp_prepare_rename       // 预检查重命名
```

LSP 工具使用 `withLspClient` 封装来管理 LSP 连接：

```typescript
// src/tools/lsp/utils.ts
export async function withLspClient(filePath, callback) {
  const client = await lspManager.getOrCreateClient(filePath)
  return await callback(client)
}
```

`lsp_diagnostics` 是 Sisyphus 验证代码质量的核心工具：每次编辑后都必须调用。

## Interactive Bash — tmux 交互式终端

```typescript
// src/tools/interactive-bash/tools.ts
// 在 tmux session 中运行交互式命令
// 支持长运行进程、交互式输入
```

## Session Manager — 会话管理

```typescript
// src/tools/session-manager/tools.ts
// 列出 sessions、读取消息、管理状态
```

## Slashcommand — 斜杠命令

```typescript
// src/tools/slashcommand/tools.ts
// 发现并执行 .opencode/command/*.md 中定义的命令
```

## 工具选择矩阵

Sisyphus 的 prompt 中包含工具选择指导：

```
| Resource    | Cost | When to Use                                    |
|-------------|------|------------------------------------------------|
| grep, glob  | FREE | Not Complex, Scope Clear                       |
| lsp_*       | FREE | Semantic understanding needed                  |
| ast_grep    | FREE | Structural pattern matching                    |
| explore     | FREE | 2+ modules, parallel background                |
| librarian   | CHEAP| External libraries, docs                       |
| oracle      | EXPENSIVE | Architecture, hard debugging               |
```

## 关键洞察

1. **三层搜索**：文本 (grep) → 结构 (AST) → 语义 (LSP)，按需升级。
2. **AST-grep 自动提示**：当搜索无结果时，根据语言给出模式修正建议。
3. **LSP 是验证核心**：`lsp_diagnostics` 是"NO EVIDENCE = NOT COMPLETE"的执行者。
4. **工具是分级的**：FREE → CHEAP → EXPENSIVE，Agent 知道何时用哪个。

---

[← 上一章：后台 Agent](s06-background-agents.md) | [下一章：提示工程 →](s08-prompt-engineering.md)
