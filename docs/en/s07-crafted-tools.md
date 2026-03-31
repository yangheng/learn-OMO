# S07: Crafted Tools

> **Motto**: "AST search is grep's final form"

[← Previous](s06-background-agents.md) | [Next →](s08-prompt-engineering.md)

---

## Three Levels of Code Understanding

| Level | Tool | Understands |
|-------|------|-------------|
| Text | grep/glob | Strings, patterns |
| Structure | ast-grep | AST nodes, 25 languages |
| Semantics | LSP | Definitions, references, symbols, diagnostics |

## AST-Grep

```typescript
// src/tools/ast-grep/tools.ts
ast_grep_search: pattern="console.log($MSG)", lang="typescript"
ast_grep_replace: pattern→rewrite with meta-variables, dryRun=true by default
// Auto-hints on empty results (e.g., "Remove trailing colon for Python")
```

## LSP Tools

```typescript
// src/tools/lsp/tools.ts
lsp_goto_definition    // Jump to where something is defined
lsp_find_references    // Find ALL usages across workspace
lsp_diagnostics        // Errors/warnings — Sisyphus's verification tool
lsp_rename             // Safe cross-file rename
lsp_document_symbols   // File symbol list
lsp_workspace_symbols  // Workspace-wide symbol search
```

`lsp_diagnostics` is the enforcement mechanism for "NO EVIDENCE = NOT COMPLETE."

## Other Tools

- **interactive_bash**: tmux-based interactive shell sessions
- **session_manager**: List/read sessions
- **slashcommand**: Execute `.opencode/command/*.md` commands
- **skill/skill_mcp**: Load skills and manage MCP connections

---

[← Previous: Background Agents](s06-background-agents.md) | [Next: Prompt Engineering →](s08-prompt-engineering.md)
