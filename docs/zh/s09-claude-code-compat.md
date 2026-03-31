# S09：Claude Code 兼容层 — 最好的迁移是无感迁移

> **格言**："最好的迁移是无感迁移"

[← 上一章](s08-prompt-engineering.md) | [下一章 →](s10-skill-mcp-management.md)

---

## 问题：Claude Code 用户如何无痛迁移到 OpenCode + OMO？

OMO 提供了三个兼容加载器，让 Claude Code 的 Agent、MCP 和 Plugin 直接在 OpenCode 中工作。

## 兼容层架构

```
Claude Code 配置
    │
    ├── ~/.claude/agents/*.md    ──→ claude-code-agent-loader
    │                                 ├── 解析 frontmatter
    │                                 └── 转换为 AgentConfig
    │
    ├── ~/.claude/mcp-servers    ──→ claude-code-mcp-loader
    │                                 └── 加载 MCP 服务配置
    │
    └── ~/.claude/plugins        ──→ claude-code-plugin-loader
                                      └── 加载 CC 插件
```

## Agent Loader — 从 Markdown 到 AgentConfig

```typescript
// src/features/claude-code-agent-loader/loader.ts:L29-L55
function loadAgentsFromDir(agentsDir: string, scope: AgentScope): LoadedAgent[] {
  for (const entry of entries) {
    if (!isMarkdownFile(entry)) continue
    const content = readFileSync(agentPath, "utf-8")
    const { data, body } = parseFrontmatter<AgentFrontmatter>(content)

    const config: AgentConfig = {
      description: `(${scope}) ${data.description || ""}`,
      mode: "subagent",
      prompt: body.trim(),
    }

    const toolsConfig = parseToolsConfig(data.tools)
    if (toolsConfig) config.tools = toolsConfig
  }
}
```

Claude Code 的 Agent 文件格式：

```markdown
---
name: my-agent
description: A custom agent
tools: read,write,bash
---

You are a custom agent that...
```

OMO 自动将其转换为 OpenCode 的 `AgentConfig`，`scope` 标记来源（`user` 或 `project`）。

## 两级加载

```typescript
// src/features/claude-code-agent-loader/loader.ts:L62-L76
export function loadUserAgents(): Record<string, AgentConfig> {
  const userAgentsDir = join(getClaudeConfigDir(), "agents")
  return loadAgentsFromDir(userAgentsDir, "user")
}

export function loadProjectAgents(): Record<string, AgentConfig> {
  const projectAgentsDir = join(process.cwd(), ".claude", "agents")
  return loadAgentsFromDir(projectAgentsDir, "project")
}
```

- **用户级**：`~/.claude/agents/` — 跨项目可用
- **项目级**：`.claude/agents/` — 仅在当前项目可用

## Claude Code Hooks 兼容

```typescript
// src/index.ts
const claudeCodeHooks = createClaudeCodeHooksHook(ctx, {
  disabledHooks: (pluginConfig.claude_code?.hooks ?? true) ? undefined : true,
  keywordDetectorDisabled: !isHookEnabled("keyword-detector"),
}, contextCollector)
```

这个 Hook 在所有生命周期阶段都被调用，确保 Claude Code 的 Hook 行为被正确模拟。

## Session State 映射

```typescript
// src/features/claude-code-session-state/index.ts
export {
  setMainSession,
  getMainSessionID,
  setSessionAgent,
  updateSessionAgent,
  clearSessionAgent,
}
```

OMO 维护了一个 session-agent 映射，跟踪每个 session 当前使用的 Agent。

## Skill 兼容

```typescript
// src/index.ts:L105-L110
const includeClaudeSkills = pluginConfig.claude_code?.skills !== false
const [userSkills, ...] = await Promise.all([
  includeClaudeSkills ? discoverUserClaudeSkills() : Promise.resolve([]),
  // ...
])
```

Claude Code 的 skill 通过配置 `claude_code.skills` 控制是否加载。

## 关键洞察

1. **Markdown 是通用接口**：Claude Code 的 Agent 定义是 Markdown + frontmatter，OMO 直接解析它。
2. **Scope 标记**：通过 `(user)` / `(project)` 前缀标记 Agent 来源，避免命名冲突。
3. **渐进式兼容**：用户可以选择性关闭 CC 兼容 (`claude_code.skills: false`)。
4. **不是模拟，是翻译**：OMO 不是在"模拟" Claude Code，而是将 CC 的概念翻译为 OpenCode 的等价物。

---

[← 上一章：提示工程](s08-prompt-engineering.md) | [下一章：Skill 与 MCP 管理 →](s10-skill-mcp-management.md)
