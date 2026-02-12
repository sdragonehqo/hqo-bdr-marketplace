---
name: onboard
description: Guided first-time onboarding for new BDRs. Walks through connector setup, Gmail signature, plugin configuration, and a test run — all in one conversation.
arguments: []
---

# /hqo:onboard

Full guided onboarding for a new BDR. This is the FIRST command a new user should run. It replaces the need to read any documentation — everything happens interactively in this conversation.

## Tone

Be friendly, clear, and encouraging. This person may have never used Cowork before. Explain what each step does and WHY before asking them to do it. Use simple language — no jargon. Celebrate small wins ("Great, HubSpot is connected!").

## Flow

### Phase 1: Welcome

Start with:

> **Welcome to the HqO BDR Outreach Plugin!**
>
> I'm going to walk you through getting everything set up. This takes about 10 minutes and you only need to do it once.
>
> Here's what we'll cover:
> 1. Check your connectors (HubSpot, Clay, Gmail, Slack)
> 2. Make sure your Gmail signature is ready
> 3. Save your BDR profile
> 4. Run a quick test
>
> Ready? Let's start with your connectors.

### Phase 2: Connector Check

Check each required connector by attempting a lightweight call. Explain what each one does before testing.

**HubSpot:**
> HubSpot is your CRM — the plugin uses it to look up companies, find contacts, and check engagement history.
>
> Let me check if it's connected...

- Try a simple HubSpot call (e.g., `get_user_details` or search for a known company)
- If it works: "✓ HubSpot is connected!"
- If it fails: "HubSpot isn't connected yet. Here's how to fix it:"
  - "Go to **Settings** (gear icon) → **Connectors** → find **HubSpot** → click **Connect**"
  - "Sign in with your HqO HubSpot credentials and grant the permissions it asks for"
  - "Come back here and tell me when you're done — I'll check again"
  - Wait for them to confirm, then re-test

**Clay:**
> Clay enriches companies with data we don't have in HubSpot — portfolio size, recent news, additional contacts, and market intel.

- Try `get-credits-available` or similar lightweight Clay call
- If it works: "✓ Clay is connected! You have credits available."
- If it fails: Same guided flow as HubSpot — walk them through Settings → Connectors → Clay → Connect

**Gmail (BOTH connectors):**
> There are two Gmail connectors, and you need both:
> - **Google Gmail** lets the plugin read and search your inbox
> - **Gmail** (the second one) lets it create drafts in your inbox
>
> Let me check both...

- Test the read connector (e.g., `read_gmail_profile`)
- Test the draft connector (e.g., `get_profile` from the gmail tools or `list_labels`)
- If both work: "✓ Both Gmail connectors are connected!"
- If only one works: Explain which one is missing and why they need it
  - "The draft connector isn't connected. Without it, I can research accounts and generate emails, but I won't be able to put them in your Gmail drafts. Let me walk you through connecting it..."
- If neither works: Walk through connecting both, one at a time

**Slack:**
> Slack is how the plugin talks to **Ozzy**, our deep research bot. When you run outreach on a company, the plugin sends a message to the `#ozzy` Slack channel that mentions `<@U0AFG7T5ALQ>` (the Ozzy bot) to trigger it. Ozzy crawls the web — company sites, press releases, LinkedIn, conference lists — and posts back detailed portfolio and people intel. The plugin reads those results right from Slack.
>
> Let me check if Slack is connected...

- Try a lightweight Slack call (e.g., `slack_search_channels` with query "ozzy")
- If it works: "✓ Slack is connected! I can see the #ozzy channel."
- If it fails: Same guided flow — walk them through Settings → Connectors → Slack → Connect
  - "Slack isn't connected yet. Go to **Settings** (gear icon) → **Connectors** → find **Slack** → click **Connect**"
  - "Sign in with your HqO Slack workspace credentials and grant the permissions it asks for"
  - "Come back here and tell me when you're done — I'll check again"
  - Wait for them to confirm, then re-test

After verifying Slack is connected, confirm access to the `#ozzy` channel:
- Try `slack_read_channel` with channel_id `C0AEK5BFHSA` and limit `1`
- If it works: "✓ I can read the #ozzy channel — Ozzy is ready to go."
- If it fails: "I can't access the #ozzy channel. You may need to join it first — search for `#ozzy` in Slack and click Join. Let me know when you're in."

After all connectors are checked:
> **All connectors are set!** That's the hardest part done. Now let's make sure your emails will look professional.

### Phase 3: Email Signature Check

First, check `config/settings.json` → `bdr.signature_html`.

**If `signature_html` is already populated (non-empty):**

Skip this phase entirely. Just note:

> ✓ **Signature already configured** — your drafts will include your email signature. (Run `/hqo:setup` anytime to update it.)

Move on to Phase 4.

**If `signature_html` is empty or missing:**

> When I create email drafts for you, I need to include your signature directly in the draft body. The Gmail connector can't pull your signature automatically, so I need the HTML for your HqO-branded signature.
>
> This way, every draft I put in your inbox is ready to send — signature and all.
>
> The easiest way to get this is from your Gmail settings. Here's how:
>
> 1. Open **Gmail** in your browser (mail.google.com)
> 2. Click the **gear icon** (top right) → **See all settings**
> 3. Scroll down to the **Signature** section
> 4. Click inside the signature editor box where your signature is displayed
> 5. **Select all** the signature content (Cmd+A / Ctrl+A)
> 6. **Copy** it (Cmd+C / Ctrl+C)
> 7. Now open a new **Google Doc** (docs.google.com) or any text editor
> 8. **Paste** the signature there (Cmd+V / Ctrl+V) — this preserves the formatting
> 9. In that Google Doc, go to **File → Download → Web Page (.html, zipped)**
> 10. Open the downloaded `.zip`, find the `.html` file inside
> 11. **Right-click** the `.html` file → **Open with** a text editor (TextEdit on Mac, Notepad on Windows) — NOT a browser
> 12. **Select all** (Cmd+A / Ctrl+A), **copy** (Cmd+C / Ctrl+C), and **paste it here**
>
> I know that's a few steps — but you only need to do this once.

If the BDR says they have an `.html` file already (from their manager or GTM Engineering), take the shortcut:

> Perfect — just **right-click** the `.html` file → **Open with** a text editor (NOT a browser), **select all**, **copy**, and **paste it here**.

If the BDR has no signature at all:

> No problem — let's create one. Tell me your full name, title, phone number, and HqO email, and I'll generate one for you.

When they provide the details, generate a simple, professional HTML signature block:

```html
<table cellpadding="0" cellspacing="0" style="font-family: Arial, sans-serif; font-size: 13px; color: #333333;">
  <tr><td style="font-weight: bold; font-size: 14px;">[Full Name]</td></tr>
  <tr><td style="color: #666666;">[Title] | HqO</td></tr>
  <tr><td>[Phone] | <a href="mailto:[Email]" style="color: #1a73e8; text-decoration: none;">[Email]</a></td></tr>
  <tr><td><a href="https://hqo.co" style="color: #1a73e8; text-decoration: none;">hqo.co</a></td></tr>
</table>
```

Show the generated HTML to the BDR and ask if it looks right. Offer to adjust.

**After receiving the signature HTML:**

1. Save the raw HTML string to `config/settings.json` under `bdr.signature_html`
2. Confirm to the BDR:

> ✓ **Signature saved!** Here's a preview of what it'll look like in your drafts:
>
> [render a plain-text representation of the signature for them to verify]
>
> Every email draft I create will include this signature at the bottom. If you ever need to update it, run `/hqo:setup`.

Wait for confirmation before moving on.

### Phase 4: BDR Profile Setup

> Now I need to know a few things about you so I can personalize your outreach emails.

Ask these one at a time, waiting for each answer:

1. "What's your first name?"
2. "What's your HqO email address?"
3. Ask tone preference using a multiple-choice question:
   - **Professional** (recommended) — polished and direct
   - **Casual** — conversational and warm
   - **Consultative** — advisory and insight-led

Save answers to `config/settings.json` (the `signature_html` field was already saved in Phase 3):
```json
{
  "bdr": {
    "name": "[their name]",
    "email": "[their email]",
    "tone": "[their choice]",
    "signature_html": "[already saved from Phase 3]"
  }
}
```

Confirm:
> Got it — you're saved as **[Name]** ([email]), tone set to **[tone]**.

### Phase 5: Test Run

> Let's do a quick test to make sure everything works. I'll run a research check on a company you're already familiar with.
>
> Give me the name of a company you've been prospecting (or I can use "Brookfield Properties" as a test).

Wait for their answer, then run `/hqo:research` on the company they choose (or Brookfield as default).

After results come in, walk them through the output:

> Here's what the plugin found. Let me explain each section and the tools behind them:
>
> **ICP Score** — Rates how well the company fits our ideal customer profile, based on portfolio size and deal potential. This comes from **Clay** (company enrichment) and **HubSpot** (CRM data).
>
> **Deep Research Insights** — If deep research is enabled, the plugin sends a message to the **#ozzy Slack channel** mentioning `<@U0AFG7T5ALQ>` (the Ozzy bot) to trigger autonomous web research. Ozzy crawls company websites, press releases, SEC filings, LinkedIn profiles, and conference speaker lists. After about a minute, Ozzy posts back structured results — portfolio verification, ICP validation, and people intel — which the plugin reads directly from Slack. This goes far beyond what Clay or HubSpot provide.
>
> **Company Overview** — Background pulled from **Clay** and **HubSpot**, enriched with Ozzy's findings when available.
>
> **Key Contacts** — People at the company mapped by priority using **Clay** (contact discovery) and **HubSpot** (existing records). Asset Managers and executives come first because they're our primary buyers. When Ozzy runs, it also surfaces LinkedIn activity, press mentions, and conference appearances for top contacts — these become personalized email openers. If Clay finds contacts that aren't in HubSpot yet, the plugin will ask if you want to add them to the CRM — that way the whole team can see them.
>
> **Customer Parallels** — Existing HqO customers in the same market, pulled from **HubSpot**, that we can reference in outreach.
>
> **Engagement Status** — Shows if anyone on the team has already reached out to these contacts, from **HubSpot** engagement history.

Then:
> That's the research side. When you're ready to actually generate outreach emails, you'd run `/hqo:outreach [company]` — same thing but with personalized emails at the end. Then `/hqo:push-drafts` to put them in your Gmail.

### Phase 6: Wrap-up

> **You're all set!** Here's your cheat sheet:
>
> **Commands:**
>
> | Command | What It Does |
> |---------|-------------|
> | `/hqo:outreach [company]` | Full workflow — research + emails |
> | `/hqo:research [company]` | Research only, no emails |
> | `/hqo:push-drafts` | Create Gmail drafts from generated emails |
> | `/hqo:setup` | Update your name, email, or tone preference |
>
> **What's working behind the scenes:**
>
> | Tool | What It Does |
> |------|-------------|
> | **HubSpot** | CRM lookups — company records, contacts, engagement history |
> | **Clay** | Company enrichment — portfolio size, markets, news, additional contacts |
> | **Ozzy (via Slack)** | Deep web research — mentions `<@U0AFG7T5ALQ>` in `#ozzy` channel to trigger the bot, gets back verified portfolio data, ICP validation, and people intel (LinkedIn, press, conferences) |
> | **Gmail** | Creates draft emails directly in your inbox for review before sending |
>
> Start with `/hqo:outreach [company]` whenever you're ready to prospect a new account. The plugin handles the rest — pulling data from HubSpot and Clay, calling Ozzy for deep research via Slack, then generating personalized emails and putting them in your Gmail drafts.
>
> Any questions?

## Error Recovery

- If a connector check fails mid-flow, don't restart from the beginning. Pick up where you left off.
- If the test run fails (e.g., Clay credits exhausted), explain what happened and assure them the setup is still complete — they just need to resolve the specific issue.
- If they seem confused at any point, offer to explain further. Never rush past a step.

## Important Rules

- NEVER skip the connector checks (HubSpot, Clay, Gmail, Slack). A missing connector will break the workflow later and the BDR won't know why.
- NEVER assume connectors are working — always test them with an actual call.
- ALWAYS wait for the BDR's response before moving to the next phase. This is a conversation, not a monologue.
- Keep each message focused on ONE thing. Don't dump a wall of text.
