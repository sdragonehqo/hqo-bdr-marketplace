---
name: outreach
description: Full prospecting workflow — research a target account, score ICP fit, map contacts, validate engagement, find customer parallels, and generate personalized outreach emails.
arguments:
  - name: company
    description: Target company name (colloquial name preferred, e.g. "Brookfield" not "Brookfield Asset Management Inc.")
    required: true
---

# /hqo:outreach [company]

Run the full BDR prospecting workflow for a target account.

## Steps

1. **Company Identification**
   - Search HubSpot for existing records matching `{{company}}`
   - Use Clay to enrich: portfolio size, markets, ownership structure, recent news
   - Identify the company by its colloquial name

2. **ICP Scoring**
   - Calculate deal size: `portfolio_sf × $0.04/sf`
   - Assign tier (1–4) per `references/icp-criteria.md`
   - Flag timing triggers from Clay news results

3. **Contact Mapping**
   - Search HubSpot for existing contacts at the company
   - Use Clay to discover additional contacts by title
   - Prioritize by persona hierarchy (Asset Manager → Property Manager → Experience → Influencers)
   - For CONNECTED/REPLIED contacts: pull actual engagement content from HubSpot, validate status

4. **Customer Parallels**
   - Query HubSpot: `lifecyclestage = customer` AND matching city/state
   - Match asset class before including
   - Follow rules in `references/customer-references.md`

5. **Email Generation**
   - Confirm BDR identity (name, tone) if not already known
   - Generate personalized emails per `skills/email-generation.md`
   - Use persona angles from `references/persona-angles.md`
   - Include validated customer reference in each email

6. **Output**
   - ICP Match Rating (tier + rationale)
   - Company Overview
   - Recent Triggers
   - Key Contacts table (validated)
   - Customer Parallels
   - Draft emails (sorted by contact priority)
   - Offer: "Run `/hqo:push-drafts` to create Gmail drafts"
