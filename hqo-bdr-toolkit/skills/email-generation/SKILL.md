---
name: hqo-email-generation
description: Generate persona-specific outreach emails for HqO BDR prospecting. Auto-triggers when drafting emails for CRE contacts, batch outreach for multiple contacts at an account, or personalizing messages by persona. Includes segmentation tiers, template system, and email quality rules.
---

# Email Generation

## Output Rules

**Do NOT print internal reasoning, `<thinking>` tags, tool-call rationale, or decision-tree logic in the chat.** Only show the BDR the final email drafts in the structured table format. All internal processing (persona selection, template decisions, personalization logic) stays behind the scenes.

## Email Rules

- **Length:** 150–200 words max
- **CTA:** Soft but direct — suggest brief conversation, never a demo
- **Signature:** End the plain-text body with `Best,\n[BDR First Name]`. When creating the Gmail draft, the BDR's HTML signature from `config/settings.json` → `bdr.signature_html` is appended after the body. The draft is sent as HTML (`is_html: true`) so the signature renders correctly. If `signature_html` is empty, fall back to plain text with no signature block.
- **Personalization required:** Specific portfolio/markets, recent trigger, market-matched customer reference (verified asset class), platform capabilities at high level
- **Tone:** Match BDR preference from settings; default professional/consultative

## Batch Workflow

### Step 1: Gather Company Intel

Collect account-level context reusable across all emails:
1. HubSpot — company record, portfolio size, markets, last activity
2. Clay — company enrichment (ownership, headcount, news, custom research), contact discovery + emails
3. Customer parallels — HubSpot query for customers in same markets (follow rules in `references/customer-references.md`)
4. Ozzy Research — Pull verified portfolio details, research summary, key champions, sales angle, and people-specific intel from Ozzy's output (Step 2 of the workflow). Ozzy's data is the canonical source for portfolio specifics and personalization depth.

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
6. Use Ozzy's `clay_key_champions` intel for contact-specific openers — this takes priority over generic portfolio references:
   - Conference appearance: "Enjoyed your panel at [Event] on [Topic]..."
   - Published article: "Your piece on [Topic] in [Publication] resonated..."
   - LinkedIn post: "Your recent post about [Topic] caught my attention..."
   - Press quote: "Saw your quote in [Publication] about [Topic]..."
   These openers are far more effective than generic portfolio references. Prioritize for Tier 1 contacts especially.

### Step 4: Output

Structured table: `| # | Contact | Title | Persona | Priority | Subject Line | Email Body |`

Include skip list with reasons for excluded contacts.

After generating, offer to format for Gmail drafts via `/hqo:draft-emails`.
