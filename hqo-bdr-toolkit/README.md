# HqO BDR Outreach Plugin

Cowork plugin for HqO BDR prospecting — research target accounts, generate personalized outreach, and create Gmail drafts automatically. No API keys, no webhooks, no external tools.

## What It Does

1. **Research** — Enriches target companies via Clay + HubSpot (portfolio size, markets, contacts, news)
2. **Score** — Ozzy handles ICP scoring, portfolio verification, key champions, and sales angle development automatically
3. **Map** — Discovers and validates contacts by persona priority, enriched with Ozzy's key champion intel
4. **Write** — Generates Challenger-method emails using Ozzy's sales angle and contact-specific openers
5. **Deliver** — Saves a draft review file, then creates Gmail drafts directly in your inbox

## Getting Started

### First Time? Run This:

```
/hqo:onboard
```

That's it. The onboard command walks you through everything — connector setup, Gmail signature, profile config, and a test run. Takes about 10 minutes. You don't need to read the rest of this document.

### Already Set Up?

Jump straight to prospecting:

```
/hqo:research Brookfield Properties
```

## Commands

| Command | Description |
|---------|-------------|
| `/hqo:onboard` | **Start here** — guided setup, test run, and settings updates |
| `/hqo:research [company]` | Full workflow: research → score → contacts → emails |
| `/hqo:draft-emails` | Save draft file for review, then create Gmail drafts |

## Prerequisites

- Cowork desktop app (Pro or Max plan)
- The following Cowork connectors (the onboard command helps you set these up):

| Connector | Purpose | Required |
|-----------|---------|----------|
| HubSpot | CRM lookup, contact mapping, engagement history | Yes |
| Clay | Company enrichment, contact discovery, news/triggers | Yes |
| Slack | Ozzy research tool (ICP scoring, portfolio data, key champions, sales angles) | Yes |
| Google Gmail | Read and search your inbox | Yes |
| Gmail | Create drafts directly in your inbox | Yes |
| ZoomInfo | Supplemental contact enrichment | Optional |

## File Structure

```
hqo-bdr-outreach/
├── .claude-plugin/plugin.json        # Plugin manifest
├── .mcp.json                         # MCP connector config
├── config/settings.json              # BDR personalization + deep research config
├── commands/
│   ├── onboard.md                    # /hqo:onboard (setup + settings)
│   ├── research.md                   # /hqo:research (full workflow)
│   └── draft-emails.md              # /hqo:draft-emails
├── skills/
│   ├── prospecting/
│   │   ├── SKILL.md                  # Core prospecting knowledge
│   │   └── references/
│   │       ├── icp-criteria.md       # ICP definition & tiering
│   │       └── customer-references.md # Customer proof point rules
│   ├── deep-research/
│   │   ├── SKILL.md                  # Ozzy research tool (ICP, portfolio, champions)
│   │   └── references/
│   │       ├── ozzy-response-schema.md       # Response field docs
│   │       └── research-to-workflow-mapping.md # Field → workflow mapping
│   └── email-generation/
│       ├── SKILL.md                  # Email writing rules
│       └── references/
│           ├── persona-angles.md     # Persona pain points & templates
│           └── gmail-draft-schema.md # Draft creation schema
├── README.md
└── ARCHITECTURE.md
```

## How It's Different From the Old Approach

The previous Claude Projects setup loaded a 500-line instruction set every conversation, hitting token limits and rate caps. This plugin uses on-demand skill loading — only the relevant files load when a command runs. References only load when the parent skill needs them.

No external tools to configure. Everything flows through Cowork's built-in MCP connectors (HubSpot, Clay, Slack, Gmail) that the BDR already has enabled.
