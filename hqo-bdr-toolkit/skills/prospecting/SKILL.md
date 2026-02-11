---
name: hqo-prospecting
description: HqO BDR prospecting expertise for commercial real estate outreach. Auto-triggers when researching target accounts, identifying contacts, scoring ICP fit, validating engagement history, or generating outreach for CRE companies. Covers REX Platform value props, persona hierarchy, customer reference matching, and engagement validation rules.
---

# HqO BDR Prospecting

## Setup

If BDR name and tone aren't configured in `.claude/settings.local.json`, ask before generating emails. Check settings first. Default tone: professional and consultative.

## Company Context

**Product:** REX Platform — Tenant Experience & Engagement for Commercial Real Estate
**Value prop:** Platform connecting tenant experience to NOI; real-time data and automation across full asset lifecycle; dashboards, reporting, and asset health scores for investor-grade insight
**North star:** Qualified meetings booked → Pipeline generated (SQLs)
**Scope:** Net-new logos only. Expansion = Account Management territory.

## Challenger Reframe

Most CRE orgs manage assets, not tenant relationships. No system to measure tenant health, no way to see risk before churn, no visibility into the biggest NOI lever: tenant lifetime value.

**Shift:** From "respond when tenants complain" → "proactively manage tenant health and protect LTV of every relationship"

**Key themes:** Tenant health visibility, churn prediction, NOI protection through tenant LTV, portfolio-wide consistency, data-driven vs. reactive management.

**Capabilities to mention:** REX Platform reporting/dashboards, Tenant CRM and health scoring, Marketing/events/content platform, Automated workflows.

## Workflow

1. **Company ID** — Identify by colloquial name, search HubSpot for records, Clay enrichment (portfolio size, markets, ownership, news)
2. **ICP Scoring** — Calculate deal size (sf × $0.04), assign tier, flag timing triggers. See `references/icp-criteria.md`
2b. **Deep Research (Ozzy)** — If `config/settings.json` has `deep_research.enabled = true`, call the research wrapper for autonomous web research. Validates ICP, enriches portfolio details, surfaces people intel for Tier 1+2 contacts. See `skills/deep-research/SKILL.md`. If wrapper is unreachable, continue with Clay+HubSpot data. If Ozzy returns "Not a Fit", **stop and present the conflict to BDR** before continuing.
3. **Contact Mapping** — HubSpot existing contacts + Clay discovery by title, prioritize by persona hierarchy. Enrich Tier 1+2 contacts with deep research people intel when available. See `references/persona-angles.md`
4. **Engagement Validation** — For CONNECTED/REPLIED contacts, pull actual engagement content. See validation rules below.
5. **Customer Parallels** — HubSpot query: lifecyclestage=customer AND matching city/state. See `references/customer-references.md`
6. **Email Generation** — Confirm BDR identity, draft per guidelines in `email-generation` skill, webhook via `/hqo:push-drafts`

## Engagement Validation

Contact status labels (CONNECTED, REPLIED, ATTEMPTED) are unreliable. Always pull actual engagement content.

**Disqualifying signals:**

| Signal | Indicators | Action |
|--------|-----------|--------|
| Left company | Auto-reply departure, bounce, different LinkedIn employer | Mark CHURNED, remove, note replacement |
| Soft no | "No budget," "not a priority," "will reach out if needed" | SOFT_NO with date. 6-month cool-down. New trigger required. Never "following up." |
| Hard no | Explicit stop, unsubscribe, hostile, competitor mentioned | DO_NOT_CONTACT, remove from all |
| Referral | "Contact [Name] instead" | Add referred contact, deprioritize original |
| Stale | Last engagement >6 months | Treat as cold, not warm |
| Auto-reply false positive | Status=REPLIED but OOO/auto-response | Reclassify |

## Output Sections

1. **ICP Match Rating** — Tier (1-4) with deal size rationale
1b. **Deep Research Insights** — Confidence score, reasons for/against, data gaps, verified portfolio details, people intel, recommended action. See `skills/deep-research/SKILL.md` for full output format. Omit if deep research was unavailable.
2. **Company Overview** — Name, portfolio size, markets, asset classes, ownership. Use Ozzy's verified data when available.
3. **Recent Triggers** — News/events from last 12 months
4. **Key Contacts (Validated)** — Table: Name, Title, Persona, Status, Context, Action, Priority
5. **Customer Parallels** — Market + asset class matched
6. **Draft Emails** — Per email-generation skill guidelines

## Constraints

**Never:** Generic messaging, lead with features, ask for demos, ignore HubSpot data, reach out to existing customers, mismatched customer references, trust status labels without validation, "following up" after soft nos, re-engage SOFT_NO <6 months, contact CHURNED, generate emails without BDR identity, continue workflow after Ozzy returns "Not a Fit" without explicit BDR confirmation.

**Always:** Colloquial company name, prioritize Asset Managers, connect experience to NOI, include portfolio/market details, provide ICP tier with rationale, validate engagement content, query for customer parallels, sign with BDR's name, run deep research when enabled, use Ozzy's people intel for Tier 1+2 email personalization when available.
