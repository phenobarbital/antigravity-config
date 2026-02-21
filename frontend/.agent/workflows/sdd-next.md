---
description: Suggest next unblocked SDD tasks to assign
---

# /sdd-next — Suggest Next Tasks to Assign

Read `sdd/tasks/.index.json`, identify unblocked tasks, and suggest assignments.

## Guardrails
- This is a **read-only** workflow — do not modify any files.
- If `sdd/tasks/.index.json` does not exist, inform the user and suggest running `/sdd-task` first.

## Steps

### 1. Read the Task Index
Read `sdd/tasks/.index.json`.

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

  TASK-003  [high/L]  Theme Switcher component
  TASK-004  [high/L]  Dashboard layout

These can run in parallel. To start a task:

  1. Read sdd/tasks/active/TASK-003-theme-switcher.md
  2. Follow the implementation notes and acceptance criteria
  3. On completion, move to sdd/tasks/completed/ and update the index
```

If no tasks are unblocked, print:
```
⏸️  No unblocked tasks available.
    All pending tasks have unmet dependencies.
    Run /sdd-status to see the full board.
```

## Reference
- Index file: `sdd/tasks/.index.json`
- SDD methodology: `sdd/WORKFLOW.md`
