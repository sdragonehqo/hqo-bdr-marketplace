# Ozzy Response Schema

Ozzy (OpenClaw agent) returns structured JSON from autonomous web research. All fields use the `clay_` prefix — these come from Ozzy's research wrapper, NOT from the Clay MCP connector.

## Response Fields

| Field | Type | Description | Used In |
|-------|------|-------------|---------|
| `hubspot_id` | string | HubSpot record ID (passed through from request) | Record matching |
| `clay_icp` | string | ICP determination: "Strong Fit", "Moderate Fit", "Weak Fit", "Not a Fit" | ICP validation |
| `clay_company_tier` | string | Tier assignment: "Tier 1", "Tier 2", "Tier 3", "Tier 4", "N/A" | ICP scoring |
| `clay_icp_fit_score` | string | Numeric fit score (1–10) | ICP output |
| `clay_icp_fit_rating` | string | Categorical: "Strong", "Moderate", "Weak", "Not a Fit" | ICP output |
| `clay_icp_reasons_for` | string | Why this company IS a fit — specific evidence | Email hooks, BDR context |
| `clay_icp_reasons_against` | string | Why this company is NOT a fit — specific evidence | BDR context, disqualification |
| `clay_icp_data_gaps` | string | What Ozzy could NOT verify or find | BDR awareness |
| `clay_portfolio_total_sf` | string | Verified total portfolio square footage | ICP re-scoring, email personalization |
| `clay_portfolio_num_buildings` | string | Specific building/property count | Email personalization |
| `clay_portfolio_markets` | string | Verified market/geography list | Customer parallel matching |
| `clay_portfolio_asset_classes` | string | Verified asset classes (Office, Mixed-Use, etc.) | Customer reference matching |
| `clay_estimated_deal_value` | string | Ozzy's independent deal size estimate | ICP output |
| `clay_recommended_action` | string | "Pursue Tier 1", "Pursue Tier 2", "Pursue Tier 3", "Low Priority", "Do Not Pursue" | BDR decision |
| `clay_research_summary` | string | 2–3 sentence narrative summary | Email openers, BDR context |
| `clay_last_enriched` | string | ISO 8601 timestamp of when research was performed | Freshness check |

## Sample Response — "Not a Fit"

```json
{
  "hubspot_id": "226715928309",
  "clay_icp": "Not a Fit",
  "clay_company_tier": "N/A",
  "clay_icp_fit_score": "1",
  "clay_icp_fit_rating": "Not a Fit",
  "clay_icp_reasons_for": "None - not a CRE company",
  "clay_icp_reasons_against": "Executive coaching/wellbeing consultancy, not in commercial real estate, no property portfolio or building management operations",
  "clay_icp_data_gaps": "N/A - company business model clearly identified",
  "clay_portfolio_total_sf": "",
  "clay_portfolio_num_buildings": "",
  "clay_portfolio_markets": "",
  "clay_portfolio_asset_classes": "",
  "clay_estimated_deal_value": "",
  "clay_recommended_action": "Do Not Pursue",
  "clay_research_summary": "ThriveWise is an executive wellbeing and coaching consultancy based in Edinburgh, UK. Led by Dr. Sarah Taylor, they provide leadership development and wellness coaching to organizations. They are not in commercial real estate and have no property portfolio.",
  "clay_last_enriched": "2026-02-11T16:34:10.570Z"
}
```

## Sample Response — "Strong Fit"

```json
{
  "hubspot_id": "12345",
  "clay_icp": "Strong Fit",
  "clay_company_tier": "Tier 1",
  "clay_icp_fit_score": "9",
  "clay_icp_fit_rating": "Strong",
  "clay_icp_reasons_for": "Major institutional owner-operator with 50M+ sf Class A office portfolio, active in tenant experience initiatives, recent $200M amenity renovation program across 12 markets",
  "clay_icp_reasons_against": "Some portfolio overlap with existing HqO customer in Chicago market",
  "clay_icp_data_gaps": "Unable to verify exact SF for recently acquired Atlanta properties",
  "clay_portfolio_total_sf": "52,000,000",
  "clay_portfolio_num_buildings": "147",
  "clay_portfolio_markets": "New York, Boston, Chicago, San Francisco, Los Angeles, Atlanta, Dallas, DC, Seattle, Denver, Miami, Houston",
  "clay_portfolio_asset_classes": "Commercial Office, Mixed-Use",
  "clay_estimated_deal_value": "$2,080,000",
  "clay_recommended_action": "Pursue Tier 1",
  "clay_research_summary": "Major institutional owner-operator managing 147 Class A office and mixed-use properties across 12 US markets totaling ~52M sf. Recently launched a $200M tenant experience renovation program. New SVP of Asset Management hired Q4 2025 with a mandate to improve tenant retention metrics.",
  "clay_last_enriched": "2026-02-11T17:00:00.000Z"
}
```
