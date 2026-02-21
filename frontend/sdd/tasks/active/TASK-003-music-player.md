# TASK-003: Implement MusicPlayer component

**Feature**: ui-lyriahandler
**Spec**: sdd/specs/ui-lyriahandler.spec.md
**Status**: [ ] pending | [ ] in-progress | [ ] done
**Priority**: high
**Effort**: S
**Depends-on**: TASK-001
**Assigned-to**: unassigned

## Context
Audio player component that receives a generated music Blob, creates an object URL, and provides playback controls and a download button. Must properly clean up object URLs to prevent memory leaks.

## Scope
Create `src/lib/components/modules/MusicGenerator/MusicPlayer.svelte`:
- Accepts `audioBlob: Blob` and optional `filename: string` (default: `lyria-generation.wav`) via `$props()`
- Creates an object URL from the blob using `URL.createObjectURL()`
- Renders an `<audio>` element with native `controls` attribute for play/pause/seek
- Renders a download `<Button>` that triggers a file download (create an `<a>` element, set `href` to object URL, set `download` to filename, click programmatically)
- Uses `$effect` with cleanup to revoke the object URL when the blob changes or the component is destroyed
- Audio format is WAV (`audio/wav` MIME type)

## Files to Create/Modify
- `src/lib/components/modules/MusicGenerator/MusicPlayer.svelte` — **create**

## Implementation Notes
- Use Svelte 5 runes exclusively: `$props()`, `$state`, `$derived`, `$effect`
- Use Flowbite Svelte `Button` for the download action
- Style with Tailwind CSS — card-like container with padding
- The `<audio>` element should have `controls` and `preload="metadata"`
- Object URL pattern:
  ```typescript
  let audioUrl = $derived(URL.createObjectURL(audioBlob));
  $effect(() => {
      // audioUrl is created from audioBlob
      return () => URL.revokeObjectURL(audioUrl);
  });
  ```
- Download pattern:
  ```typescript
  function download() {
      const a = document.createElement('a');
      a.href = audioUrl;
      a.download = filename;
      a.click();
  }
  ```
- Use `DownloadOutline` or similar icon from `flowbite-svelte-icons` if available

## Reference Code
- `src/lib/components/agents/ChatBubble.svelte` — component props pattern
- Flowbite Svelte `Button` usage in `src/lib/components/agents/AgentChat.svelte`

## Acceptance Criteria
- [ ] Component renders `<audio>` element with playback controls
- [ ] Audio source is created from the provided Blob
- [ ] Download button triggers file download with correct filename
- [ ] Object URL is revoked on component destroy (no memory leaks)
- [ ] Uses Svelte 5 runes (`$props`, `$derived`, `$effect`)
- [ ] Styled with Tailwind CSS
- [ ] `npm run check` passes

## Test Specification
```typescript
import { describe, it, expect } from 'vitest';

describe('MusicPlayer', () => {
    it('should be importable', async () => {
        const module = await import('$lib/components/modules/MusicGenerator/MusicPlayer.svelte');
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
