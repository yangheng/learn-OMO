# Oh My OpenCode — Architecture Design

> A 5-10 minute architecture overview for senior developers.

## 30-Second Overview

OMO is an OpenCode plugin that turns a generic AI coding tool into a reliable engineering system through **8+ specialist Agents**, a **Hook pipeline**, and an **auto-retry loop**. The core innovation is the **Prometheus→Metis→Momus three-stage review pipeline** — inspired by GAN-style Generator/Evaluator separation. The planner (Prometheus) is *forbidden from writing code*, and the reviewer (Momus) rejects plans until satisfied, countering the AI "checking its own work" blind spot.

![Three-Stage Review Pipeline](../images/review-pipeline.webp)

---

## Key Design Decisions

| Decision | Why | Trade-off |
|----------|-----|-----------|
| Agents delegate via `delegate_task`, never shared | Each agent has isolated system prompt + tool whitelist | Extra dispatch overhead |
| Prometheus can only write `.md` (Hook-enforced) | Planner can never "just write the code" | Requires Hook interception layer |
| Momus reviews documentation completeness, not design quality | Prevents reviewer from overriding technical decisions | Won't catch bad architecture |
| Ralph Loop auto-retries until completion | AI often can't finish in one turn | Has max_iterations cap (default 100) |
| Hooks transparently modify behavior | Agents don't know they're being constrained — keeps prompts clean | Harder to debug |
| Background agents via tmux | Non-blocking parallel exploration | Requires tmux + concurrency control |

---

## Agent Orchestration Patterns

### Pattern 1: Sisyphus Solo Loop (Simple Tasks)

```
User → Sisyphus → [use tools directly] → Ralph Loop checks <promise>DONE</promise> → Done
```

### Pattern 2: Sisyphus + Specialist Delegation (Medium Tasks)

| Agent | Role | Cost |
|-------|------|------|
| **Explore** | Codebase grep | FREE |
| **Librarian** | External docs/OSS search | CHEAP |
| **Oracle** | Architecture decisions, deep reasoning | EXPENSIVE |

Default bias: **DELEGATE. Work yourself only when super simple.**

### Pattern 3: Prometheus→Metis→Momus Three-Stage Review (Complex Tasks) ⭐

This is OMO's core design, inspired by Anthropic's research on separating Planner, Generator, and Evaluator roles — analogous to GAN's Generator vs. Discriminator.

#### Stage 1: Prometheus Interviews — "What do you want?"

Prometheus is a strategic consultant **forbidden from writing code**. Enforced by Hook:

```typescript
// src/hooks/prometheus-md-only/constants.ts
export const ALLOWED_EXTENSIONS = [".md"]
export const BLOCKED_TOOLS = ["Write", "Edit", "write", "edit"]
```

From the system prompt (`src/agents/prometheus-prompt.ts`):

> **YOU ARE A PLANNER. YOU ARE NOT AN IMPLEMENTER. YOU DO NOT WRITE CODE.**
>
> When user says "do X" → Always interpret as "create a work plan for X"

Prometheus interviews the user, records decisions to `.sisyphus/drafts/*.md`, and runs a self-clearance checklist before generating a plan.

#### Stage 2: Metis Pre-screens — "What did you miss?"

Before plan generation, Prometheus consults Metis (goddess of wisdom). Metis classifies intent and catches AI failure patterns:

```typescript
// src/agents/metis.ts — AI-Slop detection
// | Scope inflation      | "Also tests for adjacent modules" |
// | Premature abstraction | "Extracted to utility"           |
// | Over-validation       | "15 error checks for 3 inputs"  |
```

Metis is **read-only** — she analyzes and advises, never modifies files.

#### Stage 3: Momus Reviews — "Not good enough. Redo it."

Named after the Greek god who found fault in everything. Momus reviews until satisfied:

```typescript
// src/agents/momus.ts
// ABSOLUTE CONSTRAINT: You are a REVIEWER, not a DESIGNER.
// The implementation direction is NOT NEGOTIABLE.
// Evaluate only: "Is this documented clearly enough to execute?"
//
// Historical Data: Plans average 7 rejections before receiving OKAY.
```

Key constraint: Momus judges **documentation completeness**, not design quality. This prevents the reviewer from overstepping.

#### The Full Pipeline

```
1. INTERVIEW → Prometheus gathers requirements → saves to drafts/
2. METIS CONSULTATION → Pre-screens for gaps
3. PLAN GENERATION → Writes to .sisyphus/plans/*.md
4. MOMUS REVIEW → Loop until OKAY verdict
5. SUMMARY → Present to user, guide to execution
```

#### GAN Analogy

| GAN Concept | OMO Equivalent | Role |
|-------------|---------------|------|
| Generator | Prometheus (plans) + Sisyphus (executes) | Produces plans and code |
| Discriminator | Momus (reviews) | Judges quality, rejects until satisfied |
| Training loop | REJECT → revise → resubmit | Adversarial iteration improves quality |

### Pattern 4: Background Parallel Execution (Large Tasks)

`BackgroundManager` (`src/features/background-agent/manager.ts`) runs agents in tmux with three-tier concurrency control: per-model → per-provider → global default (5).

Atlas (`src/agents/atlas.ts`) orchestrates large tasks through `delegate_task()` until all subtasks complete.

---

## Hook System

Hooks are OMO's invisible skeleton — agents don't know they're being modified.

| Hook | Trigger | Purpose |
|------|---------|---------|
| `prometheus-md-only` | `tool.execute.before` | Block Prometheus from writing non-`.md` files |
| `ralph-loop` | `event` (session end) | Check for completion tag, auto-retry if missing |
| `edit-error-recovery` | `tool.execute.after` | Detect edit failures, inject recovery prompt |

Pattern: **before** hooks can block/modify input; **after** hooks can modify output.

---

## Error Recovery

Three layers:

1. **Edit error recovery** — Detects `oldString not found` / `multiple matches` / `same content`, injects "READ the file first" directive
2. **Ralph Loop** — Auto-retries up to 100 iterations, checks for `<promise>DONE</promise>` tag
3. **Sisyphus 3-failure circuit breaker** — After 3 consecutive failures: STOP → REVERT → consult Oracle → ask user

---

## Code Entry Points

| I want to understand… | Read this file |
|----------------------|----------------|
| Main orchestration | `src/agents/sisyphus.ts` |
| Three-stage review — planning | `src/agents/prometheus-prompt.ts` |
| Three-stage review — pre-screening | `src/agents/metis.ts` |
| Three-stage review — review | `src/agents/momus.ts` |
| Code-write prohibition enforcement | `src/hooks/prometheus-md-only/` |
| Auto-retry loop | `src/hooks/ralph-loop/` |
| Edit error recovery | `src/hooks/edit-error-recovery/` |
| Background execution | `src/features/background-agent/` |
| Dynamic prompt assembly | `src/agents/dynamic-agent-prompt-builder.ts` |
| Architecture advisor | `src/agents/oracle.ts` |
| Codebase search | `src/agents/explore.ts` |
| External docs search | `src/agents/librarian.ts` |
| Large task orchestration | `src/agents/atlas.ts` |
