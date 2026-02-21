# Navigator Frontend SDD Workflow

## Overview

This document defines the **Spec-Driven Development (SDD)** methodology for Navigator Frontend, optimized for Claude Code and Antigravity with multi-agent task distribution.

The key idea: specifications are the Single Source of Truth (SSOT). Claude Code agents
consume spec documents and produce **Task Artifacts** — discrete, self-contained files
in `sdd/tasks/active/` that can be independently picked up and executed by any Claude Code
agent in parallel.

---

## The SDD Lifecycle

```
                                ┌─ /sdd-proposal → discuss → auto-generate spec ─┐
                                │                                                 ↓
[Human] ────────────────────────┤                                           Feature Spec → [Planner] Tasks → [Executors] Code → [Reviewer] Validation
                                │                                                 ↑              ↑                                       |
                                └──────── /sdd-spec → scaffold spec ──────────────┘              └────────── Feedback Loop ──────────────┘
```

### Phase 0 — Feature Proposal *(optional)*
Start here when the idea is not yet well-defined. Use `/sdd-proposal` to discuss
a feature in non-technical language. The agent walks through motivation, scope,
and impact with you, producing `sdd/proposals/<feature>.proposal.md`.

The proposal can then automatically scaffold a formal spec (Phase 1).

### Phase 1 — Feature Specification
Start here when you already know what you want to build. Use `/sdd-spec` to scaffold
`docs/sdd/specs/<feature>.spec.md`, or accept one auto-generated from `/sdd-proposal`.

### Phase 2 — Task Generation (Claude Code Planner Agent)
Run `/sdd-task <spec-file>` to decompose the spec into Task Artifacts.

Each task is written to `sdd/tasks/active/TASK-<id>-<slug>.md`.
The index `sdd/tasks/.index.json` is updated with task metadata.

Tasks are designed to be:
- **Atomic** — completable independently
- **Bounded** — clear scope, no ambiguity
- **Testable** — every task includes its own test criteria
- **Assignable** — formatted so any Claude Code agent can start immediately

### Phase 3 — Task Execution (Claude Code Executor Agents)
Each executor agent picks up a task file:
```bash
# In a new Claude Code session:
claude "Read sdd/tasks/active/TASK-003-sidebar-nav.md and implement it"
```

Tasks declare their dependencies, so agents know what must be done first.

### Phase 4 — Validation (Claude Code Reviewer Agent)
After execution, tasks move to `sdd/tasks/completed/`.
A reviewer agent validates against the Test Specification using `/sdd-codereview`.

---

## Task Artifact Format

Every task file (`sdd/tasks/active/TASK-<NNN>-<slug>.md`) follows this structure:

```markdown
# TASK-<NNN>: <Title>

**Feature**: <parent feature name>
**Spec**: docs/sdd/specs/<feature>.spec.md
**Status**: [ ] pending | [ ] in-progress | [x] done
**Priority**: high | medium | low
**Depends-on**: TASK-<X>, TASK-<Y>   (or "none")
**Assigned-to**: (agent session ID or "unassigned")

## Context
Brief explanation of why this task exists and how it fits the feature.

## Scope
Exactly what this task must implement. Be precise.

## Files to Create/Modify
- `src/lib/components/FeatureName/Widget.svelte` — main component
- `src/lib/stores/feature.svelte.ts` — rune-based state store
- `src/routes/(app)/feature/+page.svelte` — page integration
- `tests/unit/feature.test.ts` — Vitest unit tests

## Implementation Notes
Technical guidance for the agent: patterns to follow, existing code to reference,
gotchas, constraints.

## Reference Code
Existing patterns in the codebase the agent should follow:
- See `src/lib/components/` for component structure patterns
- See `src/lib/stores/` for rune-based state management ($state, $derived)
- See `src/routes/(app)/` for SvelteKit route/layout patterns

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] All unit tests pass: `npx vitest run tests/unit/feature.test.ts`
- [ ] Type check passes: `npx svelte-check`

## Test Specification
```typescript
// Minimal test scaffold the agent must make pass (Vitest)
import { describe, it, expect } from 'vitest';

describe('FeatureWidget', () => {
    it('should render correctly', () => {
        // ...
    });

    it('should handle edge case', () => {
        // ...
    });
});
```

## Output
When complete, the agent must:
1. Move this file to `sdd/tasks/completed/`
2. Update `sdd/tasks/.index.json` status to "done"
3. Add a brief completion note below

### Completion Note
(Agent fills this in when done)
```

---

## Task Index Schema (`sdd/tasks/.index.json`)

```json
{
  "feature": "feature-name",
  "spec": "docs/sdd/specs/feature-name.spec.md",
  "created_at": "ISO-8601",
  "tasks": [
    {
      "id": "TASK-001",
      "slug": "base-component",
      "title": "Create base Widget component",
      "status": "done",
      "priority": "high",
      "depends_on": [],
      "assigned_to": null,
      "file": "sdd/tasks/completed/TASK-001-base-component.md"
    },
    {
      "id": "TASK-002",
      "slug": "state-store",
      "title": "Implement rune-based state store",
      "status": "in-progress",
      "priority": "high",
      "depends_on": ["TASK-001"],
      "assigned_to": "session-abc123",
      "file": "sdd/tasks/active/TASK-002-state-store.md"
    }
  ]
}
```

---

## Parallelism Rules

Claude Code agents can work in parallel when tasks have no shared dependencies:

```
TASK-001 (base component)
    ├── TASK-002 (state store)       ← parallel after 001
    ├── TASK-003 (API integration)   ← parallel after 001
    └── TASK-004 (page layout)       ← parallel after 001
            └── TASK-005 (E2E tests) ← waits for 002, 003, 004
```

A Claude Code agent should **never start a task** if its `depends_on` tasks
are not in `sdd/tasks/completed/`.

---

## Commands Reference

These commands are available as both Claude Code commands (`.claude/commands/`) and
Antigravity workflows (`.agent/workflows/`):

| Command | Description |
|---|---|
| `/sdd-proposal` | Propose and discuss a feature idea before building a spec |
| `/sdd-spec` | Scaffold a new Feature Specification |
| `/sdd-task <spec.md>` | Decompose a spec into Task Artifacts |
| `/sdd-start <task>` | Start working on an SDD task by name or ID |
| `/sdd-status` | Show task index status summary |
| `/sdd-next` | Suggest next unblocked tasks to assign |
| `/sdd-codereview` | Run a code-review analysis over a completed task |

---

## Quality Rules for Agents

1. **Never modify files outside the task scope** — respect boundaries
2. **Follow existing patterns** — reference code mentioned in the task
3. **Write tests first** — TDD approach per task (Vitest for unit, Playwright for E2E)
4. **Update the index** — always update `sdd/tasks/.index.json` on completion
5. **Small commits** — one task = one logical commit
6. **Type safety** — all code must pass `svelte-check`; use TypeScript everywhere
7. **Use Svelte 5 runes** — always use `$state`, `$derived`, `$effect`, `$props`; never legacy reactive declarations
8. **Ask via the spec** — if unclear, note the ambiguity in the completion note
   and let the Planner agent refine the spec for the next iteration
