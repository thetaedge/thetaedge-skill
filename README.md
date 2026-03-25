# thetaedge-skill

ThetaEdge Skill for OpenClaw and Claude Code — enabling AI agents to interact with **thetix**, ThetaEdge's AI-powered options income generation assistant.

## What is ThetaEdge?

ThetaEdge is an AI-powered options income generation platform. Its AI assistant, **thetix**, provides portfolio analysis, opportunity screening, covered call and cash-secured put calculations, and conversational intelligence about your positions.

## What This Skill Enables

This skill exposes five thetix capabilities to AI agents (OpenClaw or Claude Code):

1. **Thetix Chat** — Conversational AI for portfolio analysis, opportunity discussion, and dashboard queries
2. **Thetix Cards & Boards** — Dashboard widgets that materialize data visualizations, tables, and analytics from natural language queries
3. **Opportunities** — Covered call and cash-secured put screening, calculation, and analysis
4. **Accounts** — List and create user brokerage accounts
5. **Ideas** — Retrieve AI-generated trading ideas from daily and onboarding reports

## Installation

### Claude Code

Clone directly into your personal skills directory. Claude Code auto-discovers skills from `~/.claude/skills/`:

```bash
git clone https://github.com/thetaedge/thetaedge-skill.git ~/.claude/skills/thetix
```

Then add your API key to `~/.claude/settings.json`:

```json
{
  "env": {
    "THETAEDGE_API_KEY": "te_your_key_here"
  }
}
```

This is persistent and works across macOS, Linux, and Windows.

### OpenClaw

```bash
git clone https://github.com/thetaedge/thetaedge-skill.git
cd thetaedge-skill && ./scripts/install-thetix-skill.sh
```

Or add manually to your OpenClaw configuration:

```yaml
skills:
  - path: /path/to/thetaedge-skill
```

## Project Structure

```
thetaedge-skill/
├── CLAUDE.md          # Dev guide for working on this project
├── README.md          # This file
├── SKILL.md           # Main skill definition (frontmatter + instructions)
├── reference.md       # Detailed API reference for thetix endpoints
├── scripts/           # Helper scripts
│   └── .gitkeep
└── .claude/
    └── settings.local.json
```

## Development

This project defines a skill following the AgentSkills specification shared by OpenClaw and Claude Code. The skill is defined in `SKILL.md` with YAML frontmatter and markdown instructions.

- **`SKILL.md`** — The skill definition that agents consume
- **`reference.md`** — Detailed API endpoint documentation extracted from the ThetaEdge webapp
- **`CLAUDE.md`** — Guidelines for developing and maintaining this skill

### Prerequisites

- A ThetaEdge account with an active subscription
- A ThetaEdge API key (see Setup below)

## Setup

### 1. Generate an API Key

1. Log in to [ThetaEdge](https://app.thetaedge.com)
2. Go to **Profile > API Keys**
3. Click **Create Key**, give it a name (e.g., "Claude Code" or "OpenClaw"), and click create
4. Copy the key immediately — it is only shown once

### 2. Configure the Skill

Set your API key as an environment variable so the skill can authenticate:

```bash
export THETAEDGE_API_KEY="te_your_key_here"
```

Or add it to your OpenClaw skill configuration:

```json
{
  "skills": {
    "entries": {
      "thetix": {
        "enabled": true,
        "env": {
          "THETAEDGE_API_KEY": "te_your_key_here"
        }
      }
    }
  }
}
```

### 3. Verify

Invoke the skill with `/thetix` and ask it to list your card collections. If the API key is valid, you should see your thetix boards.
