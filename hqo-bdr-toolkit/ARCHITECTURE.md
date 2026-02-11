# Architecture

## System Overview

```
┌──────────────────────────────────────────────────────┐
│                  BDR Workstation                      │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │              Cowork Desktop App                 │  │
│  │                                                │  │
│  │  /hqo:outreach "Brookfield Properties"         │  │
│  │       │                                        │  │
│  │       ▼                                        │  │
│  │  ┌─────────────┐  ┌────────────────────────┐   │  │
│  │  │  Skills      │  │  MCP Connectors        │   │  │
│  │  │  prospecting │  │  ├─ HubSpot (CRM)      │   │  │
│  │  │  deep-resrch │  │  ├─ Clay (Enrich)      │   │  │
│  │  │  email-gen   │  │  └─ Gmail (Drafts)     │   │  │
│  │  │  references/ │  │                        │   │  │
│  │  └──────┬──────┘  └────────────────────────┘   │  │
│  │         │                                      │  │
│  │         ├──► Step 2b: Deep Research ───────┐   │  │
│  │         │                                  │   │  │
│  │         │    ┌─────────────────────────┐    │   │  │
│  │         │    │  Sync Wrapper (Laptop)  │◄───┘   │  │
│  │         │    │  ngrok → Proxy → Ozzy   │        │  │
│  │         │    │  (OpenClaw web research) │        │  │
│  │         │    └─────────────────────────┘        │  │
│  │         │                                      │  │
│  │         ▼                                      │  │
│  │  /hqo:push-drafts                             │  │
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

1. BDR runs `/hqo:outreach Brookfield Properties` in Cowork
2. Plugin autonomously:
   - Searches HubSpot for existing records
   - Enriches via Clay (portfolio, contacts, news)
   - Scores ICP fit
   - **Runs deep research via Ozzy** (if enabled) — validates ICP, enriches portfolio details, surfaces people intel for Tier 1+2 contacts
   - If Ozzy returns "Not a Fit" → **stops and presents conflict** to BDR for confirmation
   - Maps contacts by persona, enriched with Ozzy's people intel when available
   - Validates engagement history
   - Finds customer parallels in same market (using Ozzy's verified markets when available)
   - Generates personalized emails (using people intel for openers when available)
3. BDR reviews output, runs `/hqo:push-drafts`
4. Plugin saves `drafts-brookfield-properties-2026-02-10.json` to workspace for review
5. BDR confirms in chat
6. Plugin creates Gmail drafts directly via Gmail MCP connector
7. BDR opens Gmail, reviews drafts, sends

## Why This Architecture

**Previous approach** required n8n webhooks, Gmail OAuth2 tokens, API keys, and Slack webhook URLs — too much setup for a non-technical BDR team. The Claude Projects config also loaded 500+ lines every conversation, hitting token limits.

**Current approach** uses Cowork's built-in MCP connectors + on-demand skill loading. BDRs just need:
- Cowork with the plugin installed
- HubSpot, Clay, and Gmail connectors enabled

No API keys. No webhooks. No external tools. Everything runs through the connectors the BDR already has.

**Deep research** adds an optional HTTP call to a sync wrapper running on a dedicated laptop. The wrapper calls Ozzy (OpenClaw agent) for autonomous web research — crawling company sites, press, SEC filings, LinkedIn, conference speaker lists. If the wrapper is unreachable, the workflow continues with Clay+HubSpot data only.

## MCP Connectors Required

| Connector | Purpose | Required |
|-----------|---------|----------|
| HubSpot | CRM lookup, contact mapping, customer parallels, engagement history | Yes |
| Clay | Company enrichment, contact discovery, news/triggers | Yes |
| Gmail | Create drafts directly in BDR's inbox | Yes |
| ZoomInfo | Supplemental contact enrichment | Optional |

### External Services

| Service | Protocol | Purpose | Required |
|---------|----------|---------|----------|
| Sync Wrapper → Ozzy | HTTP POST | Deep web research — ICP validation, portfolio verification, people intel | Optional |

The sync wrapper is NOT an MCP connector. It's an HTTP endpoint (`config/settings.json` → `deep_research.endpoint`) running on a dedicated laptop behind ngrok. The wrapper converts async OpenClaw agent calls into synchronous request-response.

## Token Efficiency

The plugin uses on-demand skill loading — only the relevant skill files load when a command runs, instead of the full 500-line instruction set loading every time (which was the issue with the previous Claude Projects approach).

| Component | Loads When |
|-----------|-----------|
| prospecting/SKILL.md | /hqo:research or /hqo:outreach |
| deep-research/SKILL.md | /hqo:research or /hqo:outreach (only if `deep_research.enabled`) |
| email-generation/SKILL.md | /hqo:outreach |
| references/* | Only when referenced by parent skill |
| commands/* | Only when the specific command is invoked |

## Security Notes

- Gmail drafts only — plugin never auto-sends emails
- Draft review file saved before Gmail creation — BDR confirms before anything touches Gmail
- No credentials stored in plugin files — all auth handled by Cowork connectors
- No PII stored in plugin — all data flows through MCP at runtime
- BDR settings (name, email, tone) stored locally in config/settings.json
- Deep research auth token stored in config/settings.json — only used for the sync wrapper endpoint
- Deep research is additive — if wrapper is unreachable, workflow continues without it
- Ozzy's research data flows through at runtime only — no research results cached in plugin files

## Cost

- **Cowork:** $0 incremental (HqO has Pro seats)
- **Clay:** Existing credits
- **HubSpot MCP:** Included with Pro
- **Gmail MCP:** Free (built into Cowork)
- **Deep Research (Ozzy):** OpenClaw agent running on dedicated laptop — no per-query cost beyond existing infrastructure
