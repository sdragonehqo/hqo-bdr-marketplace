---
name: research
description: Full prospecting workflow — research a target account, run ICP scoring via Ozzy, map contacts, validate engagement, find customer parallels, and generate personalized outreach emails.
arguments:
  - name: company
    description: Target company name (colloquial name preferred, e.g. "Brookfield" not "Brookfield Asset Management Inc.")
    required: true
---

# /hqo:research [company]

Full BDR prospecting workflow for a target account. Research, score, map contacts, and generate emails — all in one command.

## Execution Model

**Each step runs independently and pauses for the BDR before continuing.** After completing a step and presenting its output, STOP and wait for the BDR to acknowledge or ask questions before moving to the next step. Do NOT run all steps back-to-back in a single response.

**Do NOT print internal reasoning, thinking, or tool-call rationale in the chat.** Keep the visible output clean — only show the BDR the structured results for each step. All internal logic (which tools to call, how to parse responses, decision trees) stays behind the scenes.

## Steps

### Step 1: Company Identification
- Search HubSpot for existing records matching `{{company}}`
- Use Clay to enrich: portfolio size, markets, ownership structure, recent news
- Identify the company by its colloquial name
- **Output:** Company name, domain, HubSpot ID, basic portfolio info
- **STOP** — present results, wait for BDR before continuing

### Step 2: ICP Scoring & Research (Ozzy)
- Trigger Ozzy via Slack per `skills/deep-research/SKILL.md`
- Ozzy handles ICP scoring, portfolio verification, key champions, and sales angle — Claude does NOT duplicate this work
- This step runs automatically every time (not optional)
- If Ozzy is unavailable, fall back to Clay + HubSpot data with basic `portfolio_sf × $0.04/sf` scoring
- Present Ozzy's full research results to BDR
- Ask BDR to approve pushing findings to HubSpot
- **Output:** Research Results (ICP rating, portfolio, key champions, sales angle — see deep-research skill for full format)
- **STOP** — present results, wait for BDR before continuing

### Step 3: Contact Mapping
- Search HubSpot for existing contacts at the company
- Use Clay to discover additional contacts by title
- Prioritize by persona hierarchy (Asset Manager → Property Manager → Experience → Influencers)
- Enrich contacts with Ozzy's `clay_key_champions` intel when available — match by name/title
- For CONNECTED/REPLIED contacts: pull actual engagement content from HubSpot, validate status
- **Output:** Key Contacts table (validated statuses, enriched with Ozzy intel where available)
- **STOP** — present results, wait for BDR before continuing

### Step 4: Customer Parallels
- Query HubSpot: `lifecyclestage = customer` AND matching city/state
- Use Ozzy's `clay_portfolio_markets` for more accurate market matching
- Match asset class before including (use Ozzy's `clay_portfolio_asset_classes`)
- Follow rules in `references/customer-references.md`
- **Output:** Customer Parallels + Company Overview + Recent Triggers
- **STOP** — present results, wait for BDR before continuing

### Step 5: Email Generation
- Confirm BDR identity (name, tone) if not already known
- Generate personalized emails per `skills/email-generation/SKILL.md`
- Use persona angles from `references/persona-angles.md`
- Use Ozzy's `clay_sales_angle` as the foundation for email approach
- Use Ozzy's `clay_key_champions` insights for personalized openers (quotes, conferences, articles)
- Include validated customer reference in each email
- **Output:** Draft emails (sorted by contact priority)
- Offer: "Run `/hqo:draft-emails` to create Gmail drafts"
