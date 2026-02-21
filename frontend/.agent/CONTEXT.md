# Navigator Frontend Next — Architectural Context

## What is Navigator Frontend Next
Modern Svelte 5 + SvelteKit frontend for the **Navigator** ecosystem.
Provides the user interface for **AI-Parrot** agents, data dashboards,
and administrative tools. Multi-tenant by design — a single deployment
serves multiple clients resolved via subdomain.

---

## Core Abstractions (always extend or reuse these)

### Widget (OOP + Runes)
Base class for all dashboard widgets.
Location: `src/lib/dashboard/domain/widget.svelte.ts`
- Uses `$state` fields for reactive properties (title, icon, loading, data, etc.)
- Manages its own lifecycle: `attach()`, `detach()`, `close()`, `refresh()`
- Integrates `DataSource` for automatic data fetching, polling, and caching
- Subclass for each widget type (table, chart, map, iframe, image, video, etc.)

### DashboardTab
Groups widgets into a named tab with a grid layout.
Location: `src/lib/dashboard/domain/dashboard-tab.svelte.ts`

### DashboardContainer
Top-level orchestrator for tabs, widget creation, and persistence.
Location: `src/lib/dashboard/domain/dashboard-container.svelte.ts`

### DataSource
Reactive data fetching layer attached to widgets.
Location: `src/lib/dashboard/domain/data-source.svelte.ts`
- Supports REST endpoints, QuerySource queries, and static data
- Built-in polling, caching (`$state`), error handling

### BaseAuthProvider
Pluggable auth interface.
Location: `src/lib/navauth/providers/base.ts`
- Implementations: Navigator JWT, Microsoft (MSAL), Google, SSO, Basic
- Selected per client via `registry.ts`
- Never call auth SDKs directly — go through the provider interface

### API Client
Typed HTTP functions for the ai-parrot backend.
Location: `src/lib/api/`
- `http.ts` — Axios instance with auth header injection and base URL
- Endpoint modules: `agent.ts`, `chatInteraction.ts`, `crew.ts`, `programs.ts`, etc.

---

## Key Patterns to Follow

### Svelte 5 Runes (mandatory)
```svelte
<script lang="ts">
  // Props
  let { title, onSave }: { title: string; onSave: () => void } = $props();

  // Reactive state
  let count = $state(0);
  let doubled = $derived(count * 2);

  // Side effects
  $effect(() => {
    console.log('count changed:', count);
  });
</script>
```

### Creating a New Widget
```typescript
// src/lib/dashboard/domain/my-widget.svelte.ts
import { Widget } from './widget.svelte.js';

export class MyWidget extends Widget {
    get type(): string { return 'my-widget'; }

    constructor(config: WidgetConfig) {
        super(config);
        // custom init
    }
}
```
Register in `src/lib/dashboard/domain/component-registry.ts`.

### Adding a New Auth Provider
```typescript
// src/lib/navauth/providers/my-provider.ts
import type { BaseAuthProvider } from './base.js';

export class MyProvider implements BaseAuthProvider {
    // implement login(), logout(), getToken(), etc.
}
```
Register in `src/lib/navauth/providers/registry.ts`.

### Store pattern (runes in .svelte.ts)
```typescript
// src/lib/stores/my-store.svelte.ts
let items = $state<Item[]>([]);
let loading = $state(false);

export function getItems() { return items; }
export async function fetchItems() {
    loading = true;
    items = await api.getItems();
    loading = false;
}
```

### API calls (always typed, always through api/)
```typescript
// src/lib/api/my-resource.ts
import { http } from './http.js';

export async function getThings(): Promise<Thing[]> {
    const { data } = await http.get<Thing[]>('/api/v1/things');
    return data;
}
```

---

## What Lives Where
```
src/
├── app.html                 # HTML shell
├── app.css                  # Global Tailwind imports + custom styles
├── hooks.server.ts          # Subdomain → client resolution, JWT decode
├── components/              # Top-level shared components (Toast, BotCard)
├── lib/
│   ├── api/                 # Typed API client functions (ai-parrot backend)
│   ├── auth.ts              # Auth utility exports
│   ├── components/
│   │   ├── agents/          # AgentChat, ChatBubble, ConversationList, DataTable, DataChart
│   │   ├── common/          # Shared UI components
│   │   ├── modules/         # Module-specific components (per program)
│   │   └── visualizations/  # Visualization wrappers
│   ├── config.ts            # Runtime config (API URL, storage namespace)
│   ├── config/              # Static config files
│   ├── dashboard/
│   │   ├── components/      # Svelte components for the dashboard UI
│   │   │   ├── dashboard/   # Dashboard chrome (toolbar, sidebar)
│   │   │   ├── grids/       # Grid layout components
│   │   │   ├── layouts/     # Layout strategy components
│   │   │   ├── modals/      # Dashboard-specific modals
│   │   │   ├── settings/    # Widget settings panels
│   │   │   └── widgets/     # Svelte wrappers for Widget classes
│   │   └── domain/          # Domain model (Widget, DashboardTab, DataSource, etc.)
│   ├── data/                # Static client/tenant data (manual-data.ts)
│   ├── navauth/             # Auth providers and store
│   │   ├── providers/       # Auth provider implementations
│   │   ├── components/      # Login components
│   │   └── store.svelte.ts  # Auth state management
│   ├── server/              # Server-only code (crypto.ts)
│   ├── services/            # Chat DB (Dexie), WebSocket service
│   ├── stores/              # App-wide Svelte 5 rune stores
│   ├── types/               # TypeScript type definitions
│   └── utils/               # Shared utilities (markdown, layout, conversation)
├── routes/
│   ├── +layout.svelte       # Root layout (auth guard, sidebar)
│   ├── +page.svelte         # Home / landing
│   ├── auth/                # Auth callback routes
│   ├── login/               # Login page
│   ├── program/             # Dynamic program routes
│   │   └── [slug]/[module]/[submodule]/  # Nested program navigation
│   ├── programs/            # Programs listing
│   └── share/               # Public share routes
```

---

## What NOT to Do
- Never use legacy Svelte reactive syntax (`$:`, `let x = 0` for reactivity)
- Never bypass the auth provider registry — always use the pluggable interface
- Never call backend APIs with raw `fetch` — use the `src/lib/api/` typed clients
- Never add inline styles — use Tailwind utility classes
- Never commit environment files from `env/` — they contain secrets
- Never modify patched Flowbite Svelte files directly — edit `patch-flowbite-svelte.mjs`
- Never import server-only modules (`$lib/server/`) in client-side code

---

## Backend Integration
- **AI-Parrot**: REST API at `/api/v1/` (proxied by Vite in dev)
- **WebSocket**: Chat at `/ws` (proxied by Vite in dev)
- **Auth**: JWT tokens issued by Navigator backend, decoded in `hooks.server.ts`
- See `ai-parrot` repo for backend CONTEXT.md

---

## Current Active Development
Branch: `main`

Active areas (check these before modifying):
- `src/lib/dashboard/` — Widget system and dashboard UI
- `src/lib/components/agents/` — Agent chat interface
- `src/lib/navauth/` — Auth provider system
- `src/lib/stores/` — App state management
- `src/lib/services/` — Chat persistence and WebSocket
