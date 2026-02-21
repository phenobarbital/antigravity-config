# Navigator Frontend Next — Development Guide for Claude

## Project

Modern Svelte 5 + SvelteKit frontend for Navigator and AI-Parrot.
See @.agent/CONTEXT.md for full architectural context.

**Main Branch**: `main`

## Development Environment

### Package Management

**CRITICAL RULES:**
1. **Package Manager**: Use **`npm`** for package management
   ```bash
   npm install <package>
   npm run dev
   npm run build
   ```

2. **Post-install**: The project patches Flowbite Svelte via `scripts/postinstall/patch-flowbite-svelte.mjs`. Do NOT manually edit patched files.

3. **Dependencies**: Manage via `package.json`. Always verify a library exists in `package.json` before importing.

4. **Environment Variables**: Stored in `./env/` directory (not `.env` at root). Vite loads from this directory.

## Architecture Patterns

### Multi-Tenant Subdomain Routing
The app resolves the current client (tenant) from the subdomain in `hooks.server.ts`:
- `epson.trocdigital.io` → client slug `epson`
- `localhost:5174` → client slug `localhost`
- Client configs are defined in `src/lib/data/manual-data.ts`

### Authentication Provider Registry
Auth is pluggable via `src/lib/navauth/providers/`:
- `base.ts` — BaseAuthProvider interface
- `navigator.ts` — Navigator backend JWT auth
- `microsoft.ts` — Azure MSAL (Entra ID)
- `google.ts` — Google OAuth
- `sso.ts` — SSO passthrough
- `basic.ts` — Username/password fallback
- `registry.ts` — Provider selection per client config

### Dashboard Widget System (OOP + Runes)
Widgets use class-based architecture with Svelte 5 `$state` fields:
- **Base class**: `src/lib/dashboard/domain/widget.svelte.ts`
- **Subclasses**: `*-widget.svelte.ts` (table, chart, map, iframe, etc.)
- **Container**: `dashboard-container.svelte.ts` manages tabs and widget lifecycle
- **Data layer**: `data-source.svelte.ts` handles fetch/polling/caching per widget
- **Persistence**: `persistence.ts` saves/loads dashboard state

New widget types MUST extend `Widget` and register in `component-registry.ts`.

### Svelte 5 Runes (Non-Negotiable)
```typescript
// CORRECT — Svelte 5 Runes
let count = $state(0);
let doubled = $derived(count * 2);

// WRONG — Legacy reactive declarations
let count = 0;
$: doubled = count * 2;
```

### Store Pattern
Stores use Svelte 5 runes in `.svelte.ts` files:
- `src/lib/stores/auth.svelte.ts` — Auth state
- `src/lib/stores/client.svelte.ts` — Client/tenant state
- `src/lib/stores/websocket.svelte.ts` — WebSocket connection
- `src/lib/stores/notifications.svelte.ts` — Notification system
- `src/lib/stores/theme.svelte.js` — Dark mode toggle

### API Client Layer
Typed API functions in `src/lib/api/`:
- `http.ts` — Base HTTP client (Axios + auth header injection)
- `agent.ts` — Agent CRUD
- `chatInteraction.ts` — Chat WebSocket + HTTP endpoints
- `crew.ts` — Agent crew management
- `programs.ts` — Program/module hierarchy

### Chat System
- **WebSocket**: `src/lib/services/websocket-service.ts` — real-time messaging
- **Persistence**: `src/lib/services/chat-db.ts` — Dexie IndexedDB for offline access
- **Components**: `src/lib/components/agents/` — AgentChat, ChatBubble, ConversationList, etc.

## Non-Negotiable Rules

### Environment
- Package manager: `npm` (never `yarn`, `pnpm`, or `bun`)
- Dev server: `npm run dev` (port configured in `env/`)
- Backend proxy: Vite proxies `/api` → `http://localhost:5000` and `/ws` → `ws://localhost:5000`

### Code Standards
- All new files: TypeScript (`.ts`, `.svelte.ts`, `.svelte`)
- Svelte 5 Runes only — no legacy reactive syntax
- Flowbite Svelte for UI primitives — don't reinvent buttons, modals, inputs
- Tailwind CSS for all styling — no inline styles, no CSS modules
- `prettier --write .` before committing

### Workflow: Think → Act → Verify
1. For complex tasks: create plan in `artifacts/plan_[task_id].md` first
2. Implement incrementally
3. Run `npm run check` after ANY logic change — no exceptions
4. Run `npm run build` to verify SSR compatibility
5. Test in browser — especially SSR and subdomain routing

### Security
- Never commit API keys — use environment variables in `env/`
- Never run `rm -rf` or system-level deletions
- JWT tokens are encrypted server-side (`src/lib/server/crypto.ts`)
- Never expose server-only code to client bundles

## Key References
- Architecture & patterns: @.agent/CONTEXT.md
- Skills: @.agent/skills/
- Workflows: @.agent/workflows/
