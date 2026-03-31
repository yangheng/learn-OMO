# S10: Skill & MCP Management

> **Motto**: "Plugins for plugins, turtles all the way down"

[← Previous](s09-claude-code-compat.md) | [Next →](s11-error-recovery.md)

---

## Five-Layer Skill Discovery

```typescript
// src/index.ts
const mergedSkills = mergeSkills(
  builtinSkills,           // 1. OMO built-in (e.g., playwright)
  pluginConfig.skills,     // 2. User config
  userSkills,              // 3. ~/.claude/ user-level
  globalSkills,            // 4. ~/.config/opencode/skills/
  opencodeProjectSkills    // 5. .opencode/skills/ project-level
)
```

## Builtin Skills — Conflict Detection

```typescript
// src/index.ts:L119-L126
const builtinSkills = createBuiltinSkills({ browserProvider }).filter((skill) => {
  if (disabledSkills.has(skill.name)) return false
  if (skill.mcpConfig) {
    for (const mcpName of Object.keys(skill.mcpConfig)) {
      if (systemMcpNames.has(mcpName)) return false  // Skip if system MCP exists
    }
  }
  return true
})
```

## SkillMcpManager — Per-Session MCP Isolation

```typescript
class SkillMcpManager {
  async disconnectSession(sessionId: string)  // Cleanup on session delete
}
```

## Skills in Delegation

```typescript
delegate_task(category="visual-engineering", load_skills=["playwright", "frontend-ui-ux"], ...)
// Skill content is prepended to subagent's system prompt via buildSystemContent()
```

---

[← Previous: Claude Code Compatibility](s09-claude-code-compat.md) | [Next: Error Recovery →](s11-error-recovery.md)
