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
2b. **Deep Research (Ozzy)** — Autonomous web research to validate ICP and deepen portfolio/people intel. If Ozzy returns "Not a Fit", stop and present conflict to BDR. If wrapper unavailable, continue with Clay+HubSpot data. See `skills/deep-research/SKILL.md`.
3. **Contact Mapping** — HubSpot contacts + Clay discovery, engagement validation. Enriched with Ozzy's people intel for Tier 1+2 contacts when available.
4. **Customer Parallels** — HubSpot customer query by market/asset class. Uses Ozzy's verified markets when available.

## Output

- ICP Match Rating
- Deep Research Insights (confidence score, reasons for/against, data gaps, verified portfolio, people intel, recommended action — omit if unavailable)
- Company Overview (portfolio, markets, asset classes, ownership — uses verified data from deep research when available)
- Recent Triggers (news, hires, construction)
- Key Contacts table (validated statuses)
- Customer Parallels
- Recommendation: proceed to outreach or deprioritize

After review, BDR can run `/hqo:outreach {{company}}` to generate emails.
