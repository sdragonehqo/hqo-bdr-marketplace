---
name: setup
description: Quick reconfiguration for BDRs. Update name, email, tone, or email signature without full onboarding.
arguments: []
---

# /hqo:setup

Walk the BDR through reconfiguring their settings.

## Steps

1. Read current values from `config/settings.json` and show them:
   - "Here are your current settings:"
   - **Name:** [bdr.name]
   - **Email:** [bdr.email]
   - **Tone:** [bdr.tone]
   - **Signature:** [show "✓ Configured" if `bdr.signature_html` is non-empty, or "✗ Not configured" if empty]
2. Ask: "What would you like to update?" — let them pick one or more:
   - Name
   - Email
   - Tone preference (professional, casual, or consultative)
   - Email signature
3. For each selected item, walk through the update:
   - **Name/Email/Tone:** Ask for the new value, confirm, save.
   - **Signature:** Follow the same walkthrough as onboarding Phase 3:
     - If they have an HTML file: right-click → open with text editor → copy raw HTML → paste here
     - If they have a signature in Gmail: extract via Gmail settings → Google Doc → Download as HTML → open in text editor → paste here
     - If they need one created: ask for full name, title, phone, email → generate HTML block → confirm
     - Save to `bdr.signature_html` in `config/settings.json`
4. Save all updates to `config/settings.json`
5. Confirm: "Updated! Here's your new config:" — show the updated values
6. "Run `/hqo:outreach [company]` to start prospecting."

## Notes

- If config/settings.json already has values, show current settings and ask what they want to change — don't re-ask everything
- Deep research config is managed by GTM Engineering, not BDRs
- If the BDR's signature is empty and they're not updating it, gently remind them: "Heads up — you don't have a signature configured yet. Drafts will be created without one. Want to add it now?"
