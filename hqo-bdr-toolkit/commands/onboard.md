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
> 1. Check your connectors (HubSpot, Clay, Gmail)
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

After all connectors are checked:
> **All connectors are set!** That's the hardest part done. Now let's make sure your emails will look professional.

### Phase 3: Gmail Signature

> When I create email drafts for you, I write the body and end with "Best, [Your Name]". Gmail automatically adds your email signature when you send the draft.
>
> So your signature needs to be set up in Gmail. Do you already have an HqO-branded signature in Gmail?

Wait for their answer.

**If yes:**
> Perfect — you're all set on signatures. Let's move on.

**If no or unsure:**
Walk them through it step by step:

> No problem — let me walk you through it. Open Gmail in your browser (mail.google.com) and follow along:
>
> 1. Click the **gear icon** (top right) → **See all settings**
> 2. Scroll down to the **Signature** section
> 3. Click **+ Create new** and name it "HqO Standard"
> 4. Add your signature with:
>    - Your full name
>    - Your title | HqO
>    - Your phone number | your email
>    - hqo.co
>
> If your manager or GTM Engineering gave you an HTML signature file:
> 1. Open the .html file in Chrome (double-click or drag into browser)
> 2. Select all (Ctrl+A / Cmd+A), copy (Ctrl+C / Cmd+C)
> 3. Paste into the Gmail signature editor
>
> Under **Signature defaults**, set it as the default for BOTH "New emails" AND "Replies/forwards", then Save Changes.
>
> Let me know when you're done, or if you need help finding the HTML file.

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

Save answers to `config/settings.json`:
```json
{
  "bdr": {
    "name": "[their name]",
    "email": "[their email]",
    "tone": "[their choice]"
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

> Here's what the plugin found. Let me explain each section:
>
> **ICP Score** — This rates how well the company fits our ideal customer profile, based on portfolio size and deal potential.
>
> **Company Overview** — Background pulled from Clay and HubSpot.
>
> **Key Contacts** — People at the company mapped by priority. Asset Managers and executives come first because they're our primary buyers.
>
> **Customer Parallels** — Existing HqO customers in the same market that we can reference in outreach.
>
> **Engagement Status** — Shows if anyone on the team has already reached out to these contacts.

Then:
> That's the research side. When you're ready to actually generate outreach emails, you'd run `/hqo:outreach [company]` — same thing but with personalized emails at the end. Then `/hqo:push-drafts` to put them in your Gmail.

### Phase 6: Wrap-up

> **You're all set!** Here's your cheat sheet:
>
> | Command | What It Does |
> |---------|-------------|
> | `/hqo:outreach [company]` | Full workflow — research + emails |
> | `/hqo:research [company]` | Research only, no emails |
> | `/hqo:push-drafts` | Create Gmail drafts from generated emails |
>
> Start with `/hqo:outreach [company]` whenever you're ready to prospect a new account. The plugin handles the rest.
>
> Any questions?

## Error Recovery

- If a connector check fails mid-flow, don't restart from the beginning. Pick up where you left off.
- If the test run fails (e.g., Clay credits exhausted), explain what happened and assure them the setup is still complete — they just need to resolve the specific issue.
- If they seem confused at any point, offer to explain further. Never rush past a step.

## Important Rules

- NEVER skip the connector checks. A missing connector will break the workflow later and the BDR won't know why.
- NEVER assume connectors are working — always test them with an actual call.
- ALWAYS wait for the BDR's response before moving to the next phase. This is a conversation, not a monologue.
- Keep each message focused on ONE thing. Don't dump a wall of text.
