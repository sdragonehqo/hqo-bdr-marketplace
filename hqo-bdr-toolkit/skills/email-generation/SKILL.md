---
name: hqo-email-generation
description: Generate persona-specific outreach emails for HqO BDR prospecting. Auto-triggers when drafting emails for CRE contacts, batch outreach for multiple contacts at an account, or personalizing messages by persona. Includes segmentation tiers, template system, and email quality rules.
---

# Email Generation

## Email Rules

- **Length:** 150–200 words max
- **CTA:** Soft but direct — suggest brief conversation, never a demo
- **Signature:** Just `Best,\n[BDR First Name]` — full sig appended by Gmail
- **Personalization required:** Specific portfolio/markets, recent trigger, market-matched customer reference (verified asset class), platform capabilities at high level
- **Tone:** Match BDR preference from settings; default professional/consultative

## Batch Workflow

### Step 1: Gather Company Intel

Collect account-level context reusable across all emails:
1. HubSpot — company record, portfolio size, markets, last activity
2. Clay — company enrichment (ownership, headcount, news, custom research), contact discovery + emails
3. Customer parallels — HubSpot query for customers in same markets (follow rules in `references/customer-references.md`)

### Step 2: Segment Contacts

| Tier | Personas | Approach |
|------|----------|----------|
| 1 (high) | Asset Manager, SVP/Head of Portfolio, MD, Executives | Fully bespoke — unique hook per contact |
| 2 (medium) | Property Manager, Regional Director, Experience Manager | Persona template + individual opener |
| 3 (low) | Marketing, IT, Building Engineer | Persona template, lighter personalization |
| Skip | Admin, Assistant, HR, non-ICP | Exclude with reason |

Validate engagement status for CONNECTED/REPLIED contacts per prospecting skill rules.

### Step 3: Generate Emails

For each contact, use persona templates from `references/persona-angles.md`:
1. Start with persona template
2. Customize subject line with contact-specific detail or trigger
3. Personalize opening (title, tenure, news, unique attribute)
4. Body aligned with persona pain/value
5. Soft/direct CTA (conversation, not demo)

### Step 4: Output

Structured table: `| # | Contact | Title | Persona | Priority | Subject Line | Email Body |`

Include skip list with reasons for excluded contacts.

After generating, offer to format for Gmail drafts via `/hqo:push-drafts`.
