---
name: hqo-deep-research
description: Deep autonomous web research on target companies and contacts via Ozzy (OpenClaw agent). Runs after initial ICP scoring to validate fit, enrich portfolio details, and surface people intel for Tier 1+2 contacts. Auto-triggers as step 2b in the prospecting workflow.
---

# Deep Research (Ozzy)

## Purpose

Ozzy is an autonomous research agent that crawls the web — company websites, press releases, SEC filings, LinkedIn profiles, conference speaker lists, industry publications — to build a deep company and people profile. This goes far beyond what Clay or HubSpot provide.

Deep research runs **after** Step 2 (ICP Scoring) and **before** Step 3 (Contact Mapping). It validates the initial ICP determination and enriches everything downstream.

## When to Use

- After Steps 1–2 of the prospecting workflow have completed (need company_name, domain, hubspot_id)
- When `config/settings.json` has `deep_research.enabled = true`
- If the wrapper is unreachable, skip gracefully and continue with Clay+HubSpot data

## Calling Ozzy

Read the Slack channel config from `config/settings.json` under `deep_research`.

### Step 1: Send Request

Send a message to the `#ozzy` Slack channel using the Slack MCP `slack_send_message` tool:

- **channel_id:** Use `deep_research.slack_channel_id` from settings
- **message:** Format as: `@Ozzy ICP:{hubspot_id},{domain},{company_name}`

Example:
```
@Ozzy ICP:61514582769,gcomfort.com,George Comfort & Sons
```

**Tell the BDR:** "Running deep research on [company] — Ozzy is crawling the web for detailed portfolio and people intel. This takes about a minute..."

### Step 2: Wait for Response

Wait `deep_research.wait_seconds` (default: 90 seconds) for Ozzy to process.

### Step 3: Read Response

Read the `#ozzy` channel using the Slack MCP `slack_read_channel` tool:

- **channel_id:** Same channel ID from settings
- **limit:** 10 (to capture Ozzy's full response — Ozzy may post multiple messages)

Look for messages from the Ozzy bot (user ID: `deep_research.ozzy_bot_user_id`). The structured JSON response will be in a code block (``` delimited) containing all `clay_*` fields matching the schema in `references/ozzy-response-schema.md`.

### Parsing the Response

1. Find the message from Ozzy that contains a JSON code block with `hubspot_id` matching your request
2. Parse the JSON from inside the code block
3. The response will match the schema in `references/ozzy-response-schema.md`

If Ozzy posts multiple messages (narrative + JSON), use the one with the structured JSON code block.

## Handling the Response

### ICP Validation (Critical)

Compare Ozzy's `clay_icp` with the initial ICP determination from Step 2:

**If Ozzy confirms the fit** (both agree the company is a fit):
- Merge Ozzy's verified portfolio data into the company profile
- Add `clay_icp_fit_score` and `clay_icp_fit_rating` to output
- Continue to Step 3

**If Ozzy says "Not a Fit" but Step 2 said it WAS a fit:**
- **STOP the workflow immediately**
- Present BOTH assessments to the BDR:
  - "Initial research (Clay+HubSpot) suggested [Tier X] fit because [reasons]"
  - "Deep research (Ozzy) says NOT a fit because: [clay_icp_reasons_against]"
  - "Recommended action: [clay_recommended_action]"
  - "Data gaps: [clay_icp_data_gaps]"
- Ask: "Do you want to continue with outreach anyway, or skip this company?"
- Only proceed if BDR explicitly confirms

**If Ozzy says fit but with a different tier than Step 2:**
- Present both tiers with reasoning
- Use the more conservative (lower) tier unless BDR overrides

### Portfolio Enrichment

When Ozzy returns portfolio data, it takes priority over Clay's estimates:

- `clay_portfolio_total_sf` — Use for deal size recalculation if significantly different from Clay's
- `clay_portfolio_num_buildings` — Use in email personalization ("across your 147 properties")
- `clay_portfolio_markets` — Use for customer parallel matching (more accurate than Clay)
- `clay_portfolio_asset_classes` — Use for customer reference matching (verified > inferred)

### People Intel

Ozzy researches **Tier 1 + Tier 2 contacts only**:
- **Tier 1:** Asset Manager, SVP/Head of Portfolio, MD, Executives
- **Tier 2:** Property Manager, Regional Director, Experience Manager

For each contact, Ozzy may surface:
- LinkedIn activity (recent posts, profile changes)
- Press mentions (quotes, interviews)
- Conference appearances (speaking, panels)
- Published articles (authored pieces, thought leadership)
- Role tenure and transitions

This intel feeds directly into email personalization in Step 5. See `references/research-to-workflow-mapping.md` for the full mapping.

## Error Handling

| Scenario | Action |
|----------|--------|
| Slack send fails or Ozzy bot is offline | Log "Deep research unavailable — continuing with Clay+HubSpot data." Continue workflow. |
| No response from Ozzy in `#ozzy` after `max_wait_seconds` | Read channel one more time. If still no response, continue workflow without deep research. |
| Response missing fields | Use whatever fields are present. Missing fields = use Clay/HubSpot data for those. |
| `clay_icp` = "Not a Fit" | **Hard stop.** Present conflict. Require BDR confirmation. See ICP Validation above. |
| Multiple responses in channel (from previous runs) | Match on `hubspot_id` inside the JSON to find the correct response. Use the most recent matching message. |

## Graceful Degradation

Deep research is **additive**. If it fails for any reason:
- Steps 3–6 proceed exactly as they would without deep research
- Output sections still render, just without the "Deep Research Insights" subsection
- Email personalization falls back to Clay+HubSpot data (existing behavior)
- Note in output: "Deep research was unavailable for this run"

## Output Section

When deep research completes successfully, add this section to the output **after ICP Match Rating**:

### Deep Research Insights

- **Confidence Score:** [clay_icp_fit_score]/10 — [clay_icp_fit_rating]
- **Reasons For:** [clay_icp_reasons_for]
- **Reasons Against:** [clay_icp_reasons_against]
- **Data Gaps:** [clay_icp_data_gaps]
- **Verified Portfolio:** [clay_portfolio_num_buildings] properties, [clay_portfolio_total_sf] sf across [clay_portfolio_markets]
- **Asset Classes:** [clay_portfolio_asset_classes]
- **Estimated Deal Value:** [clay_estimated_deal_value]
- **Recommended Action:** [clay_recommended_action]
- **Summary:** [clay_research_summary]
- **People Intel:** [per-contact findings for Tier 1+2, if available]
- **Research Date:** [clay_last_enriched]
