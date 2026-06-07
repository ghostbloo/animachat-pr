# CLAUDE.md — shared (`@deprecated-claude/shared`)

Shared TypeScript types and Zod schemas consumed by both backend and frontend.

## Commands (from `deprecated-claude-app/shared/`)

```bash
npm run build       # tsc -> dist/  (REQUIRED before backend/frontend use it)
npm run dev         # tsc --watch
npm run typecheck   # tsc --noEmit
```

This package publishes from `dist/` (`main`/`types` point there). If backend or
frontend report missing/stale types from `@deprecated-claude/shared`, rebuild it.

## Files (`src/`)

- `index.ts` — barrel export.
- `types.ts` — core domain types (conversations, messages, events, models).
- `api-types.ts` — request/response shapes for the REST API.
- `grants.ts` — capability/grant types (`admin`, `mint`, `send`).
- `import-types.ts` — claude.ai / archive import formats.
- `sharing.ts` — conversation sharing types.
- `usage.ts` — token usage / cost types.

## Convention

This is the **single source of truth** for cross-package contracts. Change a shape
here, rebuild, then update backend and frontend together.
