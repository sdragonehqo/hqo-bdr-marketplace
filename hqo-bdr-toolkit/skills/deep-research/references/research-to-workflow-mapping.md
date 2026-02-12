# Deep Research → Workflow Mapping

How Ozzy's output fields feed into each step of the prospecting workflow.

## Into HubSpot (Immediate — CRM sync)

After Ozzy's response is parsed, all non-empty `clay_*` fields are pushed to the HubSpot company record via `manage_crm_objects`. This happens automatically before any downstream workflow steps.

| Ozzy Field | HubSpot Property | Notes |
|------------|------------------|-------|
| All `clay_*` fields | Same name (1:1 match) | Only push non-empty values. Skip `hubspot_id` (it's the record ID, not a property). |

**Why:** Persists deep research in the CRM so other BDRs and the team see Ozzy's findings even outside the plugin. Also means re-running research on the same company will overwrite stale data.

## Into ICP Scoring (Step 2 — validation/override)

| Ozzy Field | How It's Used |
|------------|---------------|
| `clay_icp` | Confirms or contradicts initial Clay+HubSpot ICP determination. If "Not a Fit" → **hard stop**, present conflict, require BDR confirmation. |
| `clay_company_tier` | Cross-reference with initial tier from `sf × $0.04`. If different, present both with rationale. |
| `clay_icp_fit_score` | Added to output as "Deep Research Confidence Score" (1–10). |
| `clay_icp_fit_rating` | Categorical confirmation. Show alongside initial rating. |
| `clay_icp_reasons_for` | Specific hooks for why this is a good target. Feed into email personalization. |
| `clay_icp_reasons_against` | Flags for BDR to evaluate before proceeding. |
| `clay_icp_data_gaps` | Transparency — what Ozzy could NOT verify. BDR decides if gaps matter. |
| `clay_recommended_action` | Go/no-go recommendation. Present prominently. |
| `clay_estimated_deal_value` | Compare with Step 2 calculation. If significantly different, note discrepancy. |

## Into Contact Mapping (Step 3 — enrichment)

Ozzy researches Tier 1 + Tier 2 contacts only:
- **Tier 1:** Asset Manager, SVP/Head of Portfolio, MD, Executives
- **Tier 2:** Property Manager, Regional Director, Experience Manager

| Intel Type | Source | How It's Used |
|------------|--------|---------------|
| LinkedIn activity | Recent posts, profile changes | Email opener: "Your recent post about [topic]..." |
| Press mentions | Quotes in publications, interviews | Email opener: "Saw your quote in [publication]..." |
| Conference appearances | Speaking engagements, panel participation | Email opener: "Enjoyed your panel at [event]..." |
| Published articles | Authored pieces, thought leadership | Email opener: "Your piece on [topic] resonated..." |
| Role tenure/transitions | How long in current role, previous roles | Context for approach angle |
| Professional background | Career trajectory, areas of expertise | Personalization depth |

## Into Customer Parallels (Step 4 — enrichment)

| Ozzy Field | How It's Used |
|------------|---------------|
| `clay_portfolio_markets` | More accurate market list for HubSpot customer query. Use Ozzy's verified markets instead of (or in addition to) Clay's. |
| `clay_portfolio_asset_classes` | Better asset class matching for customer references. Verified > inferred. |

## Into Email Generation (Step 5 — enrichment)

| Ozzy Field | Email Usage |
|------------|-------------|
| `clay_portfolio_num_buildings` | "across your 147 properties" — specific, not vague |
| `clay_portfolio_markets` | Name specific markets in email body |
| `clay_icp_reasons_for` | Hooks for openers — the specific reasons Ozzy identified |
| `clay_research_summary` | Source material for personalized openers and pain framing |
| People intel (per contact) | **Highest value** — conference, article, LinkedIn openers far outperform generic portfolio references |

## Priority: People Intel for Email Openers

When deep research surfaces people-specific intel, it takes priority over generic company references for email personalization:

1. **Best:** Contact-specific intel (conference talk, article, LinkedIn post)
2. **Good:** Company-specific trigger from `clay_research_summary` (new hire, renovation, acquisition)
3. **Fallback:** Portfolio details from `clay_portfolio_*` fields
4. **Last resort:** Generic Clay enrichment data (existing behavior)

## Conflict Resolution

When Ozzy's data conflicts with Clay/HubSpot:

| Data Point | Resolution |
|------------|------------|
| Portfolio size | Use the more detailed source. If Ozzy found specific buildings Clay missed, prefer Ozzy's. Note discrepancy. |
| ICP tier | Present both with reasoning. BDR decides. |
| Market list | Merge both — Ozzy may have found markets Clay missed, and vice versa. |
| Contact info | HubSpot is source of truth for emails and engagement status. Ozzy enriches context, not contact details. |
| Company classification | If Ozzy says "Not a Fit" → hard stop. Present evidence. BDR confirms. |
