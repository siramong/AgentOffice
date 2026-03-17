# Agent Office — Copilot Autonomous Work Instructions

This file tells GitHub Copilot (and any AI agent) how to contribute to this repo without human intervention.

---

## Project summary

Agent Office is a VS Code extension + Electron app that renders AI agents as animated pixel characters in an office. The original codebase (forked from `pablodelucca/pixel-agents`) hardwired to Claude Code's JSONL transcripts. **We are making it agent-agnostic via an MCP server and GitHub CLI integration.**

The existing code is well-structured. Read `CLAUDE.md` (inherited from the fork) for a full architecture overview before starting any work.

---

## How to pick up work

1. Read `ROADMAP.md` — every `[ ]` checkbox is an open task
2. Open the corresponding GitHub Issue (linked by `[FEAT-XX]` prefix)
3. Create a branch: `git checkout -b feat/feat-XX-short-description`
4. Implement, following the constraints below
5. Run `npm run build` — it must pass
6. Open a PR targeting `main` with title matching the issue

---

## Codebase conventions (mandatory)

### Constants
- **Never** add inline numbers or strings to source files
- Extension backend constants → `src/constants.ts`
- Webview constants → `webview-ui/src/constants.ts`
- CSS color tokens → `webview-ui/src/index.css` `:root` block

### TypeScript
- No `enum` — use `as const` objects: `export const MyThing = { A: 'a', B: 'b' } as const`
- Type-only imports must use `import type`
- `noUnusedLocals` and `noUnusedParameters` are enforced — remove unused symbols

### UI styling
- No inline hex colors (`#aabbcc`) anywhere outside `constants.ts`
- Box shadows must use `var(--pixel-shadow)` or literally `2px 2px 0px`
- Font family must reference `FS Pixel Sans`
- All UI elements: `border-radius: 0` (pixel art aesthetic)
- ESLint rules `no-inline-colors`, `pixel-shadow`, `pixel-font` are set to warn — treat them as errors

### New modules
- Every new `.ts`/`.tsx` file gets a JSDoc comment at the top explaining its purpose
- Keep backend (`src/`) and frontend (`webview-ui/src/`) concerns strictly separated

---

## Architecture touchpoints for new work

### Adding a new backend module
1. Create `src/<module>/index.ts` that exports the public API
2. Add constants to `src/constants.ts`
3. Wire into `src/extension.ts` (for VS Code) or `electron/main.ts` (for Electron) via `context.subscriptions`

### Adding a new webview message type
1. Define the message shape in `webview-ui/src/hooks/useExtensionMessages.ts`
2. Add a handler in the `handler` function switch block
3. Add the corresponding `postMessage` call in the extension backend

### Adding a new furniture asset
1. Create folder: `webview-ui/public/assets/furniture/<ID>/`
2. Add PNG sprite(s) and `manifest.json` (see existing manifests for format)
3. Asset appears automatically in the editor palette on next build

### Adding a new UI component
1. Create in `webview-ui/src/components/` (standalone) or `webview-ui/src/office/components/` (canvas-related)
2. Export from the directory's `index.ts`
3. Follow pixel art style — see `SettingsModal.tsx` and `ToolOverlay.tsx` as reference

---

## Build & test

```bash
# Full build (type-check + lint + esbuild + vite)
npm run build

# Watch mode (backend only — webview needs separate build)
npm run watch

# Lint
npm run lint
cd webview-ui && npm run lint

# Format check
npm run format:check
```

The CI workflow (`.github/workflows/ci.yml`) runs all of these. A PR cannot merge if the build fails.

---

## What NOT to do

- Do not change the `webview-ui/src/office/` rendering engine unless the task explicitly requires it
- Do not break the existing Claude Code JSONL flow — it must keep working
- Do not add dependencies without noting them in the PR description
- Do not commit `dist/`, `out/`, `*.vsix`, or `node_modules/`
- Do not add `console.log` statements (use `console.warn` or `console.error` if needed, and remove before merging)

---

## PR checklist

Before opening a PR, verify:
- [ ] `npm run build` passes locally
- [ ] No new inline color literals
- [ ] No unused variables or imports
- [ ] New constants added to the appropriate constants file
- [ ] New modules have JSDoc at the top
- [ ] PR title matches `feat: <issue title>` or `fix: <what you fixed>`
- [ ] PR body links to the relevant GitHub Issue (`Closes #XX`)
