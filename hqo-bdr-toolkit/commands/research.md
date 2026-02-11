---
name: research
description: Research a target account without generating emails. Returns ICP score, company intel, contact map, and customer parallels.
arguments:
  - name: company
    description: Target company name
    required: true
---

# /hqo:research [company]

Research-only mode. Runs steps 1–4 of the outreach workflow (no email generation).

## Steps

1. **Company Identification** — HubSpot lookup + Clay enrichment
2. **ICP Scoring** — Tier assignment with deal size rationale
3. **Contact Mapping** — HubSpot contacts + Clay discovery, engagement validation
4. **Customer Parallels** — HubSpot customer query by market/asset class

## Output

- ICP Match Rating
- Company Overview (portfolio, markets, asset classes, ownership)
- Recent Triggers (news, hires, construction)
- Key Contacts table (validated statuses)
- Customer Parallels
- Recommendation: proceed to outreach or deprioritize

After review, BDR can run `/hqo:outreach {{company}}` to generate emails.
