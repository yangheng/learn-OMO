[English](./README.md) | [中文](./README-zh.md)

# learn-OMO: Deep Dive into Oh My OpenCode

> Follow one task from start to finish — understand every gear in OMO's machinery.

## What This Is

This is not API docs or a feature list. It's a **story**: you tell OMO "refactor this module", then follow that task through the entire system — from plugin bootstrap to Ralph Loop completion.

Each chapter picks up where the previous one ends. Each chapter follows the code. Each chapter covers one thing.

## Architecture

![Architecture](./docs/images/architecture.webp)

## Chapters

| # | Chapter | Story Arc |
|---|---------|-----------|
| 01 | [Plugin Bootstrap](./docs/en/ch01-plugin-bootstrap.md) | OMO plugin loads into OpenCode, registers hooks/tools/agents |
| 02 | [Sisyphus Receives Task](./docs/en/ch02-sisyphus-planning.md) | Sisyphus analyzes intent, assesses codebase, makes a plan |
| 03 | [Delegation System](./docs/en/ch03-delegation.md) | delegate_task dispatches work to Sisyphus-Junior + skills |
| 04 | [Specialist Agents](./docs/en/ch04-specialist-agents.md) | Oracle thinks, Explore searches, Librarian reads docs |
| 05 | [Hook Pipeline](./docs/en/ch05-hook-pipeline.md) | think-mode, rules injection, context management — transparent enhancement |
| 06 | [Error Recovery](./docs/en/ch06-error-recovery.md) | Edit failures, session crashes, context overflow — auto-fix |
| 07 | [Background Execution](./docs/en/ch07-background-agents.md) | Run multiple agents in parallel, concurrent search & execution |
| 08 | [Dynamic Prompts](./docs/en/ch08-dynamic-prompts.md) | Each agent's prompt assembled on-the-fly from available resources |
| 09 | [Crafted Tools](./docs/en/ch09-crafted-tools.md) | AST-grep, LSP, interactive bash — beyond native capabilities |
| 10 | [CC Compatibility](./docs/en/ch10-cc-compatibility.md) | Claude Code users migrate without changing any config |
| 11 | [Ralph Loop](./docs/en/ch11-ralph-loop.md) | Auto-retry until task complete — OMO's killer feature |

## Quick Comparison

| Capability | Bare OpenCode | OpenCode + OMO | Claude Code |
|-----------|---------------|----------------|-------------|
| Multi-agent | ❌ | ✅ 8+ specialists | ✅ Built-in |
| Auto-retry loop | ❌ | ✅ Ralph Loop | ❌ |
| AST-aware search | ❌ | ✅ ast-grep (25 langs) | ❌ |
| Full LSP tools | ❌ | ✅ definition/refs/rename | Partial |
| Plan→Review pipeline | ❌ | ✅ Prometheus→Metis→Momus | ❌ |
| Error auto-recovery | ❌ | ✅ 3 recovery hooks | ❌ |
| Customizability | Medium | Very High (30+ hooks) | Low |

## How to Read

**Read front to back** — chapters form a continuous story, each building on the last.

**Read alongside the code** — every snippet includes file paths and line numbers.

## Source

Oh My OpenCode: [GitHub](https://github.com/opensoft/oh-my-opencode)

## License

MIT
