<h1 align="center">
  🕹️ Agent Office
</h1>

<h2 align="center">
  The pixel art command center for <em>any</em> AI agent — not just Claude
</h2>

<div align="center">

[![license](https://img.shields.io/github/license/your-username/agent-office?color=0183ff&style=flat)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat)](CONTRIBUTING.md)
[![MCP Compatible](https://img.shields.io/badge/MCP-compatible-blueviolet?style=flat)](docs/mcp-protocol.md)

</div>

<div align="center">
<a href="#features">Features</a> •
<a href="#quick-start">Quick Start</a> •
<a href="#supported-agents">Supported Agents</a> •
<a href="#mcp-server">MCP Server</a> •
<a href="#github-integration">GitHub Integration</a> •
<a href="#roadmap">Roadmap</a> •
<a href="#contributing">Contributing</a>
</div>

<br/>

> **Forked from [pablodelucca/pixel-agents](https://github.com/pablodelucca/pixel-agents)** — extended to support any AI agent via MCP, GitHub CLI integration, and a standalone Electron app.

---

**Agent Office** turns your multi-agent AI workflows into something you can actually see, manage, and control. Each agent becomes an animated pixel character in a shared office. They walk to their desks, animate based on what they're doing, and signal when they need your attention.

The original project was hardwired to Claude Code's JSONL transcripts. **Agent Office makes the visualization layer generic**: any agent that speaks [MCP](https://modelcontextprotocol.io) can register itself, report its status, and appear in the office.

![Agent Office screenshot](docs/screenshot.png)

---

## Features

- **Agent-agnostic** — Claude Code, GitHub Copilot CLI, Codex, Gemini CLI, or your own custom agent via the MCP bridge
- **Live activity tracking** — characters animate based on what the agent is actually doing
- **GitHub CLI integration** — agents are linked to real PRs, issues, and branches; their work shows on a Kanban wall
- **MCP server** — agents connect to a local MCP server and push status events; no file parsing hacks
- **Office layout editor** — design your workspace with floors, walls, and furniture
- **Standalone Electron app** — no VS Code required (VS Code extension also supported)
- **Speech bubbles** — permission requests and idle signals surface visually
- **Sound notifications** — ascending chime when an agent finishes its turn
- **Persistent layouts** — your office survives restarts and is shared across windows
- **Diverse characters** — 6 base skins with per-agent hue shifts

---

## Quick Start

### VS Code Extension (legacy path, still works)

```bash
git clone https://github.com/your-username/agent-office.git
cd agent-office
npm install
cd webview-ui && npm install && cd ..
npm run build
```

Press **F5** in VS Code to launch the Extension Development Host.

### Standalone App (new)

```bash
npm run app
```

Opens a desktop window — no VS Code needed. Agents connect via the built-in MCP server.

### Connect an Agent

Any agent that supports MCP can register with the local server:

```json
{
  "mcpServers": {
    "agent-office": {
      "url": "http://localhost:7842/mcp"
    }
  }
}
```

For Claude Code specifically, the original JSONL-watching path still works with zero configuration.

---

## Supported Agents

| Agent | Connection Method | Status |
|---|---|---|
| Claude Code | JSONL file watcher (original) | ✅ Stable |
| Claude Code | MCP bridge | 🚧 In Progress |
| GitHub Copilot CLI | MCP bridge | 📋 Planned |
| OpenAI Codex CLI | MCP bridge | 📋 Planned |
| Gemini CLI | MCP bridge | 📋 Planned |
| Custom / DIY | MCP SDK | ✅ Supported |

---

## MCP Server

Agent Office runs a local MCP server (default: `http://localhost:7842`) that exposes a small set of tools agents can call to report their state:

```
agent_register(name, model?)         → register and get an agent_id
agent_tool_start(agent_id, tool, description)
agent_tool_done(agent_id, tool_id)
agent_waiting(agent_id)              → signal idle / waiting for input
agent_heartbeat(agent_id)            → keep connection alive
```

The server broadcasts these events to all connected office views (VS Code webview or Electron window) via WebSocket.

See [docs/mcp-protocol.md](docs/mcp-protocol.md) for the full spec and SDK examples.

---

## GitHub Integration

When an agent is linked to a GitHub repository, Agent Office can pull in context from `gh`:

- **Branch name** shown under the character's name tag
- **PR status** (open / draft / review requested) shown as a badge
- **CI status** animates the character: green = passing, red = blocked
- **Kanban wall** furniture item displays open issues assigned to each agent
- **Merge events** trigger a confetti spawn animation

Connect a repo:

```bash
# in the office settings modal
gh auth login   # if not already authenticated
# then set the repo in Settings → GitHub Repository
```

---

## Architecture

```
agent-office/
├── src/                     # Extension / Electron backend
│   ├── mcp/                 # MCP server (new)
│   │   ├── server.ts        # Express + WS MCP endpoint
│   │   └── protocol.ts      # Tool definitions + event types
│   ├── github/              # GitHub CLI bridge (new)
│   │   ├── ghClient.ts      # gh CLI wrapper
│   │   └── prWatcher.ts     # PR/CI polling
│   ├── adapters/            # Agent adapters (new)
│   │   ├── claudeCode.ts    # Original JSONL watcher (unchanged)
│   │   └── mcpAgent.ts      # Generic MCP-connected agent
│   └── ...                  # Existing extension code
├── electron/                # Standalone app entry (new)
│   └── main.ts
└── webview-ui/              # React frontend (unchanged API surface)
```

---

## Roadmap

See [ROADMAP.md](ROADMAP.md) for the full milestone plan and GitHub Issues for individual tasks.

**v1.2 — MCP Bridge** (current sprint)
- Local MCP server that agents can connect to
- Generic agent adapter replacing hardcoded JSONL parsing
- Keep Claude Code JSONL path working as a built-in adapter

**v1.3 — GitHub Integration**
- `gh` CLI wrapper for PR/issue/CI data
- Branch name and PR status on character overlay
- Kanban wall furniture item showing issue queue

**v1.4 — Standalone Electron App**
- No VS Code dependency for basic usage
- System tray icon, auto-start on login
- Multi-window layout sync via the existing `~/.pixel-agents/layout.json` mechanism

**v1.5 — Agent Marketplace**
- Pre-built adapter configs for popular CLI agents
- One-click connect flow from the Settings modal

---

## Contributing

PRs are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for setup instructions.

The fastest way to contribute right now:
1. Write an adapter for your favorite agent CLI
2. Test the MCP bridge against a new client
3. Add a furniture item or character skin

---

## Credits

- Original pixel-agents project by [@pablodelucca](https://github.com/pablodelucca) — MIT License
- Characters based on [JIK-A-4 Metro City](https://jik-a-4.itch.io/metrocity-free-topdown-character-pack)
- All office assets open-source and included in this repository

## License

MIT — see [LICENSE](LICENSE)
