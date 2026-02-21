# TASK-001: Define Lyria TypeScript types

**Feature**: ui-lyriahandler
**Spec**: sdd/specs/ui-lyriahandler.spec.md
**Status**: [ ] pending | [x] in-progress | [ ] done
**Priority**: high
**Effort**: S
**Depends-on**: none
**Assigned-to**: unassigned

## Context
Foundation types for the Lyria music generation feature. All other tasks depend on these interfaces being defined first. The types must accurately reflect the real API catalog response (genres/moods as string arrays, parameters with min/max/default, full JSON schema) and the POST request payload.

## Scope
Create `src/lib/types/lyria.ts` with TypeScript interfaces for:
1. `LyriaParameterRange` — describes a numeric parameter with min/max/default
2. `LyriaCatalog` — the GET response: `genres: string[]`, `moods: string[]`, `parameters: Record<string, LyriaParameterRange>`, `schema: Record<string, unknown>`
3. `MusicGenerationRequest` — the POST payload: `prompt` (required), `genre` (optional/nullable), `mood` (optional/nullable), `stream` (boolean), `bpm`, `temperature`, `density`, `brightness`, `timeout` (all optional with defaults from catalog)

## Files to Create/Modify
- `src/lib/types/lyria.ts` — **create** — all Lyria type definitions

## Implementation Notes
- `genres` and `moods` are plain `string[]` from the API — NOT `{value, name}` objects. The component layer will transform them into Flowbite Select `items` format.
- `genre` and `mood` in the request are nullable (`string | null`, default `null`)
- `prompt` is the only required field in the request
- `stream` controls response format: `true` = blob, `false` = Content-Disposition attachment
- Parameters (bpm, temperature, density, brightness) have specific ranges — define them accurately from the catalog schema
- `timeout` is a generation timeout, not a UI concern, but include it in the request type

## Reference Code
- See `src/lib/types/` for existing type definition patterns in the project

## Acceptance Criteria
- [ ] `LyriaCatalog` interface matches the actual GET response shape
- [ ] `MusicGenerationRequest` interface matches the JSON schema from the API
- [ ] `LyriaParameterRange` captures min/max/default for numeric params
- [ ] All fields have JSDoc descriptions
- [ ] `npm run check` passes

## Test Specification
```typescript
// Type-level tests — verified by svelte-check / tsc, no runtime tests needed
// Ensure the types compile and are importable
import type { LyriaCatalog, MusicGenerationRequest, LyriaParameterRange } from '$lib/types/lyria';
```

## Output
When complete, the agent must:
1. Move this file to `sdd/tasks/completed/`
2. Update `sdd/tasks/.index.json` status to "done"
3. Add a brief completion note below

### Completion Note
(Agent fills this in when done)
