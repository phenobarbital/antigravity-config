# TASK-004: Implement MusicGeneratorForm component

**Feature**: ui-lyriahandler
**Spec**: sdd/specs/ui-lyriahandler.spec.md
**Status**: [ ] pending | [ ] in-progress | [ ] done
**Priority**: high
**Effort**: M
**Depends-on**: TASK-001, TASK-002
**Assigned-to**: unassigned

## Context
The main form component for music generation. On mount, fetches the catalog from the API to populate genre/mood dropdowns and read parameter ranges. Exposes all generation parameters and emits the filled request payload on submit.

## Scope
Create `src/lib/components/modules/MusicGenerator/MusicGeneratorForm.svelte`:

### Catalog Loading
- On mount, call `lyriaApi.getCatalog()` to fetch genres, moods, and parameter ranges
- Show a loading spinner while fetching; show error message if fetch fails
- Transform `string[]` genres/moods into Flowbite Select `items` format: `[{ value: string, name: string }]`

### Form Fields
1. **Prompt** (`<Textarea>`) — required, text description of desired music
2. **Genre** (`<Select>`) — optional, populated from catalog genres (include empty "No genre" option)
3. **Mood** (`<Select>`) — optional, populated from catalog moods (include empty "No mood" option)
4. **BPM** (`<Range>` or `<Input type="number">`) — 60-200, default 90
5. **Temperature** (`<Range>`) — 0.0-3.0, default 1.0, step 0.1 — labeled "Creativity"
6. **Density** (`<Range>`) — 0.0-1.0, default 0.5, step 0.1 — labeled "Note Density"
7. **Brightness** (`<Range>`) — 0.0-1.0, default 0.5, step 0.1 — labeled "Brightness"
8. **Stream toggle** — checkbox/toggle: stream=true for in-browser playback (default), stream=false for direct download
9. **Generate button** (`<Button>`) — disabled when prompt is empty or when `generating` prop is true

### Props
```typescript
let {
    ongenerate,
    generating = false
}: {
    ongenerate: (request: MusicGenerationRequest) => void;
    generating?: boolean;
} = $props();
```

### Behavior
- On submit, assemble `MusicGenerationRequest` from form state
- Set `genre` and `mood` to `null` if empty selection
- Call `ongenerate(request)`
- Duration/timeout: enforce max 60 seconds client-side (per spec: "limit at 1 minute or less")

## Files to Create/Modify
- `src/lib/components/modules/MusicGenerator/MusicGeneratorForm.svelte` — **create**

## Implementation Notes
- Use Svelte 5 runes: `$state`, `$derived`, `$effect`, `$props`
- Flowbite Svelte components: `Select`, `Button`, `Textarea`, `Label`, `Spinner`, `Range`, `Toggle`
- Use parameter ranges from catalog `parameters` field to set min/max/default on range inputs dynamically
- Group advanced parameters (temperature, density, brightness) in a collapsible "Advanced Settings" section to keep the default form simple
- Tailwind for layout: vertical form with `space-y-4`, labels above inputs
- The `generating` prop from the parent controls the button disabled state and shows a spinner

## Reference Code
- `src/lib/components/agents/FeedbackModal.svelte` — Label + Select + Textarea pattern
- `src/lib/components/agents/MCPServerTab.svelte` — Input + Select + Button pattern
- `src/lib/api/lyria.ts` (TASK-002) — API client to call

## Acceptance Criteria
- [ ] Catalog is fetched on mount and populates genre/mood selects
- [ ] All parameter ranges respect min/max/default from catalog
- [ ] Prompt is required — button disabled when empty
- [ ] Genre and mood send `null` when not selected
- [ ] Stream toggle works (default: true)
- [ ] Advanced settings (temperature, density, brightness) are in a collapsible section
- [ ] `ongenerate` callback is called with correct `MusicGenerationRequest` payload
- [ ] Loading state shown while catalog is loading
- [ ] Error shown if catalog fetch fails
- [ ] Uses Svelte 5 runes exclusively
- [ ] `npm run check` passes

## Test Specification
```typescript
import { describe, it, expect } from 'vitest';

describe('MusicGeneratorForm', () => {
    it('should be importable', async () => {
        const module = await import('$lib/components/modules/MusicGenerator/MusicGeneratorForm.svelte');
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
