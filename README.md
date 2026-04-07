<!--
  🌐 SEO & Keywords
  Oh My OpenCode (OMO) 源码解读 | OpenCode 插件系统 | Sisyphus Agent 编排器
  多Agent协作框架 | Claude Code 兼容层 | AI代码重构教程 | OpenCLAW 生态
-->
[English](./README.md) | [中文](./README-zh.md)

# learn-OMO: Deep Dive into Oh My OpenCode Architecture

> Follow one real task from start to finish — understand how OMO (Oh My OpenCode) uses Sisyphus orchestrator + multi-specialist Agents + automatic recovery to complete complex code refactoring.
>
> **Keywords**: Oh My OpenCode, OMO, OpenCode, Sisyphus, Claude Code, AI Agent, Multi-Agent Collaboration, Ralph Loop, Auto-retry, AST-grep, LSP Tools, Plugin System, Code Refactoring, OpenCLAW

## What This Is

**This is not API docs or a feature list. It's a story:**

You tell OMO "refactor this module for me", then follow that task through the entire system — from Plugin Bootstrap all the way to Ralph Loop automatic retry until the task completes.

Each chapter picks up where the previous one ended. Each chapter follows the code. Each chapter covers one thing.

## Capability Comparison

| Capability | Bare OpenCode | OpenCode + OMO | Claude Code |
|-----------|---------------|----------------|-------------|
| Multi-specialist Agents | ❌ Not supported | ✅ 8+ specialist agents | ✅ Built-in |
| Auto-retry loop ( Ralph Loop) | ❌ Not supported | ✅ Ralph Loop | ❌ Not supported |
| AST-aware search | ❌ Not supported | ✅ ast-grep (25+ languages) | ❌ Not supported |
| Full LSP toolchain | ❌ Not supported | ✅ definition/refs/rename | ⚠️ Partial |
| Plan→Review pipeline | ❌ Not supported | ✅ Prometheus→Metis→Momus | ❌ Not supported |
| Error auto-recovery | ❌ Not supported | ✅ 3 recovery hooks | ❌ Not supported |
| Customizability | Medium | Very High (30+ hook points) | Low |

## Architecture Overview

![Architecture](./docs/images/architecture.webp)

## Two Reading Paths

**🏗️ Senior developer? Quick architecture overview** → [Architecture Design](./docs/en/architecture.md)

**📖 New to OMO? Start from scratch** → [Chapter 1: Plugin Bootstrap](./docs/en/ch01-plugin-bootstrap.md)

## All Chapters

| # | Chapter | Core Content |
|---|---------|--------------|
| 01 | [Plugin Bootstrap](./docs/en/ch01-plugin-bootstrap.md) | How OMO plugin loads into OpenCode, registers hooks/tools/agents |
| 02 | [Sisyphus Receives Task](./docs/en/ch02-sisyphus-planning.md) | Sisyphus analyzes intent, assesses codebase, makes an execution plan |
| 03 | [Delegation System](./docs/en/ch03-delegation.md) | How delegate_task dispatches work to Sisyphus-Junior + skills |
| 04 | [Specialist Agents](./docs/en/ch04-specialist-agents.md) | How Oracle/Explore/Librarian three specialist agents collaborate |
| 05 | [Hook Pipeline](./docs/en/ch05-hook-pipeline.md) | How think-mode, rules injection, context management transparently enhance the system |
| 06 | [Error Recovery](./docs/en/ch06-error-recovery.md) | How edit failures, session crashes, context overflow auto-recover |
| 07 | [Background Execution](./docs/en/ch07-background-agents.md) | How to run multiple agents in parallel with concurrent search & execution |
| 08 | [Dynamic Prompts](./docs/en/ch08-dynamic-prompts.md) | How each agent's prompt is assembled on-the-fly from available resources |
| 09 | [Crafted Tools](./docs/en/ch09-crafted-tools.md) | How AST-grep, LSP, interactive bash go beyond native capabilities |
| 10 | [CC Compatibility](./docs/en/ch10-cc-compatibility.md) | How Claude Code users migrate to OMO with zero config changes |
| 11 | [Ralph Loop](./docs/en/ch11-ralph-loop.md) | Auto-loop until task complete — OMO's killer feature |

## How to Read

**Read front to back** — chapters form a continuous story, each building on the last.

**Read alongside the code** — every snippet includes file paths and line numbers. Open the source code and follow along.

## Related Projects

- **Oh My OpenCode**: [https://github.com/opensoft/oh-my-opencode](https://github.com/opensoft/oh-my-opencode)
- **OpenCode**: [https://github.com/opencode-ai/opencode](https://github.com/opencode-ai/opencode)
- **OpenCLAW**: [https://openclaw.ai](https://openclaw.ai)

## License

MIT
