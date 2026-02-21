---
description: Suggest next unblocked SDD tasks to assign
---

# /sdd-next — Suggest Next Tasks to Assign

Read `tasks/.index.json`, identify unblocked tasks, and suggest assignments.

## Guardrails
- This is a **read-only** workflow — do not modify any files.
- If `tasks/.index.json` does not exist, inform the user and suggest running `/sdd-task` first.

## Steps

### 1. Read the Task Index
Read `tasks/.index.json`.

### 2. Compute Unblocked Tasks
A task is **unblocked and ready** when:
- Its status is `"pending"`.
- ALL tasks in its `depends_on` list have status `"done"`.

### 3. Sort and Limit
- Sort by priority: `high` → `medium` → `low`.
- If the user provides a count `<N>`, show only the top N. Default: show all.

### 4. Print Suggestions
Output:
```
🚀 Next unblocked tasks ready to assign:

  TASK-003  [high/L]  PgVector integration
  TASK-004  [high/L]  ArangoDB integration

These can run in parallel. To start a task:

  1. Read tasks/active/TASK-003-pgvector-integration.md
  2. Follow the implementation notes and acceptance criteria
  3. On completion, move to tasks/completed/ and update the index
```

If no tasks are unblocked, print:
```
⏸️  No unblocked tasks available.
    All pending tasks have unmet dependencies.
    Run /sdd-status to see the full board.
```

## Reference
- Index file: `tasks/.index.json`
- SDD methodology: `docs/sdd/WORKFLOW.md`
