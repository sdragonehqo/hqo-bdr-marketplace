---
name: hqo-prospecting
description: HqO BDR prospecting expertise for commercial real estate outreach. Auto-triggers when researching target accounts, identifying contacts, scoring ICP fit, validating engagement history, or generating outreach for CRE companies. Covers REX Platform value props, persona hierarchy, customer reference matching, and engagement validation rules.
---

# HqO BDR Prospecting

## Setup

If BDR name and tone aren't configured in `config/settings.json`, ask before generating emails. Check settings first. Default tone: professional and consultative.

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

## Execution Model

**Step-by-step with pauses.** Each workflow step runs independently. After completing a step and presenting its output to the BDR, STOP and wait for acknowledgment before proceeding to the next step. Never run multiple steps in a single response.

**Clean output only.** Do NOT print internal reasoning, `<thinking>` tags, tool-call rationale, or decision-tree logic in the chat. Only show the BDR the structured results for each step. All internal processing stays behind the scenes.

## Workflow

1. **Company ID** — Identify by colloquial name, search HubSpot for records, Clay enrichment (portfolio size, markets, ownership, news). **Present results, then STOP.**
2. **ICP Scoring & Research (Ozzy)** — Trigger Ozzy via `skills/deep-research/SKILL.md`. Ozzy handles ICP scoring, portfolio verification, key champions, sales angle, and news. Claude uses Ozzy's output directly — do NOT duplicate ICP scoring. If Ozzy is unavailable, fall back to Clay data with `portfolio_sf × $0.04/sf` per `references/icp-criteria.md`. Push findings to HubSpot (with BDR approval). **Present results, then STOP.**
3. **Contact Mapping** — HubSpot existing contacts + Clay discovery by title, prioritize by persona hierarchy. Enrich with Ozzy's `clay_key_champions` intel when available (match by name/title). Push Clay-discovered contacts missing from HubSpot to the CRM (with BDR approval). See `references/persona-angles.md` and contact push rules below. **Present results, then STOP.**
4. **Engagement Validation** — For CONNECTED/REPLIED contacts, pull actual engagement content. See validation rules below. (Run as part of Contact Mapping step.)
5. **Customer Parallels** — HubSpot query: lifecyclestage=customer AND matching city/state. Use Ozzy's `clay_portfolio_markets` and `clay_portfolio_asset_classes` for more accurate matching. See `references/customer-references.md`. **Present results, then STOP.**
6. **Email Generation** — Confirm BDR identity, draft per guidelines in `email-generation` skill. Use Ozzy's `clay_sales_angle` as the approach foundation. Use `clay_key_champions` insights for personalized openers. Then `/hqo:draft-emails` to push to Gmail.

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

## Contact Push to HubSpot

During Contact Mapping (Step 3), Clay may discover contacts that don't exist in HubSpot yet. These should be created as HubSpot contacts so the rest of the team has visibility.

**How it works:**

1. After Clay returns contacts and you've searched HubSpot for existing records, compare the two lists.
2. Identify contacts that Clay found but HubSpot does NOT have (match by email address).
3. Present the missing contacts to the BDR:

> Clay found **[N] contacts** that aren't in HubSpot yet:
>
> | Name | Title | Email | Persona Tier |
> |------|-------|-------|-------------|
> | [name] | [title] | [email] | Tier [1/2/3] |
> | ... | ... | ... | ... |
>
> Want me to add these to HubSpot and associate them with **[Company Name]**?

4. **Only push after the BDR approves.** Never auto-create contacts.
5. When approved, use `manage_crm_objects` to create each contact with the data Clay provided (firstname, lastname, email, jobtitle, phone, company association).
6. Associate each new contact with the company record in HubSpot.
7. Confirm what was created:

> **[N] contacts added to HubSpot** and associated with [Company Name].

**Rules:**
- **Always wait for BDR approval** before creating contacts. Never auto-push.
- Match by **email address** to determine if a contact already exists. If Clay provides an email that's already in HubSpot, skip that contact (it's a duplicate, not missing).
- Only push contacts with a valid email address. Skip contacts where Clay didn't find an email.
- If the BDR declines, continue the workflow — the Clay contacts are still available in-session for email generation.
- If any individual contact creation fails, report the failure and continue with the rest. Don't stop the whole batch.

## Output Sections

1. **Company Overview** — Name, portfolio size, markets, asset classes, ownership. Use Ozzy's verified data.
2. **Research Results** — ICP rating, confidence score, deal value, recommended action, reasons for/against, data gaps, key champions, sales angle. From Ozzy's output (see `skills/deep-research/SKILL.md` for full format).
3. **Recent Triggers** — News/events from last 12 months (from Ozzy's `clay_recent_news` and `clay_portfolio_outlook`)
4. **Key Contacts (Validated)** — Table: Name, Title, Persona, Status, Context, Action, Priority. Enriched with Ozzy's key champion insights where available.
5. **Customer Parallels** — Market + asset class matched
6. **Draft Emails** — Per email-generation skill guidelines, using Ozzy's sales angle and key champion intel

## Constraints

**Never:** Generic messaging, lead with features, ask for demos, ignore HubSpot data, reach out to existing customers, mismatched customer references, trust status labels without validation, "following up" after soft nos, re-engage SOFT_NO <6 months, contact CHURNED, generate emails without BDR identity.

**Always:** Colloquial company name, prioritize Asset Managers, connect experience to NOI, include portfolio/market details, provide ICP tier with rationale (from Ozzy), validate engagement content, query for customer parallels, sign with BDR's name, use Ozzy's key champion intel for Tier 1+2 email personalization, use Ozzy's sales angle as the email approach foundation.
