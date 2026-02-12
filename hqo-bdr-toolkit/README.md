# HqO BDR Outreach Plugin

Cowork plugin for HqO BDR prospecting — research target accounts, generate personalized outreach, and create Gmail drafts automatically. No API keys, no webhooks, no external tools.

## What It Does

1. **Research** — Enriches target companies via Clay + HubSpot (portfolio size, markets, contacts, news)
2. **Score** — Calculates ICP fit and deal size tier
3. **Deep Research** — Sends `@Ozzy` in the `#ozzy` Slack channel to trigger autonomous web research. Ozzy validates ICP, verifies portfolio details, and surfaces people intel (LinkedIn, press, conferences) for key contacts. Results are read back from Slack automatically.
4. **Map** — Discovers and validates contacts by persona priority, enriched with Ozzy's people intel
5. **Write** — Generates Challenger-method emails with customer proof points and contact-specific openers
6. **Deliver** — Saves a draft review file, then creates Gmail drafts directly in your inbox

## Getting Started

### First Time? Run This:

```
/hqo:onboard
```

That's it. The onboard command walks you through everything — connector setup, Gmail signature, profile config, and a test run. Takes about 10 minutes. You don't need to read the rest of this document.

### Already Set Up?

Jump straight to prospecting:

```
/hqo:outreach Brookfield Properties
```

## Commands

| Command | Description |
|---------|-------------|
| `/hqo:onboard` | **Start here** — guided setup + test run |
| `/hqo:outreach [company]` | Full workflow: research → score → contacts → emails |
| `/hqo:research [company]` | Research only, no email generation |
| `/hqo:push-drafts` | Save draft file for review, then create Gmail drafts |
| `/hqo:setup` | Quick reconfigure (name, email, tone) without full onboarding |

## Prerequisites

- Cowork desktop app (Pro or Max plan)
- The following Cowork connectors (the onboard command helps you set these up):

| Connector | Purpose | Required |
|-----------|---------|----------|
| HubSpot | CRM lookup, contact mapping, engagement history | Yes |
| Clay | Company enrichment, contact discovery, news/triggers | Yes |
| Slack | Deep research via Ozzy (`@Ozzy` in `#ozzy` channel) + reading results | Yes |
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
│   ├── onboard.md                    # /hqo:onboard (start here)
│   ├── setup.md                      # /hqo:setup
│   ├── outreach.md                   # /hqo:outreach
│   ├── research.md                   # /hqo:research
│   └── push-drafts.md               # /hqo:push-drafts
├── skills/
│   ├── prospecting/
│   │   ├── SKILL.md                  # Core prospecting knowledge
│   │   └── references/
│   │       ├── icp-criteria.md       # ICP definition & tiering
│   │       └── customer-references.md # Customer proof point rules
│   ├── deep-research/
│   │   ├── SKILL.md                  # Ozzy deep research integration
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

The previous Claude Projects setup loaded a 500-line instruction set every conversation, hitting token limits and rate caps. This plugin uses on-demand skill loading — only the relevant files load when a command runs. Research loads prospecting knowledge; outreach adds email generation on top. References only load when the parent skill needs them.

No external tools to configure. Everything flows through Cowork's built-in MCP connectors (HubSpot, Clay, Slack, Gmail) that the BDR already has enabled.
