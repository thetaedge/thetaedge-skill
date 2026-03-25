# CLAUDE.md

This file provides guidance to Claude Code when working on the thetaedge-skill project.

## Project Purpose

This project defines an OpenClaw/Claude Code skill called **thetix** that enables AI agents to interact with ThetaEdge's thetix API. The skill covers five capabilities: thetix chat, cards/boards, opportunities (covered calls, cash-secured puts), accounts, and ideas.

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
| Public | `/api/public/...` | Unauthenticated card/collection access |

## Development Workflow

1. Edit `SKILL.md` to change the skill instructions agents follow
2. Edit `reference.md` to update API documentation
3. Test by invoking the skill in Claude Code with `/thetix`
4. The `scripts/` directory is for future helper scripts

## Related Projects

- **ironguard-web** — The ThetaEdge webapp containing the server-side API implementation
  - Server routes: `server/main.py`
  - Card handlers: `server/modules/thetix_cards.py`
  - Chat handlers: `server/modules/thetix_chats.py`
  - Opportunity handlers: `server/modules/opportunities.py`
  - Ideas handler: `server/modules/actions.py`
