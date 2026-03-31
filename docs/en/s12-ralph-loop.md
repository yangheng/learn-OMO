# S12: The Ralph Loop

> **Motto**: "Done yet? No? Keep going."

[← Previous](s11-error-recovery.md)

---

## How It Works

```
User: /ralph-loop "implement auth" --max-iterations=10
  → startLoop() writes state to filesystem
  → Agent works on task
  → Session goes idle
  → Event handler checks: did agent output <promise>done</promise>?
    YES → Clear state, show "Ralph Loop Complete!"
    NO + under max iterations → Inject continuation prompt, agent continues
    NO + at max iterations → Stop, show warning
```

## State Management

```typescript
// src/hooks/ralph-loop/index.ts
const state: RalphLoopState = {
  active: true,
  iteration: 1,
  max_iterations: 10,
  completion_promise: "done",  // Agent must output <promise>done</promise>
  prompt: "implement auth",
  session_id: sessionID,
}
writeState(ctx.directory, state, stateDir)  // Filesystem persistence
```

## Completion Detection (Dual)

1. **Transcript file**: Parse JSONL transcript for `<promise>done</promise>` in assistant messages
2. **Session API**: Fetch last assistant message and check for promise tag

## Continuation Prompt

```
[SYSTEM DIRECTIVE - RALPH LOOP 2/10]
Your previous attempt did not output the completion promise. Continue working.
- Review your progress so far
- Continue from where you left off
- When FULLY complete, output: <promise>done</promise>
Original task: {prompt}
```

Preserves original agent and model from session.

## Triggers

- Chat message template: `"You are starting a Ralph Loop" + "<user-task>..."`
- Slash command: `/ralph-loop "task" --max-iterations=10`
- ULW variant: `/ulw-loop "task"` (ultrawork mode)
- Cancel: `/cancel-ralph`

## Error Handling

- `MessageAbortedError` → Immediately clear loop (user cancelled)
- Other errors → Mark recovering for 5 seconds, then resume

---

[← Previous: Error Recovery](s11-error-recovery.md)

🎉 **Congratulations!** You've completed all 12 sessions.

[Back to README](../../README.md) | [中文 README](../../README-zh.md)
