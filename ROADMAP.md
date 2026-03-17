# Agent Office — Roadmap

> This document is the source of truth for planned work.
> Each section maps directly to GitHub Milestones and Issues.
> Copilot / autonomous agents: pick any `[ ]` task, open a branch named `feat/<issue-slug>`, implement, and PR against `main`.

---

## How to contribute autonomously

1. Pick a task marked `[ ]` below
2. Create a branch: `git checkout -b feat/<short-slug>`
3. Implement following the patterns in `src/` (adapters) or `webview-ui/src/` (UI)
4. All magic numbers → `src/constants.ts` or `webview-ui/src/constants.ts`
5. No inline hex colors in UI files (ESLint will catch this)
6. `npm run build` must pass before opening a PR
7. PR title format: `feat: <what you did>` or `fix: <what you fixed>`

---

## Milestone v1.2 — MCP Bridge

**Goal:** any agent that speaks MCP can register with Agent Office and appear as a character.

### MCP Server (`src/mcp/`)

- [ ] **[FEAT-01] Create `src/mcp/server.ts`**
  - Express HTTP server listening on `localhost:7842` (constant: `MCP_SERVER_PORT`)
  - POST `/mcp` endpoint following MCP JSON-RPC 2.0 spec
  - WebSocket upgrade on same port for real-time event push to webview
  - Graceful shutdown on VS Code deactivate / Electron quit
  - Unit: server starts, accepts a connection, echoes back a ping

- [ ] **[FEAT-02] Define MCP tool schema in `src/mcp/protocol.ts`**
  - `agent_register({ name: string, model?: string }) → { agent_id: string }`
  - `agent_tool_start({ agent_id, tool_name, description }) → { tool_id: string }`
  - `agent_tool_done({ agent_id, tool_id }) → ok`
  - `agent_waiting({ agent_id }) → ok`  (turn complete, waiting for input)
  - `agent_heartbeat({ agent_id }) → ok`  (keep-alive, resets 30s timeout)
  - `agent_unregister({ agent_id }) → ok`
  - Export TypeScript types for all request/response shapes

- [ ] **[FEAT-03] Map MCP events → existing webview message protocol**
  - `agent_register` → `agentCreated` (reuse existing flow)
  - `agent_tool_start` → `agentToolStart`
  - `agent_tool_done` → `agentToolDone`
  - `agent_waiting` → `agentStatus: 'waiting'`
  - Disconnect / timeout → `agentClosed`
  - Keep existing message types unchanged — MCP is just another source

- [ ] **[FEAT-04] Agent timeout & cleanup**
  - Each registered agent has a 30s heartbeat window
  - If no heartbeat received, fire `agentClosed` and clean up
  - Configurable via constant `MCP_HEARTBEAT_TIMEOUT_MS`

### Generic Adapter (`src/adapters/`)

- [ ] **[FEAT-05] Extract `IAgentAdapter` interface**
  - `start(agentId, webview, persistFn): void`
  - `stop(agentId): void`
  - `onNewFile?(filePath): void`  (optional, for file-based adapters)
  - Both `ClaudeCodeAdapter` (existing JSONL logic) and `McpAgentAdapter` implement this

- [ ] **[FEAT-06] Refactor existing JSONL logic into `src/adapters/claudeCode.ts`**
  - Extract file watching + transcript parsing from `fileWatcher.ts` / `transcriptParser.ts`
  - Wrap in `ClaudeCodeAdapter implements IAgentAdapter`
  - No behavior change — purely structural refactor
  - All existing tests (manual + CI build) must still pass

- [ ] **[FEAT-07] Create `src/adapters/mcpAgent.ts`**
  - `McpAgentAdapter` handles agents connected via MCP server
  - Receives events from `src/mcp/server.ts` internal event bus
  - Translates to same webview messages as `ClaudeCodeAdapter`

### VS Code Integration

- [ ] **[FEAT-08] Start MCP server on extension activate**
  - In `extension.ts`, start server and register in `context.subscriptions`
  - Show server URL in status bar item (clickable → copies to clipboard)
  - Surface port conflict error with actionable message

- [ ] **[FEAT-09] Settings UI: show connected MCP agents**
  - In `SettingsModal.tsx`, add a "Connected Agents" section
  - List: agent name, model, connection time, heartbeat status
  - "Disconnect" button per agent

---

## Milestone v1.3 — GitHub Integration

**Goal:** agents are linked to real GitHub PRs/issues; the office visualizes their work.

### GitHub CLI Bridge (`src/github/`)

- [ ] **[FEAT-10] Create `src/github/ghClient.ts`**
  - Thin wrapper around `gh` CLI subprocess calls
  - `isAuthenticated(): Promise<boolean>`
  - `getCurrentRepo(): Promise<{ owner, name } | null>`
  - `listOpenPRs(repo): Promise<PR[]>`
  - `getPR(repo, number): Promise<PR>`
  - `listIssues(repo, assignee?): Promise<Issue[]>`
  - `getCIStatus(repo, branch): Promise<'passing' | 'failing' | 'pending' | 'none'>`
  - All calls have a 10s timeout; errors return null (don't crash the extension)

- [ ] **[FEAT-11] Create `src/github/prWatcher.ts`**
  - Poll GitHub data every 30s (constant: `GITHUB_POLL_INTERVAL_MS`)
  - Emit events: `prOpened`, `prMerged`, `prBlocked`, `ciStatusChanged`
  - Associate events with agent IDs via branch name matching
  - Start/stop tied to extension lifecycle

- [ ] **[FEAT-12] Webview message: `githubDataUpdated`**
  - Payload: `{ agentId, branch, prNumber?, prStatus?, ciStatus?, assignedIssues[] }`
  - Handler in `useExtensionMessages.ts`
  - Store per-agent GitHub state in a `Record<number, GitHubAgentData>` state slice

### Overlay UI

- [ ] **[FEAT-13] Branch name tag below agent name in `ToolOverlay.tsx`**
  - Small monospace label: `feat/my-branch` truncated at 20 chars
  - Only shown when branch data is available
  - Constant: `BRANCH_TAG_MAX_CHARS = 20`

- [ ] **[FEAT-14] PR status badge in `ToolOverlay.tsx`**
  - Colored dot: green=merged/passing, yellow=review requested, red=blocked/CI failing, gray=draft
  - Tooltip with PR title on hover
  - No badge shown when no PR data available

- [ ] **[FEAT-15] CI status animation in `characters.ts`**
  - When `ciStatus === 'failing'`, character enters a new `ERROR` state
  - Error state: character sits at desk, head slowly shaking (new 2-frame animation)
  - Uses existing `CharacterState` pattern — add `CharacterState.ERROR = 'error'`
  - Returns to normal state when CI passes

### Kanban Wall Furniture

- [ ] **[FEAT-16] Add `KANBAN_BOARD` furniture item**
  - New folder: `webview-ui/public/assets/furniture/KANBAN_BOARD/`
  - 48×64 pixel art Kanban board with 3 columns (Todo / In Progress / Done)
  - `manifest.json` with `canPlaceOnWalls: true`
  - Static sprite only (dynamic content rendered as overlay in renderer)

- [ ] **[FEAT-17] Render issue cards on Kanban board at runtime**
  - In `renderer.ts`, detect `KANBAN_BOARD` furniture instances
  - Draw small colored rectangles representing open issues over the sprite
  - Color by assignee's agent palette color
  - Max 9 cards visible (3 per column × 3 rows)

---

## Milestone v1.4 — Standalone Electron App

**Goal:** Agent Office works without VS Code.

- [ ] **[FEAT-18] Add `electron/` package**
  - `electron/main.ts`: create BrowserWindow loading `dist/webview/index.html`
  - `electron/preload.ts`: bridge `window.vscode.postMessage` to IPC
  - `package.json` script: `"app": "electron-forge start"`
  - Electron Forge config for packaging

- [ ] **[FEAT-19] Abstract VS Code API calls**
  - Create `src/platform/index.ts` with `IPlatform` interface
  - `VscodePlatform` wraps existing VS Code API calls
  - `ElectronPlatform` implements same interface using IPC + Node.js
  - `PixelAgentsViewProvider` accepts `IPlatform` in constructor

- [ ] **[FEAT-20] System tray integration (Electron)**
  - Tray icon shows agent count badge
  - Right-click menu: Show / Hide window, Quit
  - Double-click tray to show window

- [ ] **[FEAT-21] Auto-start on login (Electron, opt-in)**
  - Checkbox in Settings modal: "Launch on startup"
  - Uses `app.setLoginItemSettings()` on macOS/Windows, `.desktop` file on Linux
  - Default: off

---

## Milestone v1.5 — Agent Marketplace

**Goal:** one-click connect for popular CLI agents.

- [ ] **[FEAT-22] Agent connector config format**
  - JSON schema: `{ id, name, description, connectSteps[], mcpConfig? }`
  - Stored in `webview-ui/public/assets/agents/` (one JSON per agent type)
  - Loaded at startup, shown in a new "Connect Agent" modal

- [ ] **[FEAT-23] Built-in connector: GitHub Copilot CLI**
  - Detect `gh copilot` availability
  - Generate MCP config snippet for the user to paste
  - Show connection status in the modal

- [ ] **[FEAT-24] Built-in connector: OpenAI Codex CLI**
  - Same pattern as above

- [ ] **[FEAT-25] Built-in connector: Gemini CLI**
  - Same pattern as above

- [ ] **[FEAT-26] "Connect Agent" modal in `SettingsModal.tsx`**
  - Replace current plain list with card-based connector UI
  - Each card: logo, name, one-line description, Connect button
  - Connected agents show green checkmark + Disconnect button

---

## Ongoing / Housekeeping

- [ ] **[INFRA-01] Add unit tests for `src/mcp/server.ts`** using Vitest
- [ ] **[INFRA-02] Add unit tests for `src/github/ghClient.ts`** with mocked `gh` subprocess
- [ ] **[INFRA-03] E2E smoke test**: start server, register agent, verify webview receives `agentCreated`
- [ ] **[INFRA-04] GitHub Actions**: add test step to CI workflow
- [ ] **[DOCS-01] `docs/mcp-protocol.md`**: full tool spec with curl examples
- [ ] **[DOCS-02] `docs/adapters.md`**: guide for writing a new adapter
- [ ] **[DOCS-03] Update `CHANGELOG.md`** after each milestone release

---

## Design Constraints (read before coding)

- No inline color literals anywhere except `src/constants.ts` and `webview-ui/src/constants.ts` — ESLint will fail the build
- No `enum` — use `as const` objects
- `import type` for type-only imports (TypeScript `verbatimModuleSyntax`)
- All timing/sizing constants go in the constants files, never inline
- UI box-shadows must use `var(--pixel-shadow)` or `2px 2px 0px` pattern
- Font must reference `FS Pixel Sans`
- All new source files need a brief JSDoc comment at the top explaining what the module does
- Keep `webview-ui/src/` changes isolated to UI concerns; backend logic belongs in `src/`
