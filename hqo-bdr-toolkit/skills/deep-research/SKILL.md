---
name: hqo-deep-research
description: ICP research and company intelligence via Ozzy (OpenClaw agent). Ozzy is Claude's research tool — it handles ICP scoring, portfolio verification, people intel, and sales angle development so Claude doesn't duplicate that work. Runs automatically on every /hqo:research command.
---

# Deep Research (Ozzy)

## Output Rules

**Do NOT print internal reasoning, `<thinking>` tags, tool-call rationale, or decision-tree logic in the chat.** Only show the BDR the structured Research Results output. All internal processing (Slack calls, JSON parsing, field mapping) stays behind the scenes.

## Purpose

Ozzy is Claude's research tool. Instead of Claude doing its own ICP scoring and company analysis, Ozzy handles it — crawling the web (company websites, press releases, SEC filings, LinkedIn profiles, conference speaker lists, industry publications) to build a comprehensive company and people profile with ICP scoring, portfolio verification, key champions, and a recommended sales angle.

**Ozzy runs automatically on every `/hqo:research` command.** Claude uses Ozzy's output directly for ICP scoring, contact prioritization, and email personalization. There is no separate "Claude ICP scoring" step — Ozzy IS the ICP scoring step.

## When It Runs

Ozzy runs as **Step 2** of the `/hqo:research` workflow, immediately after Company Identification. It is NOT optional — it runs every time. The BDR does not need to request it.

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
- **Tell the BDR:** "Found existing research for [company] from [clay_last_enriched date]."
- Skip directly to **Step 5: Push to HubSpot** (the data may not have been pushed previously, or may need refreshing)
- Then continue to **Using the Results**

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

**Tell the BDR:** "Running research on [company] — Ozzy is gathering portfolio data, ICP scoring, key champions, and sales angles. This takes about a minute..."

### Step 3: Wait for Response

Wait `deep_research.wait_seconds` (default: 90 seconds) for Ozzy to process.

### Step 4: Read Response

Read the `#ozzy` channel using the Slack MCP `slack_read_channel` tool:

- **channel_id:** Same channel ID from settings
- **limit:** 10 (to capture Ozzy's full response — Ozzy may post multiple messages)

Look for messages from the Ozzy bot (user ID: `deep_research.ozzy_bot_user_id`). The structured JSON response will be in a code block (``` delimited) containing all fields matching the schema in `references/ozzy-response-schema.md`.

### Parsing the Response

1. Find the message from Ozzy that contains a JSON code block with `hubspot_id` matching your request
2. Parse the JSON from inside the code block
3. The response will match the schema in `references/ozzy-response-schema.md`

If Ozzy posts multiple messages (narrative + JSON), use the one with the structured JSON code block.

### Step 5: Review & Push to HubSpot

After successfully parsing Ozzy's JSON response, present the results to the BDR for review before pushing to HubSpot.

**Present the results first.** Show the BDR a summary of what Ozzy found — use the output format from the Output Section below. Let them review the ICP scoring, portfolio data, key champions, and recommended sales angle.

**Then ask for approval to push:**

> "Ready to push these findings to the HubSpot record for [company]? This updates the research fields on the company record so the team can see them."

**Only push after the BDR confirms.** Use the HubSpot MCP `manage_crm_objects` tool:

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

**NOT pushed to HubSpot (in-session only):** `clay_portfolio_outlook`, `clay_sales_angle`, `clay_recent_news`, `clay_key_champions`, `clay_sources`. These fields are not yet mapped in HubSpot — they are used in-session for contact mapping, email personalization, and BDR context only.

**Rules:**
- Only push fields that have non-empty values. Skip any field where the value is `""` or `null`.
- Do NOT push `hubspot_id` — that's the record identifier, not a property to update.
- **Always wait for BDR approval** before pushing. Never auto-push.
- If the BDR declines the push, continue the workflow — the data is still available in-session.
- If the HubSpot update fails, log the error and continue the workflow.

**After successful push, tell the BDR:** "Done — research findings are now on the HubSpot record for the team."

## Using the Results

Ozzy's output is the **canonical ICP assessment and company intelligence** for the workflow. Claude does not do its own ICP scoring — it uses Ozzy's directly.

### ICP Scoring

Ozzy's response IS the ICP scoring step. Use these fields directly:

- `clay_icp` — The ICP determination (Strong Fit, Moderate Fit, Weak Fit, Not a Fit)
- `clay_company_tier` — Tier assignment (Tier 1–4, or N/A)
- `clay_icp_fit_score` — Numeric score (1–10)
- `clay_estimated_deal_value` — Deal size estimate
- `clay_recommended_action` — Go/no-go recommendation
- `clay_icp_reasons_for` / `clay_icp_reasons_against` — Evidence for the BDR

Present these as the ICP scoring results. There is no separate Claude ICP calculation.

### Portfolio Data

Ozzy provides verified portfolio intel:

- `clay_portfolio_total_sf` — Verified square footage
- `clay_portfolio_num_buildings` — Property count
- `clay_portfolio_markets` — Market list (use for customer parallel matching)
- `clay_portfolio_asset_classes` — Asset classes (use for customer reference matching)
- `clay_portfolio_outlook` — Recent acquisitions, divestitures, capital programs
- `clay_recent_news` — News and triggers from the last 12 months

### Key Champions

`clay_key_champions` is an array of high-value contacts Ozzy identified through web research. Each entry includes:

- `name` — Contact name
- `title` — Current title
- `linkedin` — LinkedIn profile URL
- `insights` — What Ozzy found (quotes, conference appearances, articles, role context)
- `source_url` — Source for the intel

**Use key champions for:**
- Enriching the Contact Mapping step — add Ozzy's intel to matching contacts
- Email personalization — quotes, conference mentions, and article references make far better openers than generic portfolio references
- Prioritization — contacts Ozzy surfaced with strong insights should be prioritized

### Sales Angle

`clay_sales_angle` is Ozzy's recommended approach for the account. Use it as the foundation for email generation — it identifies which champions to lead with, which triggers to reference, and how to frame HqO's value prop for this specific company.

### Sources

`clay_sources` is an array of everything Ozzy found during research. Each entry includes type, title, URL, date, and key info. Use these for:
- Verifying claims before including in emails
- Providing the BDR context on where intel came from
- Reference material if the BDR wants to dig deeper

## Error Handling

| Scenario | Action |
|----------|--------|
| Cache check finds existing research (Step 1) | Use cached results. Skip Ozzy request + wait. Still push to HubSpot (Step 5) and continue. |
| Cache check search fails | Treat as cache miss — proceed to Step 2 and trigger a fresh Ozzy run. |
| Slack send fails or Ozzy bot is offline | Log "Research tool unavailable — falling back to Clay + HubSpot data." Continue workflow with Clay data only. Claude performs basic ICP scoring using `portfolio_sf × $0.04/sf` from Clay enrichment. |
| No response from Ozzy after `max_wait_seconds` | Read channel one more time. If still no response, fall back to Clay + HubSpot. |
| Response missing fields | Use whatever fields are present. Missing fields = use Clay/HubSpot data for those. |
| Multiple responses for same `hubspot_id` | Use the most recent matching message (closest to current time). |
| HubSpot update fails | Log the error. Research data is still available in-session — continue the workflow. |

## Graceful Degradation

If Ozzy is unavailable (offline, timeout, Slack error):
- Claude falls back to Clay + HubSpot data for ICP scoring
- Uses the formula `portfolio_sf × $0.04/sf` from Clay enrichment for deal sizing
- Assigns tier per `references/icp-criteria.md`
- Workflow continues without key champions, sales angle, or verified portfolio data
- Tell the BDR: "Ozzy is unavailable — using Clay + HubSpot data for this research."

## Output Section

When Ozzy's research completes successfully, present this to the BDR:

### Research Results

- **ICP Rating:** [clay_icp] — [clay_company_tier]
- **Confidence Score:** [clay_icp_fit_score]/10
- **Estimated Deal Value:** [clay_estimated_deal_value]
- **Recommended Action:** [clay_recommended_action]
- **Reasons For:** [clay_icp_reasons_for]
- **Reasons Against:** [clay_icp_reasons_against]
- **Data Gaps:** [clay_icp_data_gaps]
- **Portfolio:** [clay_portfolio_num_buildings] properties, [clay_portfolio_total_sf] sf across [clay_portfolio_markets]
- **Asset Classes:** [clay_portfolio_asset_classes]
- **Portfolio Outlook:** [clay_portfolio_outlook]
- **Recent News:** [clay_recent_news]
- **Key Champions:** [name, title, and key insight for each entry in clay_key_champions]
- **Sales Angle:** [clay_sales_angle]
- **Summary:** [clay_research_summary]
- **Research Date:** [clay_last_enriched]
