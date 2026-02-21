# TASK-002: Implement Lyria API client

**Feature**: ui-lyriahandler
**Spec**: sdd/specs/ui-lyriahandler.spec.md
**Status**: [ ] pending | [ ] in-progress | [ ] done
**Priority**: high
**Effort**: S
**Depends-on**: TASK-001
**Assigned-to**: unassigned

## Context
Typed API client functions for the Lyria music generation backend endpoint. Uses the existing `apiClient` (Axios instance with auth injection) from `src/lib/api/http.ts`. The endpoint supports GET (catalog) and POST (generation) on the same path.

## Scope
Create `src/lib/api/lyria.ts` with:
1. `getCatalog()` — GET `/api/v1/google/generation/music` → returns `LyriaCatalog`
2. `generate(params)` — POST `/api/v1/google/generation/music` with `MusicGenerationRequest`
   - When `stream: true`: use `responseType: 'blob'`, return `Blob`
   - When `stream: false`: use `responseType: 'blob'` as well (it's a file download with Content-Disposition), return `Blob`

## Files to Create/Modify
- `src/lib/api/lyria.ts` — **create** — Lyria API client

## Implementation Notes
- Follow the `export const xxxApi = { ... }` pattern used in `src/lib/api/crew.ts` and `src/lib/api/agentConfig.ts`
- Import `apiClient` as default from `./http`
- Import types from `$lib/types/lyria`
- Both stream modes return binary data — always use `responseType: 'blob'`. The difference is semantic (stream=true for in-browser playback, stream=false for direct download), but the client handles both the same way.
- The route is publicly accessible — auth injection from apiClient still works (sends token if available, no error if not)

## Reference Code
- `src/lib/api/http.ts` — base Axios instance
- `src/lib/api/crew.ts` — object-style API pattern
- `src/lib/api/agentConfig.ts` — another object-style example

## Acceptance Criteria
- [ ] `getCatalog()` returns typed `LyriaCatalog`
- [ ] `generate()` accepts `MusicGenerationRequest` and returns `Blob`
- [ ] Uses `apiClient` from `./http` (not raw fetch)
- [ ] `responseType: 'blob'` is set on POST requests
- [ ] `npm run check` passes

## Test Specification
```typescript
import { describe, it, expect } from 'vitest';
import { lyriaApi } from '$lib/api/lyria';

describe('lyriaApi', () => {
    it('should export getCatalog function', () => {
        expect(typeof lyriaApi.getCatalog).toBe('function');
    });

    it('should export generate function', () => {
        expect(typeof lyriaApi.generate).toBe('function');
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
