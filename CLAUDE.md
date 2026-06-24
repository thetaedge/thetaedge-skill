# CLAUDE.md

This file provides guidance to Claude Code when working on the thetaedge-skill project.

## Project Purpose

This project defines an OpenClaw/Claude Code skill called **thetix** that enables AI agents to interact with ThetaEdge's thetix API. The skill covers six capabilities: thetix chat, cards/boards, opportunities (covered calls, cash-secured puts), accounts, ideas, and trading (the trade cart). Trading runs through Thetix chat's command action-loop (which accepts API-key auth): the skill reads the cart via `GET /api/cart`, then stages, executes, removes, and clears trades by sending natural language ("add this to my cart", "execute", etc.) to `POST /api/thetix-chats/process`. The flow is stage → review → execute across separate turns; execution places real broker orders, so it requires explicit user confirmation after review. The cart REST mutation endpoints (`POST /api/cart/execute`, etc.) are session-only and back the UI, not the skill.

## Skill File Format

The skill is defined in `SKILL.md` using the AgentSkills specification:

- **YAML frontmatter** (between `---` delimiters) with fields:
  - `name` — Skill identifier
  - `description` — What the skill does (used by agents to decide when to invoke it)
  - `user-invocable` — Whether users can invoke directly (e.g. `/thetix`)
  - `allowed-tools` — Tools the skill can use when invoked
- **Markdown body** — Instructions the agent follows when the skill is active

## Key ThetaEdge API Endpoints

The skill interacts with these API groups (see `reference.md` for full details):

| Group | Base Path | Purpose |
|-------|-----------|---------|
| Thetix Cards | `/api/thetix-cards` | CRUD for dashboard widget cards |
| Card Collections | `/api/thetix-card-collections` | Organize cards into boards |
| Thetix Chats | `/api/thetix-chats` | Chat management and processing |
| Chat Collections | `/api/thetix-chat-collections` | Organize chats |
| Opportunities | `/api/opportunities` | Options opportunities and calculators |
| Ideas | `/api/thetix/ideas` | Read-only trading ideas from reports |
| Trade Cart | `/api/cart` | Stage/review/execute trades (only `GET /api/cart` allows API-key auth; mutations and execute are session-only) |
| Public | `/api/public/...` | Unauthenticated card/collection access |

## Scripts

The `scripts/` directory contains cross-platform and platform-specific helpers:

- **`install.py`** — Cross-platform installer (Python 3.7+, stdlib-only). Auto-detects Claude Code / OpenClaw, installs skill files, and configures credentials. Works on Windows, macOS, and Linux.
- **`doctor.py`** — Health check that verifies installation, credentials, API connectivity, and permissions. Supports `--json` for CI.
- **`lib/common.py`** — Shared utilities used by both Python scripts (platform detection, JSON merge, validation).
- **`install-thetix-skill.sh`** — Bash installer for OpenClaw on Linux/macOS.
- **`setup-openclaw-vm.sh`** — Ubuntu VM provisioning script for isolated OpenClaw environments.
- **`clawtunnel.sh`** — SSH tunnel helper for local development with a VM.

## Development Workflow

1. Edit `SKILL.md` to change the skill instructions agents follow
2. Edit `reference.md` to update API documentation
3. Test by invoking the skill in Claude Code with `/thetix`
4. Run `python3 scripts/doctor.py` to verify the installation

## Related Projects

- **ironguard-web** — The ThetaEdge webapp containing the server-side API implementation
  - Server routes: `server/main.py`
  - Card handlers: `server/modules/thetix_cards.py`
  - Chat handlers: `server/modules/thetix_chats.py`
  - Opportunity handlers: `server/modules/opportunities.py`
  - Ideas handler: `server/modules/actions.py`
  - Trade cart handlers: `server/modules/trade_cart.py`
  - Order execution engine: `server/services/order_execution_service.py`
  - Cart/order models: `server/models/trade_cart.py`, `server/models/order.py`
