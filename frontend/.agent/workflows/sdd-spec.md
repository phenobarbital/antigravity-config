---
description: Scaffold a Feature Specification using SDD methodology
---

# /sdd-spec — Scaffold a Feature Specification

Create a new Feature Specification for Navigator Frontend using the Spec-Driven Development methodology.
The spec becomes the Single Source of Truth (SSOT) for a feature — all requirements changes go here first.

## Guardrails
- Do NOT start implementation — this workflow only produces a specification document.
- Do NOT modify existing specs without explicit user approval.
- Always use the official template at `sdd/templates/spec.md`.

## Steps

### 1. Parse Input
Extract from the user's invocation:
- **feature-name**: first token (slug-friendly, kebab-case). If not provided, ask.
- **free-form notes**: anything after `--`, used as initial Problem Statement.

### 2. Ask Clarifying Questions
Ask the user for the following (you may ask all at once):
- **Motivation**: What problem does this feature solve? Why now?
- **Key components**: What are the main UI components, routes, or stores involved?
- **Integration points**: Which existing frontend components/APIs does this touch?
- **Acceptance criteria**: How do we know it's done?
- **Non-goals**: What is explicitly out of scope?

Use the user's answers plus any free-form notes to fill in the spec.

### 3. Assign Feature ID
- Read existing specs in `sdd/specs/` directory.
- Find the highest existing `FEAT-NNN` number and increment by 1.
- If no specs exist, start at `FEAT-001`.

### 4. Generate the Spec
1. Read the template at `sdd/templates/spec.md`.
2. Create `sdd/specs/<feature-name>.spec.md` filled in with:
   - The assigned Feature ID and today's date.
   - User's answers mapped to the template sections.
   - Suggested architectural patterns from the Svelte 5 / SvelteKit codebase (e.g., `Runes ($state, $derived)`, `Page Server Loaders (+page.server.ts)`, `UI Components (+page.svelte, custom components)`, `API Routes (+server.ts)`).
   - Component/Module breakdown (Section 3) — these will map to tasks in `/sdd-task`.
3. Set `Status: draft`.

### 5. Output and Next Steps
Print:
```
✅ Spec created: sdd/specs/<feature-name>.spec.md
   Feature ID: FEAT-<NNN>

Next steps:
  1. Review and refine the spec
  2. Mark status: approved when ready
  3. Run /sdd-task sdd/specs/<feature-name>.spec.md
```

Remind the user:
- The spec is the SSOT — changes to requirements go here first.
- Mark `status: approved` before generating tasks.

## Reference
- Template: `sdd/templates/spec.md`
- Existing specs: `sdd/specs/`
- SDD methodology: `sdd/WORKFLOW.md`
