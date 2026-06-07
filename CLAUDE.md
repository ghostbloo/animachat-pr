# CLAUDE.md

Guidance for Claude Code working in this repository.

## Repo orientation

This is a **personal fork** of `anima-research/animachat` (a.k.a. Arc Chat) — a
web app for continuing conversations with current *and* deprecated Claude models
via multiple providers (Anthropic API, AWS Bedrock, OpenRouter, Gemini), with
claude.ai import, conversation branching, and event-sourced persistence.

The real application lives entirely under **`deprecated-claude-app/`**, an npm
workspaces monorepo with three packages:

| Path | Package | Stack |
|------|---------|-------|
| `deprecated-claude-app/backend`  | `@deprecated-claude/backend`  | Node 22, Express, TypeScript (ESM), WebSockets |
| `deprecated-claude-app/frontend` | `@deprecated-claude/frontend` | Vue 3, Vuetify, Vite, TypeScript |
| `deprecated-claude-app/shared`   | `@deprecated-claude/shared`   | Shared Zod schemas + TS types |

Each package has its own `CLAUDE.md` with package-specific detail.

Gotchas:
- The **top-level `README.md` is stale CI junk** ("Test deployment …"). The real
  docs are `deprecated-claude-app/README.md`.
- The **top-level `frontend/` directory is not the app** — it only holds uploaded
  avatar assets (`frontend/public/avatars/...`). The app's frontend is
  `deprecated-claude-app/frontend/`.

## Commands

Run everything from `deprecated-claude-app/` (the workspace root):

```bash
cd deprecated-claude-app
npm install                 # install all workspaces

npm run dev                 # backend (:3010) + frontend (:5173) concurrently
npm run dev:backend         # backend only  (tsx watch)
npm run dev:frontend        # frontend only (vite)

npm run build               # build order: shared -> backend -> frontend
npm run typecheck           # tsc --noEmit for shared + backend
```

Notes:
- **`shared` must be built before backend/frontend can consume it**
  (`npm run build -w shared`); the top-level `build` already does this in order.
- Node **>= 22** is required (`engines` in the root `package.json`).
- Backend config: `cp backend/.env.example backend/.env` and fill in secrets
  (`JWT_SECRET`, AWS Bedrock creds, `ANTHROPIC_API_KEY`, Resend key, …).
- There is **no real test suite yet** — `npm run test --workspaces` is a no-op
  because no workspace defines a `test` script. Don't claim "tests pass"; verify
  changes by typechecking and running the app.

## Architecture (high level)

- **Event-sourced persistence**: state is an append-only JSONL event log, not a
  relational DB. See `backend/src/database/` (`persistence.ts`,
  `bulk-event-store.ts`, `compaction.ts`, `migration.ts`). Conversation history
  is a tree — edits/regenerations create branches rather than overwriting.
- **Multi-provider inference**: `backend/src/services/` has one module per
  provider (`anthropic.ts`, `bedrock.ts`, `gemini.ts`, `openrouter.ts`,
  `openai-compatible.ts`), orchestrated by `inference.ts` /
  `enhanced-inference.ts`.
- **Context & caching**: `context-manager.ts`, `context-strategies.ts`,
  `cache-strategies.ts` implement stepped/rolling context windows tuned for
  prompt caching.
- **Realtime**: WebSocket streaming in `backend/src/websocket/`
  (`handler.ts`, `room-manager.ts`).
- **Capabilities/grants**: features are gated by capability grants
  (`admin`, `mint`, `send`) recorded as events — see `shared/src/grants.ts` and
  `backend/src/routes/admin.ts`.

## Fork & contribution workflow (IMPORTANT)

This fork keeps **personal-only files (this `CLAUDE.md`, docs) off the upstream
merge path** using a branch overlay. Get this right before committing.

Remotes:
- `origin` → `git@github.com:ghostbloo/animachat-pr.git` (our fork)
- `anima-research` → `git@github.com:anima-research/animachat.git` (upstream)

Branch model:
- **`main`** is an *exact mirror* of `anima-research/main`. Keep it
  fast-forward-only; **never** commit personal files here and never force-push it.
- **`dev`** = `main` + a single **overlay commit** that adds these `CLAUDE.md`
  files (and any future personal-only docs). This is the default base for our own
  feature work.

Rules:
- **Personal / kept work** → branch from `dev` (carries the docs).
- **Contributions to upstream** → branch from `main` (clean, no overlay), so the
  docs never appear in a PR. PR from that branch to `anima-research`.
- The CLAUDE.md files intentionally live only on `dev` and branches cut from it.
  If you're on `main` or a contribution branch, they won't exist — that's correct.

Sync with upstream:
```bash
git fetch anima-research
git checkout main && git merge --ff-only anima-research/main && git push origin main
git checkout dev  && git rebase main && git push --force-with-lease origin dev
```
(Only `dev` gets force-pushed; `main` stays ff-only.)
