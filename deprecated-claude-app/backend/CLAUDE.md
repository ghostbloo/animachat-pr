# CLAUDE.md — backend (`@deprecated-claude/backend`)

Express + TypeScript API server. **ESM** (`"type": "module"`) run with `tsx`.

## Commands (from `deprecated-claude-app/backend/`)

```bash
npm run dev          # tsx watch src/index.ts  (port 3010)
npm run dev:https    # same, with USE_HTTPS=true
npm run build        # tsc -> dist/
npm run typecheck    # tsc --noEmit
npm start            # node dist/index.js (after build)
```

Config via `.env` (copy `.env.example`). Requires `JWT_SECRET`; AWS Bedrock creds
and provider API keys are optional depending on which models you use.

## Layout (`src/`)

- `index.ts` — server entry (Express + WebSocket upgrade).
- `routes/` — REST endpoints, one file per resource (`conversations`, `auth`,
  `admin`, `personas`, `bookmarks`, `import`, `shares`, …).
- `services/` — provider clients (`anthropic`, `bedrock`, `gemini`, `openrouter`,
  `openai-compatible`) plus inference orchestration (`inference`,
  `enhanced-inference`), context/caching (`context-manager`, `context-strategies`,
  `cache-strategies`), import, email, pricing.
- `database/` — append-only **event-log** persistence (`persistence`,
  `bulk-event-store`, `compaction`, `migration`), blob storage, personas, shares.
  Not a SQL DB — state is rebuilt by replaying events.
- `websocket/` — streaming chat (`handler`, `room-manager`).
- `config/` — JSON config loaders (models, site config) + types.
- `middleware/` — `auth` (JWT), `rate-limit`.
- `utils/` — encryption, logging, error messages.

## Conventions

- ESM imports need explicit extensions in relative paths (`./foo.js` at runtime).
- Validate external input with Zod (schemas largely come from `shared`).
- Persist state changes as **events**; don't mutate stored records in place.
