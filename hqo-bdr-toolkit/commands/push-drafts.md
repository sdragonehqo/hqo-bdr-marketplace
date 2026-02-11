---
name: push-drafts
description: Create Gmail drafts directly from generated outreach emails using the Gmail connector. Run after /hqo:outreach has generated emails.
arguments: []
---

# /hqo:push-drafts

Create Gmail drafts for all generated emails using Cowork's built-in Gmail connector. No external tools or API keys needed.

## Prerequisites

- Emails must have been generated in this session (via `/hqo:outreach`)
- BDR identity (name, email) must be confirmed
- Gmail connector must be enabled in Cowork

## Steps

1. **Collect** — Gather all generated emails from the current session
2. **Validate** — Check each email:
   - Contact email is valid format
   - Body ends with BDR sign-off (no signature block)
   - Subject line is present and non-empty
3. **Sort** — Order by priority (high → medium → low)
4. **Save Draft File** — Write all validated emails to a JSON file in the workspace:
   - File: `drafts-[company-slug]-[date].json`
   - Contains full metadata (company, BDR, timestamp) + all draft objects + skip list
   - Present the file link to BDR in chat so they can review before Gmail creation
   - Ask BDR: "Ready to create these drafts in Gmail?"
5. **Create Drafts** — After BDR confirms, for each validated email, use the Gmail MCP `create_email_draft` tool:
   - `recipient_email`: contact's email address
   - `subject`: generated subject line
   - `body`: email body text (plain text, not HTML)
   - Do NOT send — drafts only
   - Report each draft as it's created (e.g., "✓ Draft 1/8: Claire Outram — created")
6. **Confirm** — Report final results to BDR in chat:
   - Count of drafts created
   - List any skipped contacts with reasons
   - Remind BDR to review each draft in Gmail before sending

## Error Handling

- No emails generated → "Run `/hqo:outreach [company]` first"
- Gmail connector not enabled → "Enable the Gmail connector in Cowork settings"
- Single draft fails → Log the error, continue with remaining drafts, report failures at end
- All drafts fail → Check Gmail connector permissions, suggest reconnecting

## Safety

- NEVER auto-send emails. Always create drafts only.
- Always show draft file and get BDR confirmation before creating drafts in Gmail.
- BDR must manually review and send each draft from Gmail.
