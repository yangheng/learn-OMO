# learn-OMO: Deep Dive into Oh My OpenCode

> A 12-session source code analysis of **Oh My OpenCode** — the #1 OpenCode plugin that transforms it into a multi-agent coding platform rivaling Claude Code.

Modeled after [learn-claude-code](https://github.com/shareAI-lab/learn-claude-code).

## What is Oh My OpenCode?

OMO is a **plugin** for [OpenCode](https://github.com/sst/opencode) that adds:
- 🤖 **8+ specialist agents** (Sisyphus, Oracle, Librarian, Atlas, Momus, Metis, Explore, Prometheus)
- 🔄 **Ralph Loop** — auto-retry until task completion
- 🛠️ **Crafted tools** — AST-grep, LSP, interactive bash, delegate-task
- 🪝 **30+ hooks** — event-driven behavior modification
- 🔌 **Claude Code compatibility** — agent/MCP/plugin/skill loaders

## Sessions

| # | Topic | Motto |
|---|-------|-------|
| [01](docs/en/s01-plugin-architecture.md) | Plugin Architecture | "It all starts with a Plugin interface" |
| [02](docs/en/s02-multi-agent-system.md) | Multi-Agent System | "Each agent is named after a god for a reason" |
| [03](docs/en/s03-sisyphus-discipline-agent.md) | Sisyphus: The Discipline Agent | "Humans roll their boulder every day. So do you." |
| [04](docs/en/s04-agent-delegation.md) | Agent Delegation | "A Sisyphus that doesn't delegate is just an agent" |
| [05](docs/en/s05-hook-system.md) | Hook System | "Hooks are OMO's nervous system" |
| [06](docs/en/s06-background-agents.md) | Background Agents | "tmux is the poor man's container orchestration" |
| [07](docs/en/s07-crafted-tools.md) | Crafted Tools | "AST search is grep's final form" |
| [08](docs/en/s08-prompt-engineering.md) | Prompt Engineering | "Prompts aren't written, they're built" |
| [09](docs/en/s09-claude-code-compat.md) | Claude Code Compatibility | "The best migration is invisible" |
| [10](docs/en/s10-skill-mcp-management.md) | Skill & MCP Management | "Plugins for plugins, turtles all the way down" |
| [11](docs/en/s11-error-recovery.md) | Error Recovery & Resilience | "A crash isn't the end, it's a restart" |
| [12](docs/en/s12-ralph-loop.md) | The Ralph Loop | "Done yet? No? Keep going." |

## Quick Comparison

| Capability | Bare OpenCode | OpenCode + OMO | Claude Code |
|-----------|---------------|----------------|-------------|
| Multi-agent | ❌ | ✅ 8+ agents | ✅ Built-in |
| Auto-retry loop | ❌ | ✅ Ralph Loop | ❌ |
| AST-aware search | ❌ | ✅ ast-grep | ❌ |
| LSP integration | ❌ | ✅ Full LSP | Partial |
| Plan→Execute pipeline | ❌ | ✅ Prometheus→Atlas | ❌ |
| Customizability | Medium | Very High | Low |

---

Start reading: [Session 01 — Plugin Architecture](docs/en/s01-plugin-architecture.md)

*For educational purposes only. All code references from the Oh My OpenCode open-source repository.*
