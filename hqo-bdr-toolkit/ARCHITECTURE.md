# Architecture

## System Overview

```
┌──────────────────────────────────────────────────────┐
│                  BDR Workstation                      │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │              Cowork Desktop App                 │  │
│  │                                                │  │
│  │  /hqo:research "Brookfield Properties"         │  │
│  │       │                                        │  │
│  │       ▼                                        │  │
│  │  ┌─────────────┐  ┌────────────────────────┐   │  │
│  │  │  Skills      │  │  MCP Connectors        │   │  │
│  │  │  prospecting │  │  ├─ HubSpot (CRM)      │   │  │
│  │  │  deep-resrch │  │  ├─ Clay (Enrich)      │   │  │
│  │  │  email-gen   │  │  ├─ Slack (Ozzy)       │   │  │
│  │  │  references/ │  │  └─ Gmail (Drafts)     │   │  │
│  │  └──────┬──────┘  └────────────────────────┘   │  │
│  │         │                                      │  │
│  │         │  Step 2: Ozzy handles ICP scoring,   │  │
│  │         │  portfolio, champions, sales angle    │  │
│  │         │                                      │  │
│  │         ▼                                      │  │
│  │  /hqo:draft-emails                            │  │
│  │       │                                        │  │
│  │       ├──► Save JSON draft file for review     │  │
│  │       └──► Gmail MCP → Drafts created          │  │
│  │                                                │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────┐
│     Gmail Drafts     │
│  BDR reviews & sends │
└──────────────────────┘
```

## Data Flow

1. BDR runs `/hqo:research Brookfield Properties` in Cowork
2. Plugin step-by-step (pausing between each step for BDR review):
   - Searches HubSpot for existing records
   - Enriches via Clay (portfolio, contacts, news)
   - **Ozzy runs automatically** — handles ICP scoring, portfolio verification, key champions, sales angle, and news via Slack
   - Ozzy's findings pushed to HubSpot (with BDR approval)
   - Maps contacts by persona, enriched with Ozzy's key champion intel
   - Validates engagement history
   - Finds customer parallels in same market (using Ozzy's verified markets/asset classes)
   - Generates personalized emails using Ozzy's sales angle and champion intel
3. BDR reviews output, runs `/hqo:draft-emails`
4. Plugin saves `drafts-brookfield-properties-2026-02-10.json` to workspace for review
5. BDR confirms in chat
6. Plugin creates Gmail drafts directly via Gmail MCP connector
7. BDR opens Gmail, reviews drafts, sends

## Why This Architecture

**Previous approach** required n8n webhooks, Gmail OAuth2 tokens, API keys, and Slack webhook URLs — too much setup for a non-technical BDR team. The Claude Projects config also loaded 500+ lines every conversation, hitting token limits.

**Current approach** uses Cowork's built-in MCP connectors + on-demand skill loading. BDRs just need:
- Cowork with the plugin installed
- HubSpot, Clay, Slack, and Gmail connectors enabled

No API keys. No webhooks. No external tools. Everything runs through the connectors the BDR already has.

**Ozzy** is Claude's research tool — it handles ICP scoring, portfolio verification, key champions, and sales angle development so Claude doesn't duplicate that work. The plugin sends a message to the `#ozzy` Slack channel, Ozzy crawls the web and posts structured JSON back, and Claude uses that output directly. If Ozzy is unavailable, Claude falls back to Clay+HubSpot data with basic ICP scoring.

## MCP Connectors Required

| Connector | Purpose | Required |
|-----------|---------|----------|
| HubSpot | CRM lookup, contact mapping, customer parallels, engagement history | Yes |
| Clay | Company enrichment, contact discovery, news/triggers | Yes |
| Slack | Ozzy research tool (ICP scoring, portfolio data, key champions, sales angles) | Yes |
| Gmail | Create drafts directly in BDR's inbox | Yes |
| ZoomInfo | Supplemental contact enrichment | Optional |

## Token Efficiency

The plugin uses on-demand skill loading — only the relevant skill files load when a command runs, instead of the full 500-line instruction set loading every time (which was the issue with the previous Claude Projects approach).

| Component | Loads When |
|-----------|-----------|
| prospecting/SKILL.md | /hqo:research |
| deep-research/SKILL.md | /hqo:research (Step 2 — Ozzy research) |
| email-generation/SKILL.md | /hqo:research (email generation step) |
| references/* | Only when referenced by parent skill |
| commands/* | Only when the specific command is invoked |

## Security Notes

- Gmail drafts only — plugin never auto-sends emails
- Draft review file saved before Gmail creation — BDR confirms before anything touches Gmail
- No credentials stored in plugin files — all auth handled by Cowork connectors
- No PII stored in plugin — all data flows through MCP at runtime
- BDR settings (name, email, tone) stored locally in config/settings.json
- Ozzy research data flows through at runtime only — no research results cached in plugin files
- If Ozzy is unreachable, workflow falls back to Clay+HubSpot data

## Cost

- **Cowork:** $0 incremental (HqO has Pro seats)
- **Clay:** Existing credits
- **HubSpot MCP:** Included with Pro
- **Gmail MCP:** Free (built into Cowork)
- **Ozzy:** OpenClaw agent running on dedicated laptop — no per-query cost beyond existing infrastructure
