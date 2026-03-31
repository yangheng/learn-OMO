# 第九章：增强工具 — AST、LSP 与交互式 Bash

> For the comprehensive version with full code analysis, see the [Chinese edition](../zh/ch09-crafted-tools.md).

> **格言**：*"grep 找文本，AST 找结构，LSP 找语义。三者齐上，代码无处遁形。"*
## 上回
## 问题
## 代码路径
### ast_grep_search：AST 感知的代码搜索
**智能提示**：当搜索无结果时，会检查常见错误：
### LSP 工具集：IDE 级别的语义理解
### interactive_bash：持久化终端
### 工具分类
### 工具能力对比
## 架构图
## 关键洞察
**OMO 的工具不是"更好的 shell 命令"——是"IDE 能力的 API 化"。** LSP 工具给了 agent 等同于 VS Code 的语义理解能力：跳转到定义、查找引用、安全重命名、实时诊断。AST-grep 给了结构搜索能力。这些组合在一起，让 agent 做重构时能做到：先用 `lsp_find_references` 了解影响面，用 `ast_grep` 找到所有模式，修改后用 `lsp_diagnostics` 验证。
## 下一步

---

*Full source code analysis available in the [Chinese version](../zh/ch09-crafted-tools.md).*
