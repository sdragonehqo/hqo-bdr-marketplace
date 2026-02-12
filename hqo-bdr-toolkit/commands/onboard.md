---
name: onboard
description: Guided onboarding and settings management for BDRs. First-time setup walks through connectors, Gmail signature, and profile config. Returning BDRs can update any setting.
arguments: []
---

# /hqo:onboard

Guided onboarding for BDRs. For first-time users, this walks through full setup. For returning users, this is also how they update settings (name, email, tone, signature).

## Tone

Be friendly, clear, and encouraging. This person may have never used Cowork before. Explain what each step does and WHY before asking them to do it. Use simple language — no jargon. Celebrate small wins ("Great, HubSpot is connected!").

## Detecting First-Time vs. Returning

Read `config/settings.json` at the start.

**If `bdr.name` is empty or not set:** This is a first-time user. Run the full flow (Phases 1–6).

**If `bdr.name` is already populated:** This is a returning user. Show current settings and ask what they want to update:

> Here are your current settings:
>
> - **Name:** [bdr.name]
> - **Email:** [bdr.email]
> - **Tone:** [bdr.tone]
> - **Signature:** [show "Configured" if `bdr.signature_html` is non-empty, or "Not configured" if empty]
>
> What would you like to update?

Let them pick one or more: Name, Email, Tone, Signature. Walk through each selected update:

- **Name/Email/Tone:** Ask for the new value, confirm, save.
- **Signature:** Follow the same auto-extraction as Phase 3 below (send yourself an email, plugin extracts it). If extraction fails, fall back to manual generation.

Save all updates to `config/settings.json`, confirm the new values, and remind them: "Run `/hqo:research [company]` to start prospecting."

If the BDR's signature is empty and they're not updating it, gently remind them: "Heads up — you don't have a signature configured yet. Drafts will be created without one. Want to add it now?"

---

## Full First-Time Flow

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
- If it works: "HubSpot is connected!"
- If it fails: "HubSpot isn't connected yet. Here's how to fix it:"
  - "Go to **Settings** (gear icon) → **Connectors** → find **HubSpot** → click **Connect**"
  - "Sign in with your HqO HubSpot credentials and grant the permissions it asks for"
  - "Come back here and tell me when you're done — I'll check again"
  - Wait for them to confirm, then re-test

**Clay:**
> Clay enriches companies with data we don't have in HubSpot — portfolio size, recent news, additional contacts, and market intel.

- Try `get-credits-available` or similar lightweight Clay call
- If it works: "Clay is connected! You have credits available."
- If it fails: Same guided flow as HubSpot — walk them through Settings → Connectors → Clay → Connect

**Gmail (BOTH connectors):**
> There are two Gmail connectors, and you need both:
> - **Google Gmail** lets the plugin read and search your inbox
> - **Gmail** (the second one) lets it create drafts in your inbox
>
> Let me check both...

- Test the read connector (e.g., `read_gmail_profile`)
- Test the draft connector (e.g., `get_profile` from the gmail tools or `list_labels`)
- If both work: "Both Gmail connectors are connected!"
- If only one works: Explain which one is missing and why they need it
- If neither works: Walk through connecting both, one at a time

**Slack:**
> Slack is how the plugin talks to **Ozzy**, our deep research bot. When you run research on a company, the plugin can send a message to the `#ozzy` Slack channel to trigger Ozzy. Ozzy crawls the web — company sites, press releases, LinkedIn, conference lists — and posts back detailed portfolio and people intel.
>
> Let me check if Slack is connected...

- Try a lightweight Slack call (e.g., `slack_search_channels` with query "ozzy")
- If it works: "Slack is connected! I can see the #ozzy channel."
- If it fails: Same guided flow — walk them through Settings → Connectors → Slack → Connect

After verifying Slack is connected, confirm access to the `#ozzy` channel:
- Try `slack_read_channel` with channel_id `C0AEK5BFHSA` and limit `1`
- If it works: "I can read the #ozzy channel — Ozzy is ready to go."
- If it fails: "I can't access the #ozzy channel. You may need to join it first — search for `#ozzy` in Slack and click Join. Let me know when you're in."

After all connectors are checked:
> **All connectors are set!** That's the hardest part done. Now let's make sure your emails will look professional.

### Phase 3: Email Signature Check

First, check `config/settings.json` → `bdr.signature_html`.

**If `signature_html` is already populated (non-empty):**

Skip this phase entirely. Just note:

> **Signature already configured** — your drafts will include your email signature. (Run `/hqo:onboard` anytime to update it.)

Move on to Phase 4.

**If `signature_html` is empty or missing:**

> When I create email drafts for you, I need to include your signature directly in the draft body. I can pull it automatically from one of your sent emails.
>
> First, I need you to send yourself a quick email so I can grab the signature:
>
> 1. Open **Gmail** and compose a new email **to yourself**
> 2. Set the subject to: **bdr toolkit**
> 3. Type anything in the body — just make sure your signature is included
> 4. **Send it**
> 5. Tell me when it's sent

**After the BDR confirms they sent the email:**

Extract the signature automatically using the Gmail API:

1. Search sent emails for the signature source:
   - Use `gmail:fetch_emails` with `query="subject:bdr toolkit"`, `label_ids=["SENT"]`, `include_payload=true`, `max_results=5`
2. Get the message ID from the results
3. Fetch the full message:
   - Use `gmail:fetch_message_by_message_id` with the message ID and `format="full"`
4. Extract the HTML content:
   - Navigate the message payload parts to find the `text/html` MIME part
   - The HTML body is base64url-encoded in `part.body.data`
   - Decode it (base64url with `===` padding for safety)
5. Isolate the signature:
   - Look for the Gmail signature wrapper: `<div dir="ltr" class="gmail_signature" data-smartmail="gmail_signature">`
   - Extract everything from `<span class="gmail_signature_prefix">` through the closing signature `</div>` tags
   - The signature prefix (`-- `) and the signature div together form the complete signature block
   - HqO signatures typically use nested `<table>` elements with social icons, contact info rows, and a disclaimer table

6. Save the extracted HTML to `config/settings.json` under `bdr.signature_html`
7. Confirm to the BDR:

> **Signature extracted and saved!** Here's a preview of what it'll look like in your drafts:
>
> [render a plain-text representation of the signature for them to verify]
>
> Every email draft I create will include this signature at the bottom. If you ever need to update it, run `/hqo:onboard`.

**If extraction fails** (no signature found in the email, no HTML part, etc.):

> I couldn't find a signature in that email. Let's try another way — tell me your full name, title, phone number, and HqO email, and I'll generate one for you.

When they provide the details, generate a simple, professional HTML signature block:

```html
<table cellpadding="0" cellspacing="0" style="font-family: Arial, sans-serif; font-size: 13px; color: #333333;">
  <tr><td style="font-weight: bold; font-size: 14px;">[Full Name]</td></tr>
  <tr><td style="color: #666666;">[Title] | HqO</td></tr>
  <tr><td>[Phone] | <a href="mailto:[Email]" style="color: #1a73e8; text-decoration: none;">[Email]</a></td></tr>
  <tr><td><a href="https://hqo.co" style="color: #1a73e8; text-decoration: none;">hqo.co</a></td></tr>
</table>
```

Show the generated HTML to the BDR and ask if it looks right. Offer to adjust. Save to `config/settings.json` under `bdr.signature_html`.

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
> **Company Overview** — Background pulled from **Clay** and **HubSpot**.
>
> **Key Contacts** — People at the company mapped by priority using **Clay** (contact discovery) and **HubSpot** (existing records). Asset Managers and executives come first because they're our primary buyers. If Clay finds contacts that aren't in HubSpot yet, the plugin will ask if you want to add them to the CRM.
>
> **Customer Parallels** — Existing HqO customers in the same market, pulled from **HubSpot**, that we can reference in outreach.
>
> **Engagement Status** — Shows if anyone on the team has already reached out to these contacts, from **HubSpot** engagement history.

Then:
> That's the research side. When you're ready to generate emails for these contacts, just tell me. Then run `/hqo:draft-emails` to put them in your Gmail.

### Phase 6: Wrap-up

> **You're all set!** Here's your cheat sheet:
>
> **Commands:**
>
> | Command | What It Does |
> |---------|-------------|
> | `/hqo:research [company]` | Research an account, score ICP, map contacts, generate emails |
> | `/hqo:draft-emails` | Create Gmail drafts from generated emails |
> | `/hqo:onboard` | Update your name, email, tone, or signature |
>
> **What's working behind the scenes:**
>
> | Tool | What It Does |
> |------|-------------|
> | **HubSpot** | CRM lookups — company records, contacts, engagement history |
> | **Clay** | Company enrichment — portfolio size, markets, news, additional contacts |
> | **Ozzy (via Slack)** | Deep web research — triggered on request for verified portfolio data, ICP validation, and people intel |
> | **Gmail** | Creates draft emails directly in your inbox for review before sending |
>
> Start with `/hqo:research [company]` whenever you're ready to prospect a new account.
>
> Any questions?

## Error Recovery

- If a connector check fails mid-flow, don't restart from the beginning. Pick up where you left off.
- If the test run fails (e.g., Clay credits exhausted), explain what happened and assure them the setup is still complete — they just need to resolve the specific issue.
- If they seem confused at any point, offer to explain further. Never rush past a step.

## Important Rules

- NEVER skip the connector checks (HubSpot, Clay, Gmail, Slack) on first-time setup. A missing connector will break the workflow later and the BDR won't know why.
- NEVER assume connectors are working — always test them with an actual call.
- ALWAYS wait for the BDR's response before moving to the next phase. This is a conversation, not a monologue.
- Keep each message focused on ONE thing. Don't dump a wall of text.
- Deep research config is managed by GTM Engineering, not BDRs.
