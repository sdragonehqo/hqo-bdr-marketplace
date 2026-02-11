---
name: setup
description: First-time setup for a new BDR. Asks for name, email, and tone preference, then saves to config/settings.json.
arguments: []
---

# /hqo:setup

Walk the BDR through initial configuration.

## Steps

1. Ask: "What's your first name?"
2. Ask: "What's your HqO email?"
3. Ask: "Tone preference — professional, casual, or consultative?" (default: professional)
4. Save answers to `config/settings.json`
5. Confirm: "You're all set, [name]. Run `/hqo:outreach [company]` to start prospecting."

## Notes

- If config/settings.json already has values, show current settings and ask if they want to update
- Webhook URL is configured by GTM Engineering, not BDRs
