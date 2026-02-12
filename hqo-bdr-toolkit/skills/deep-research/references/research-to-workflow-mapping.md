# Ozzy → Workflow Mapping

How Ozzy's output fields feed into each step of the prospecting workflow. Ozzy's output is canonical — Claude uses it directly rather than duplicating the research.

## Into HubSpot (Step 2 — CRM sync)

After Ozzy's response is parsed and presented to the BDR, the BDR reviews the findings and approves the HubSpot push. Once confirmed, the mapped `clay_*` fields are pushed to the HubSpot company record via `manage_crm_objects`. If the BDR declines, the data is still available in-session but not persisted to the CRM.

| Ozzy Field | HubSpot Property | Notes |
|------------|------------------|-------|
| `clay_icp`, `clay_company_tier`, `clay_icp_fit_score`, `clay_icp_fit_rating`, `clay_icp_reasons_for`, `clay_icp_reasons_against`, `clay_icp_data_gaps` | Same name (1:1 match) | ICP scoring fields — push all non-empty values. |
| `clay_portfolio_total_sf`, `clay_portfolio_num_buildings`, `clay_portfolio_markets`, `clay_portfolio_asset_classes` | Same name (1:1 match) | Portfolio fields — push all non-empty values. |
| `clay_estimated_deal_value`, `clay_recommended_action`, `clay_research_summary`, `clay_last_enriched` | Same name (1:1 match) | Summary fields — push all non-empty values. |
| `clay_portfolio_outlook` | NOT pushed | Not yet mapped in HubSpot — used in-session only. |
| `clay_sales_angle` | NOT pushed | Not yet mapped in HubSpot — used in-session only. |
| `clay_recent_news` | NOT pushed | Not yet mapped in HubSpot — used in-session only. |
| `clay_key_champions` | NOT pushed | Array/object — used in-session only for contact mapping and email personalization. |
| `clay_sources` | NOT pushed | Array/object — used in-session only for source verification and BDR context. |

**Why:** Persists research in the CRM so other BDRs and the team see findings even outside the plugin. Also means re-running research on the same company will overwrite stale data.

## Into ICP Scoring (Step 2 — Ozzy IS the ICP step)

Ozzy's ICP fields are the canonical ICP assessment. Claude presents them directly — there is no separate Claude ICP calculation.

| Ozzy Field | How It's Used |
|------------|---------------|
| `clay_icp` | ICP determination (Strong Fit, Moderate Fit, Weak Fit, Not a Fit). Presented as-is. |
| `clay_company_tier` | Tier assignment (Tier 1–4, N/A). Presented as-is. |
| `clay_icp_fit_score` | Numeric confidence score (1–10). Presented as-is. |
| `clay_icp_fit_rating` | Categorical rating. Presented as-is. |
| `clay_icp_reasons_for` | Specific hooks for why this is a good target. Feed into email personalization. |
| `clay_icp_reasons_against` | Flags for BDR to evaluate. |
| `clay_icp_data_gaps` | Transparency — what Ozzy could NOT verify. BDR decides if gaps matter. |
| `clay_recommended_action` | Go/no-go recommendation. Present prominently. |
| `clay_estimated_deal_value` | Ozzy's deal size estimate. Presented as-is. |

## Into Company Overview (Step 2 — portfolio intel)

| Ozzy Field | How It's Used |
|------------|---------------|
| `clay_portfolio_total_sf` | Verified total square footage |
| `clay_portfolio_num_buildings` | Verified property count |
| `clay_portfolio_markets` | Verified market list — also used for customer parallel matching (Step 5) |
| `clay_portfolio_asset_classes` | Verified asset classes — also used for customer reference matching (Step 5) |
| `clay_portfolio_outlook` | Recent acquisitions, divestitures, capital programs — presented as context |
| `clay_recent_news` | News and triggers from last 12 months — presented in Recent Triggers section |
| `clay_research_summary` | Narrative summary — source for Company Overview |

## Clay Contacts → HubSpot (Step 3 — contact creation)

During Contact Mapping, Clay discovers contacts that may not exist in HubSpot. After comparing Clay's results against existing HubSpot contacts (matched by email), the plugin presents any missing contacts to the BDR for approval. Once confirmed, missing contacts are created in HubSpot via `manage_crm_objects` and associated with the company record.

See `skills/prospecting/SKILL.md` → "Contact Push to HubSpot" for the full flow and rules.

## Into Contact Mapping (Step 3 — enrichment)

`clay_key_champions` is the primary enrichment source for contacts. Each champion entry includes name, title, LinkedIn, insights, and source URL.

| Intel Type | Source | How It's Used |
|------------|--------|---------------|
| LinkedIn activity | Recent posts, profile changes | Email opener: "Your recent post about [topic]..." |
| Press mentions | Quotes in publications, interviews | Email opener: "Saw your quote in [publication]..." |
| Conference appearances | Speaking engagements, panel participation | Email opener: "Enjoyed your panel at [event]..." |
| Published articles | Authored pieces, thought leadership | Email opener: "Your piece on [topic] resonated..." |
| Role tenure/transitions | How long in current role, previous roles | Context for approach angle |
| Professional background | Career trajectory, areas of expertise | Personalization depth |

**Matching:** Match Ozzy's key champions to Clay/HubSpot contacts by name and title. Add Ozzy's insights to the contact's row in the Key Contacts table.

## Into Customer Parallels (Step 5 — enrichment)

| Ozzy Field | How It's Used |
|------------|---------------|
| `clay_portfolio_markets` | Verified market list for HubSpot customer query — more accurate than Clay's estimates |
| `clay_portfolio_asset_classes` | Verified asset classes for customer reference matching |

## Into Email Generation (Step 6 — enrichment)

| Ozzy Field | Email Usage |
|------------|-------------|
| `clay_sales_angle` | **Foundation for email approach** — identifies which champions to lead with, which triggers to reference, how to frame HqO's value prop |
| `clay_key_champions` → insights | **Highest value** — conference, article, LinkedIn openers far outperform generic portfolio references |
| `clay_portfolio_num_buildings` | "across your 147 properties" — specific, not vague |
| `clay_portfolio_markets` | Name specific markets in email body |
| `clay_icp_reasons_for` | Hooks for openers — specific reasons this is a good target |
| `clay_research_summary` | Source material for personalized openers and pain framing |
| `clay_recent_news` | Timely triggers to reference in emails |

## Priority: Email Opener Sources

When generating emails, prioritize opener sources in this order:

1. **Best:** Contact-specific intel from `clay_key_champions` (conference talk, article, LinkedIn post, direct quote)
2. **Good:** Company-specific trigger from `clay_recent_news` or `clay_portfolio_outlook` (new hire, renovation, acquisition)
3. **Fallback:** Portfolio details from `clay_portfolio_*` fields
4. **Last resort:** Generic Clay enrichment data
