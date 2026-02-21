# TASK-005: Create /music-generator page route

**Feature**: ui-lyriahandler
**Spec**: sdd/specs/ui-lyriahandler.spec.md
**Status**: [ ] pending | [ ] in-progress | [ ] done
**Priority**: high
**Effort**: M
**Depends-on**: TASK-002, TASK-003, TASK-004
**Assigned-to**: unassigned

## Context
The standalone SvelteKit page that orchestrates the full music generation flow: form submission, API call, loading state, error display, and audio playback/download. This is the integration point that wires all components together.

## Scope
Create `src/routes/music-generator/+page.svelte`:

### Layout
```
┌─────────────────────────────────────────┐
│     Lyria Music Generator               │
│     (page title + brief description)    │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │     MusicGeneratorForm            │  │
│  │  Prompt: [__________________]     │  │
│  │  [Genre ▼] [Mood ▼] BPM: [90]   │  │
│  │  [▼ Advanced Settings]            │  │
│  │  [x] Stream   [ Generate Music ]  │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │     MusicPlayer (if audioBlob)    │  │
│  │  ▶ ━━━━━━━━━━━━━━━━━━━ 0:30     │  │
│  │            [ Download ]           │  │
│  └───────────────────────────────────┘  │
│                                         │
│  (error alert if generation fails)      │
└─────────────────────────────────────────┘
```

### Page State
```typescript
let generating = $state(false);
let audioBlob = $state<Blob | null>(null);
let error = $state<string | null>(null);
```

### Flow
1. `MusicGeneratorForm` emits `ongenerate(request)`
2. Page handler:
   - Sets `generating = true`, clears `audioBlob` and `error`
   - Calls `lyriaApi.generate(request)`
   - On success: sets `audioBlob`
   - On error: sets `error` with user-friendly message
   - Sets `generating = false`
3. Conditionally renders `MusicPlayer` when `audioBlob` is not null
4. Conditionally renders error alert when `error` is not null

### Public Route
- No auth guard needed — publicly accessible
- No `+page.server.ts` needed (all data fetching is client-side)

## Files to Create/Modify
- `src/routes/music-generator/+page.svelte` — **create**

## Implementation Notes
- Use Svelte 5 runes: `$state`, `$derived`
- Import `MusicGeneratorForm` and `MusicPlayer` from `$lib/components/modules/MusicGenerator/`
- Import `lyriaApi` from `$lib/api/lyria`
- Error display: use a simple Tailwind alert div (`bg-red-100 text-red-700 p-4 rounded`) or Flowbite `Alert` if available
- Page title: use `<svelte:head><title>Lyria Music Generator</title></svelte:head>`
- Centered layout with max width: `max-w-2xl mx-auto p-6`
- The page should work in SSR — no server-side data loading, components render client-side only (the API calls happen on mount/submit)
- Consider wrapping the audio section in `{#if audioBlob}` for conditional rendering

## Reference Code
- `src/routes/+page.svelte` — existing page pattern
- `src/lib/components/modules/Finance/AgentDashboard.svelte` — form → submit → result pattern

## Acceptance Criteria
- [ ] Route `/music-generator` renders the page
- [ ] Form and player components are properly integrated
- [ ] Generation flow works: submit → loading → result OR error
- [ ] Loading spinner shown during generation
- [ ] Error messages displayed to user
- [ ] Previous audio replaced on new generation
- [ ] Page title set via `<svelte:head>`
- [ ] No auth guard — publicly accessible
- [ ] `npm run check` passes
- [ ] `npm run build` passes (SSR compatible)

## Test Specification
```typescript
import { describe, it, expect } from 'vitest';

describe('/music-generator page', () => {
    it('should be a valid SvelteKit page', async () => {
        const module = await import('../../routes/music-generator/+page.svelte');
        expect(module.default).toBeDefined();
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
