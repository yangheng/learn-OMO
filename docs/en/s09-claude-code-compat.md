# S09: Claude Code Compatibility

> **Motto**: "The best migration is invisible"

[← Previous](s08-prompt-engineering.md) | [Next →](s10-skill-mcp-management.md)

---

## Three Compatibility Loaders

```typescript
// src/features/claude-code-agent-loader/loader.ts
// Reads ~/.claude/agents/*.md → parses frontmatter → converts to AgentConfig
function loadAgentsFromDir(agentsDir, scope: "user" | "project"): LoadedAgent[] {
  const { data, body } = parseFrontmatter<AgentFrontmatter>(content)
  return { description: `(${scope}) ${data.description}`, mode: "subagent", prompt: body.trim() }
}

// Two levels: loadUserAgents() + loadProjectAgents()
```

**MCP Loader**: `getSystemMcpServerNames()` loads Claude Code MCP server configurations.

**Plugin Loader**: Loads Claude Code plugins for OpenCode compatibility.

## Skill Compatibility

```typescript
// src/index.ts
const includeClaudeSkills = pluginConfig.claude_code?.skills !== false
// Discovers skills from both Claude and OpenCode directories
```

## Session State Mapping

OMO maintains session→agent mapping via `setSessionAgent`/`getSessionAgent`/`clearSessionAgent`, tracking which agent is active in each session.

---

[← Previous: Prompt Engineering](s08-prompt-engineering.md) | [Next: Skill & MCP Management →](s10-skill-mcp-management.md)
