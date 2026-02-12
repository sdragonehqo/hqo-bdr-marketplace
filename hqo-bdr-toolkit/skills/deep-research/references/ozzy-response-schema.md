# Ozzy Response Schema

> **Delivery:** Ozzy posts this JSON as a code block in the `#ozzy` Slack channel.
> The agent reads it via the Slack MCP `slack_read_channel` tool.
>
> **HubSpot Sync:** After parsing, the mapped `clay_*` fields (ICP scoring, portfolio basics, summary) are pushed to the matching HubSpot company properties via `manage_crm_objects`. The field names are 1:1 — no mapping needed. Fields not yet mapped in HubSpot (`clay_portfolio_outlook`, `clay_sales_angle`, `clay_recent_news`, `clay_key_champions`, `clay_sources`) are used in-session only.

Ozzy (OpenClaw agent) returns structured JSON from autonomous web research. All fields use the `clay_` prefix — these come from Ozzy's research wrapper, NOT from the Clay MCP connector.

## Response Fields

### ICP & Scoring

| Field | Type | Description | Used In |
|-------|------|-------------|---------|
| `hubspot_id` | string | HubSpot record ID (passed through from request) | Record matching |
| `clay_icp` | string | ICP determination: "Strong Fit", "Moderate Fit", "Weak Fit", "Not a Fit" | ICP scoring (canonical) |
| `clay_company_tier` | string | Tier assignment: "Tier 1", "Tier 2", "Tier 3", "Tier 4", "N/A" | ICP scoring |
| `clay_icp_fit_score` | number | Numeric fit score (1–10) | ICP output |
| `clay_icp_fit_rating` | string | Categorical: "Strong Fit", "Moderate Fit", "Weak Fit", "Not a Fit" | ICP output |
| `clay_icp_reasons_for` | string | Why this company IS a fit — specific evidence | Email hooks, BDR context |
| `clay_icp_reasons_against` | string | Why this company is NOT a fit — specific evidence | BDR context |
| `clay_icp_data_gaps` | string | What Ozzy could NOT verify or find | BDR awareness |
| `clay_estimated_deal_value` | number | Ozzy's independent deal size estimate (in dollars) | ICP output |
| `clay_recommended_action` | string | "Pursue Tier 1", "Pursue Tier 2", "Pursue Tier 3", "Low Priority", "Do Not Pursue" | BDR decision |

### Portfolio

| Field | Type | Description | Used In |
|-------|------|-------------|---------|
| `clay_portfolio_total_sf` | number | Verified total portfolio square footage | ICP scoring, email personalization |
| `clay_portfolio_num_buildings` | number | Specific building/property count | Email personalization |
| `clay_portfolio_markets` | string | Verified market/geography list | Customer parallel matching |
| `clay_portfolio_asset_classes` | string | Verified asset classes (Office, Mixed-Use, etc.) | Customer reference matching |
| `clay_portfolio_outlook` | string | Recent acquisitions, divestitures, capital programs with sources | Company overview, triggers |

### Research & Intel

| Field | Type | Description | Used In |
|-------|------|-------------|---------|
| `clay_research_summary` | string | 2–3 sentence narrative summary | Email openers, BDR context |
| `clay_recent_news` | string | News items from last 12 months with sources and URLs | Triggers, email personalization |
| `clay_sales_angle` | string | Recommended approach — which champions to lead with, triggers to reference, how to frame HqO's value prop | Email generation foundation |
| `clay_last_enriched` | string | ISO 8601 timestamp of when research was performed | Freshness check |

### Key Champions (Array)

| Field | Type | Description | Used In |
|-------|------|-------------|---------|
| `clay_key_champions` | array | High-value contacts identified through web research | Contact mapping, email personalization |

Each entry in `clay_key_champions`:

| Sub-field | Type | Description |
|-----------|------|-------------|
| `name` | string | Contact full name |
| `title` | string | Current title |
| `linkedin` | string | LinkedIn profile URL |
| `insights` | string | What Ozzy found — quotes, conference appearances, articles, role context |
| `source_url` | string | Primary source URL for the intel |

**Note:** `clay_key_champions` is NOT pushed to HubSpot. It is used in-session for contact enrichment and email personalization.

### Sources (Array)

| Field | Type | Description | Used In |
|-------|------|-------------|---------|
| `clay_sources` | array | All sources Ozzy found during research | Source verification, BDR context |

Each entry in `clay_sources`:

| Sub-field | Type | Description |
|-----------|------|-------------|
| `type` | string | Source type: company_website, press_release, news_article, interview, conference_presentation |
| `title` | string | Source title |
| `url` | string | Source URL |
| `date` | string | Publication date (YYYY-MM-DD) |
| `key_info` | string | What was found at this source |

**Note:** `clay_sources` is NOT pushed to HubSpot. It is used in-session for source verification and BDR reference.

## Sample Response — "Not a Fit"

```json
{
  "hubspot_id": "226715928309",
  "clay_icp": "Not a Fit",
  "clay_company_tier": "N/A",
  "clay_icp_fit_score": 1,
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
  "clay_portfolio_outlook": "",
  "clay_sales_angle": "",
  "clay_recent_news": "",
  "clay_key_champions": [],
  "clay_sources": [],
  "clay_last_enriched": "2026-02-11T16:34:10.570Z"
}
```

## Sample Response — "Strong Fit"

```json
{
  "hubspot_id": "61514582769",
  "clay_icp": "Strong Fit",
  "clay_company_tier": "Tier 2",
  "clay_icp_fit_score": 8,
  "clay_icp_fit_rating": "Strong Fit",
  "clay_icp_reasons_for": "Large office-focused portfolio (13.5M SF) in top-tier markets, active acquisition mode with recent 800K SF purchase, COO explicitly prioritizing tenant engagement technology, new VP of Tenant Experience hire signals strategic commitment, $45M capital improvement budget shows investment capacity",
  "clay_icp_reasons_against": "Regional focus (Tri-State area only, not national), some properties in secondary markets, mixed ownership structure with some JV partners who may control operations decisions",
  "clay_icp_data_gaps": "Exact breakdown of SF by asset class, number of tenants across portfolio, current tech stack in use, decision timeline for technology investments",
  "clay_portfolio_total_sf": 13500000,
  "clay_portfolio_num_buildings": 45,
  "clay_portfolio_markets": "New York City, Westchester NY, Stamford CT, Hamilton NJ",
  "clay_portfolio_asset_classes": "office, retail, mixed-use",
  "clay_estimated_deal_value": 540000,
  "clay_recommended_action": "Pursue Tier 2",
  "clay_research_summary": "George Comfort & Sons is a well-established family-owned CRE firm with 100+ years of history managing 13.5M SF across 45 properties in the Tri-State area. Recent activity shows aggressive growth (Capitol Commons acquisition Q4 2023) and strong focus on tenant experience with new dedicated VP hire and $45M capital improvement program. COO explicitly stated commitment to tenant engagement technology in recent press release.",
  "clay_portfolio_outlook": "Active acquisition mode with recent 800K SF Capitol Commons purchase (Q4 2023, source: Bisnow Nov 16, 2023). Divesting secondary markets - sold 1.2M SF NJ portfolio (Q2 2023, source: Commercial Observer June 2023). $45M capital improvement program announced for 10 NYC buildings (Q3 2023, source: company press release). Strategic shift toward quality over quantity with focus on core Tri-State markets and tenant experience.",
  "clay_sales_angle": "Lead with Jennifer Lake (COO) who explicitly stated commitment to tenant engagement platforms in recent press release. Reference Capitol Commons acquisition as proof of growth/scale challenge. Cite Michael Chen's BOMA presentation about fragmented systems - HqO solves this exact pain point. Timing is perfect: new VP of Tenant Experience just hired (Jul 2023) and $45M capital improvement budget shows investment capacity. David Comfort's 'environments where people want to work' quote aligns perfectly with HqO's value prop. Approach: 'As you scale with acquisitions like Capitol Commons and invest $45M in building improvements, HqO consolidates the fragmented amenity systems Michael mentioned at BOMA into one tenant engagement platform.'",
  "clay_recent_news": "Capitol Commons Acquisition (Nov 2023): 800K SF office building in Stamford CT for $165M, expanding CT presence (source: Bisnow). Capital Improvement Program (Sep 2023): $45M investment across 10 NYC buildings for lobby upgrades, amenity spaces, and building technology (source: company press release). Coworking Partnership (Aug 2023): Partnered with Industrious for 50K SF flex space pilot across 3 NYC buildings (source: Commercial Observer). New VP of Tenant Experience hired (Jul 2023): First dedicated role reporting to COO, signals strategic commitment (source: Real Estate Weekly).",
  "clay_key_champions": [
    {
      "name": "Jennifer Lake",
      "title": "Chief Operating Officer",
      "linkedin": "https://linkedin.com/in/jenniferlake",
      "insights": "Quoted in Capitol Commons acquisition press release: 'We're investing heavily in building technology and tenant engagement platforms. Our tenants expect the same digital experience at work as they have in their personal lives.' Also speaking at NYC CRE Summit 2024 on 'Future of Office Experience' panel.",
      "source_url": "https://gcomfort.com/press/capitol-commons-acquisition-2023"
    },
    {
      "name": "Michael Chen",
      "title": "VP of Property Management",
      "linkedin": "https://linkedin.com/in/michaelchen-cre",
      "insights": "Presented at BOMA 2023 conference discussing operational challenges. Quoted: 'Managing amenity bookings across multiple legacy systems has been a persistent challenge. We're looking for integrated solutions that work across our entire portfolio.' 15-year tenure signals deep operational knowledge.",
      "source_url": "https://boma.org/conference/2023/sessions/property-management-challenges"
    },
    {
      "name": "David Comfort",
      "title": "Chairman & CEO",
      "linkedin": "https://linkedin.com/in/davidcomfort",
      "insights": "Third-generation family leadership. Quoted in Bisnow interview Oct 2023: 'We're focused on creating environments where people want to work, not just where they have to work. Amenities and experience are no longer nice-to-haves - they're essential for tenant retention.' Signals top-down commitment to experience focus.",
      "source_url": "https://bisnow.com/new-york/news/interviews/george-comfort-office-future-2023"
    }
  ],
  "clay_sources": [
    {
      "type": "company_website",
      "title": "About George Comfort & Sons",
      "url": "https://gcomfort.com/about",
      "date": "2024-02-12",
      "key_info": "Portfolio size 13.5M SF, 45 buildings, company history and markets"
    },
    {
      "type": "press_release",
      "title": "George Comfort & Sons Acquires Capitol Commons",
      "url": "https://gcomfort.com/press/capitol-commons-acquisition-2023",
      "date": "2023-11-15",
      "key_info": "800K SF acquisition, COO quote about tenant engagement technology investment"
    },
    {
      "type": "news_article",
      "title": "George Comfort Expands CT Footprint with Capitol Commons",
      "url": "https://bisnow.com/westchester/news/capitol-commons-120445",
      "date": "2023-11-16",
      "key_info": "Acquisition details, $165M transaction, strategic expansion analysis"
    },
    {
      "type": "conference_presentation",
      "title": "Property Management Challenges in Modern CRE",
      "url": "https://boma.org/conference/2023/sessions/property-management-challenges",
      "date": "2023-09-28",
      "key_info": "Michael Chen presentation about fragmented systems and integration needs"
    }
  ],
  "clay_last_enriched": "2026-02-12T18:34:00.000Z"
}
```
