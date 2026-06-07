# CLAUDE.md — frontend (`@deprecated-claude/frontend`)

Vue 3 + Vuetify 3 SPA, built with Vite. TypeScript.

## Commands (from `deprecated-claude-app/frontend/`)

```bash
npm run dev       # vite dev server (port 5173, proxies API to backend :3010)
npm run build     # vite build -> dist/
npm run preview   # preview the production build
```

## Layout (`src/`)

- `views/` — routed pages (`ConversationView`, `AdminView`, `PersonasView`,
  `ArchiveView`, `LoginView`, `SharedView`, …).
- `components/` — reusable UI (e.g. `BookmarksPanel.vue`).
- `store/` — app state.
- `composables/` — reusable composition functions.
- `services/` — API client / websocket wrappers talking to the backend.
- `utils/` — markdown (`marked`), sanitization (`dompurify`), KaTeX, avatars, etc.
- `styles/` — SCSS.

## Notes

- Rendering pipeline uses `marked` + `dompurify` + `prismjs` + `katex`; preserve
  sanitization when touching message rendering.
- Conversation history is a **tree** (branches from edits/regens); the UI exposes
  branch navigation — keep that model in mind when editing `ConversationView`.
- Shared types come from `@deprecated-claude/shared`; build `shared` first if you
  see stale/missing types.
