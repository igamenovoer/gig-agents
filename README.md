# gig-agents

> **Distributed AI agent orchestration — dynamic, markdown-driven, built on [CAO](https://github.com/awslabs/cli-agent-orchestrator).**

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue)](https://www.python.org/)
[![Status: Sketch](https://img.shields.io/badge/status-sketch-orange)]()

---

## What is gig-agents?

**gig-agents** is a lightweight framework for orchestrating a *team* of CLI-based AI coding agents — such as [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Codex CLI](https://github.com/openai/codex), [Gemini CLI](https://github.com/google-gemini/gemini-cli), [Kimi CLI](https://github.com/MoonshotAI/moonshot-cli), and others — as distributed, loosely coupled processes that collaborate on tasks.

The name "gig" captures the spirit: agents are hired on demand, work their shift, and can leave when done — no rigid org chart required.

### Core philosophy

| Traditional orchestration | gig-agents |
|---------------------------|------------|
| Static DAG / graph of agents | Dynamic team assembled at runtime |
| Hardcoded roles & edges | Roles declared in Markdown / prompts |
| Agents live inside one process | Agents are standalone OS processes |
| Hierarchy defined in code | Hierarchy emerges from instructions |
| User watches from outside | User is a first-class participant |

**gig-agents explicitly avoids a hardcoded orchestration graph.** Instead, agent organisation — who leads, who follows, who delegates — is determined at runtime through plain Markdown instruction files and natural-language prompts. Any agent can be the "supervisor" of a sub-task; any agent can delegate to another. The shape of the team emerges from the work, not from the code.

---

## Key Concepts

### 1. CLI agents as first-class citizens

Each agent is just a CLI tool running in its own terminal (tmux pane). Supported providers (v1):

| Agent | CLI tool | Notes |
|-------|----------|-------|
| Claude Code | `claude` | Anthropic |
| Codex CLI | `codex` | OpenAI |
| Gemini CLI | `gemini` | Google |
| Kimi CLI | `kimi` | Moonshot |
| (extensible) | any MCP-capable CLI | via provider plugin |

### 2. Dynamic join / leave

Agents are **not** spawned exclusively by gig-agents. A user can:

```bash
# Start a new Claude agent manually in a separate terminal
claude --session my-backend-agent

# Then register it with the running gig-agents runtime
gig join --name my-backend-agent --provider claude
```

Conversely, any agent can leave at any time — the runtime detects the exit and updates the team roster accordingly. The remaining agents continue working.

### 3. Markdown-driven organisation

Agent roles, responsibilities, and collaboration patterns are expressed as plain Markdown files dropped into a watched directory (or pushed via API):

```markdown
# team.md

## Supervisor
- **lead**: you are the project lead. delegate backend tasks to `backend-agent` and frontend tasks to `frontend-agent`. check in when both are done.

## Workers
- **backend-agent**: implement REST API endpoints in Python/FastAPI.
- **frontend-agent**: implement the React UI that calls the API.
```

The runtime broadcasts the relevant sections to each agent and lets *them* figure out coordination — gig-agents just delivers the instructions and keeps the channels open.

### 4. Communication channels

gig-agents provides two communication planes:

| Plane | Transport | Purpose |
|-------|-----------|---------|
| **Agent ↔ Agent** | CAO MCP server (tmux + WebSocket) | Task handoff, messaging, result sharing |
| **Human ↔ Agent** | CLI TUI / HTTP endpoints | Direct steering, monitoring, broadcasting |

### 5. Human-in-the-loop CLI TUI

A built-in terminal UI lets the user:

- View the live roster of running agents and their status.
- Send a message to one agent, a group, or all agents (`broadcast`).
- Inspect agent output in real time.
- Promote/demote agents, reassign tasks, or inject new instructions on the fly.

```
┌─────────────────────────── gig-agents TUI ────────────────────────────┐
│  Team: my-project   Runtime: 00:14:32   Agents: 3 active              │
├────────────┬──────────────┬────────────────────────────────────────────┤
│ Name       │ Provider     │ Status                                     │
├────────────┼──────────────┼────────────────────────────────────────────┤
│ lead       │ claude       │ ● working — "delegating API task..."       │
│ backend    │ codex        │ ● working — "writing /users endpoint..."   │
│ frontend   │ gemini       │ ○ idle                                     │
├────────────┴──────────────┴────────────────────────────────────────────┤
│ > send backend "add pagination support to the /users endpoint"         │
└────────────────────────────────────────────────────────────────────────┘
```

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        gig-agents runtime                        │
│                                                                  │
│   ┌──────────────┐   ┌──────────────────┐   ┌───────────────┐  │
│   │  Agent roster│   │  Instruction     │   │  CAO layer    │  │
│   │  (join/leave)│   │  broadcaster     │   │  (tmux + MCP) │  │
│   └──────┬───────┘   └───────┬──────────┘   └───────┬───────┘  │
│          │                   │                       │          │
│          └───────────────────┴───────────────────────┘          │
│                              │                                   │
│          ┌───────────────────┼───────────────────┐              │
│          ▼                   ▼                   ▼              │
│     CLI TUI              HTTP API            MCP server         │
│   (human input)       (REST endpoints)   (agent-to-agent)       │
└──────────────────────────────────────────────────────────────────┘
         │                     │                   │
    ┌────┴────┐           ┌────┴────┐         ┌────┴────┐
    │ claude  │           │  codex  │         │ gemini  │
    │ (tmux)  │           │ (tmux)  │         │ (tmux)  │
    └─────────┘           └─────────┘         └─────────┘
```

### Built on CAO

gig-agents v1 is built **on top of** [CLI Agent Orchestrator (CAO)](https://github.com/awslabs/cli-agent-orchestrator). CAO provides the foundational layer:

- tmux-based agent session management
- MCP server for inter-agent messaging
- Provider abstraction (Claude Code, Codex, Q CLI, etc.)
- Handoff / Assign / Send-message communication patterns

gig-agents extends CAO with:

- **Open roster** — agents register and deregister dynamically, not just when CAO spawns them.
- **Instruction broadcaster** — Markdown files are parsed and routed to the right agents automatically.
- **No hardcoded supervisor** — any agent can act as coordinator for any sub-task.
- **Human TUI** — a first-class terminal interface for real-time steering.
- **HTTP endpoints** — programmatic control for scripts and external tools.

---

## Project Structure (planned)

```
gig-agents/
├── src/
│   └── gig_agents/
│       ├── runtime/          # Core runtime: roster, event loop, lifecycle
│       ├── roster/           # Agent join/leave, health tracking
│       ├── broadcaster/      # Markdown instruction parsing & routing
│       ├── channels/         # Communication planes (MCP bridge, HTTP, TUI)
│       ├── providers/        # CLI agent provider plugins (claude, codex, gemini, kimi…)
│       ├── tui/              # Terminal UI (Textual-based)
│       ├── api/              # FastAPI HTTP endpoints
│       └── cli/              # `gig` CLI entry point
├── docs/
│   ├── concepts.md           # Deep-dive into core concepts
│   ├── quickstart.md         # 5-minute getting-started guide
│   └── providers/            # Per-provider setup guides
├── examples/
│   ├── hello-team/           # Minimal 2-agent example
│   └── fullstack-app/        # Frontend + backend + lead example
├── tests/
│   ├── unit/
│   └── integration/
├── pyproject.toml
├── LICENSE
├── NOTICE
└── README.md
```

---

## Roadmap

### v0.1 — Foundation (this sketch)
- [x] Project structure & README
- [ ] CAO dependency wiring (`pyproject.toml`)
- [ ] `gig` CLI skeleton (`gig start`, `gig join`, `gig list`, `gig send`)
- [ ] Agent roster (register / deregister / heartbeat)
- [ ] Basic instruction broadcaster (Markdown → agent message)

### v0.2 — Communication
- [ ] Human TUI (Textual, read-only roster view)
- [ ] HTTP REST endpoints (`/agents`, `/send`, `/broadcast`)
- [ ] TUI input pane for human-to-agent messaging

### v0.3 — Dynamic organisation
- [ ] Watched instruction directory (drop `.md` → auto-broadcast)
- [ ] Multi-provider roster (claude + codex + gemini in same session)
- [ ] Agent health monitoring & auto-eviction on exit

### v1.0 — Production-ready
- [ ] Full provider plugin system
- [ ] Persistent session snapshots
- [ ] Web dashboard (optional)
- [ ] Comprehensive docs & examples

---

## Relationship to CAO

gig-agents respects and depends on CAO. Key differences in philosophy:

| | CAO | gig-agents |
|-|-----|-----------|
| Agent topology | Supervisor + workers (defined at start) | Fluid — any agent can lead |
| Agent lifecycle | Spawned by CAO | Spawned by anyone, joined later |
| Instructions | Prompts passed at spawn time | Markdown files pushed at any time |
| Human interface | Terminal, direct tmux interaction | Dedicated TUI + HTTP API |
| Orchestration model | Code-defined workflow patterns | Emergent from markdown instructions |

---

## Requirements (planned)

- Python 3.10+
- [tmux](https://github.com/tmux/tmux) 3.3+
- [uv](https://docs.astral.sh/uv/)
- [CAO](https://github.com/awslabs/cli-agent-orchestrator) ≥ 1.1.0
- At least one supported CLI agent installed

---

## Installation (planned)

```bash
uv tool install git+https://github.com/igamenovoer/gig-agents.git
gig --help
```

---

## Quick Start (planned)

```bash
# 1. Start the gig-agents runtime
gig start --name my-project

# 2. Spawn an agent (or let the user start one manually and join)
gig spawn --name lead --provider claude
gig spawn --name worker --provider codex

# 3. Drop an instruction file to set up the team
cat > team.md << 'EOF'
## lead
You are the project lead. Delegate implementation to `worker` and review their output.

## worker
Implement tasks as delegated by `lead`. Ask for clarification if needed.
EOF
gig broadcast team.md

# 4. Open the TUI to watch and steer
gig tui
```

---

## Contributing

This project is in early sketch phase. Contributions, ideas, and feedback are very welcome — open an issue to discuss.

---

## License

Apache License 2.0 — see [LICENSE](LICENSE).

Built on [CLI Agent Orchestrator (CAO)](https://github.com/awslabs/cli-agent-orchestrator) — Apache 2.0, Copyright Amazon.com, Inc. or its affiliates.
