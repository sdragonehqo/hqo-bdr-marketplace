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

### Step 1: Check for Existing Research (Cache Check)

Before triggering a new Ozzy run, check if research already exists for this company in the `#ozzy` channel.

Search the channel using the Slack MCP `slack_search_public` tool:

- **query:** The `hubspot_id` value (e.g., `61514582769`) with `in:<#C0AEK5BFHSA>`

**Important:** Search for the `hubspot_id`, NOT the company name or domain. The `hubspot_id` is the only stable identifier — company names can vary and domains can change, but the HubSpot record ID is immutable.

Look through the search results for a message from the Ozzy bot (user ID: `deep_research.ozzy_bot_user_id`) that contains a JSON code block with `"hubspot_id": "{your_id}"`.

**If a matching response is found:**
- Parse the JSON from the existing message
- **Tell the BDR:** "Found existing deep research for [company] — using cached results from [clay_last_enriched date]."
- Skip directly to **Step 5: Push to HubSpot** (the data may not have been pushed previously, or may need refreshing)
- Then continue to **Handling the Response**

**If no matching response is found:**
- Continue to Step 2 to trigger a fresh Ozzy run

### Step 2: Send Request

Send a message to the `#ozzy` Slack channel using the Slack MCP `slack_send_message` tool:

- **channel_id:** Use `deep_research.slack_channel_id` from settings
- **message:** Format as: `<@U0AFG7T5ALQ> ICP:{hubspot_id},{domain},{company_name}`

The `<@U0AFG7T5ALQ>` is the Slack user mention tag for the Ozzy bot — this is what actually pings the bot and triggers research. Plain text `@Ozzy` does NOT work. Always use the `<@...>` format.

Example:
```
<@U0AFG7T5ALQ> ICP:61514582769,gcomfort.com,George Comfort & Sons
```

**Tell the BDR:** "Running deep research on [company] — Ozzy is crawling the web for detailed portfolio and people intel. This takes about a minute..."

### Step 3: Wait for Response

Wait `deep_research.wait_seconds` (default: 90 seconds) for Ozzy to process.

### Step 4: Read Response

Read the `#ozzy` channel using the Slack MCP `slack_read_channel` tool:

- **channel_id:** Same channel ID from settings
- **limit:** 10 (to capture Ozzy's full response — Ozzy may post multiple messages)

Look for messages from the Ozzy bot (user ID: `deep_research.ozzy_bot_user_id`). The structured JSON response will be in a code block (``` delimited) containing all `clay_*` fields matching the schema in `references/ozzy-response-schema.md`.

### Parsing the Response

1. Find the message from Ozzy that contains a JSON code block with `hubspot_id` matching your request
2. Parse the JSON from inside the code block
3. The response will match the schema in `references/ozzy-response-schema.md`

If Ozzy posts multiple messages (narrative + JSON), use the one with the structured JSON code block.

### Step 5: Push to HubSpot

After successfully parsing Ozzy's JSON response, immediately update the company record in HubSpot with all `clay_*` fields. This ensures deep research data is persisted in the CRM — not just used in this session.

Use the HubSpot MCP `manage_crm_objects` tool to update the company:

- **objectType:** `companies`
- **objectId:** The HubSpot company record ID (the `hubspot_id` from Ozzy's response)
- **properties:** All `clay_*` fields from Ozzy's JSON response, mapped 1:1 as property name → value

The HubSpot company object has matching properties for every `clay_*` field in the response schema. Push them all:

| Ozzy Response Field | HubSpot Company Property |
|---------------------|--------------------------|
| `clay_icp` | `clay_icp` |
| `clay_company_tier` | `clay_company_tier` |
| `clay_icp_fit_score` | `clay_icp_fit_score` |
| `clay_icp_fit_rating` | `clay_icp_fit_rating` |
| `clay_icp_reasons_for` | `clay_icp_reasons_for` |
| `clay_icp_reasons_against` | `clay_icp_reasons_against` |
| `clay_icp_data_gaps` | `clay_icp_data_gaps` |
| `clay_portfolio_total_sf` | `clay_portfolio_total_sf` |
| `clay_portfolio_num_buildings` | `clay_portfolio_num_buildings` |
| `clay_portfolio_markets` | `clay_portfolio_markets` |
| `clay_portfolio_asset_classes` | `clay_portfolio_asset_classes` |
| `clay_estimated_deal_value` | `clay_estimated_deal_value` |
| `clay_recommended_action` | `clay_recommended_action` |
| `clay_research_summary` | `clay_research_summary` |
| `clay_last_enriched` | `clay_last_enriched` |

**Rules:**
- Only push fields that have non-empty values. Skip any field where the value is `""` or `null`.
- Do NOT push `hubspot_id` — that's the record identifier, not a property to update.
- This push happens automatically — do NOT ask the BDR for confirmation. The data is enrichment, not a destructive change.
- If the HubSpot update fails, log the error and continue the workflow. The deep research data is still available in-session even if the CRM push fails.

**Tell the BDR:** "Deep research complete — pushed Ozzy's findings to the HubSpot record so they're saved for the team."

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
| Cache check finds existing research (Step 1) | Use cached results. Skip Ozzy request + wait. Still push to HubSpot (Step 5) and continue to Handling the Response. |
| Cache check search fails | Treat as cache miss — proceed to Step 2 and trigger a fresh Ozzy run. |
| Slack send fails or Ozzy bot is offline | Log "Deep research unavailable — continuing with Clay+HubSpot data." Continue workflow. |
| No response from Ozzy in `#ozzy` after `max_wait_seconds` | Read channel one more time. If still no response, continue workflow without deep research. |
| Response missing fields | Use whatever fields are present. Missing fields = use Clay/HubSpot data for those. |
| `clay_icp` = "Not a Fit" | **Hard stop.** Present conflict. Require BDR confirmation. See ICP Validation above. |
| Multiple responses for same `hubspot_id` | Use the most recent matching message (closest to current time). |
| HubSpot update fails after successful Ozzy response | Log the error. Deep research data is still available in-session — continue the workflow. Note in output: "Could not sync deep research to HubSpot." |

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
