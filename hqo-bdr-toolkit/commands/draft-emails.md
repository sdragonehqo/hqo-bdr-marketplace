---
name: draft-emails
description: Create Gmail drafts directly from generated outreach emails using the Gmail connector. Run after /hqo:research has generated emails.
arguments: []
---

# /hqo:draft-emails

Create Gmail drafts for all generated emails using Cowork's built-in Gmail connector. No external tools or API keys needed.

## Output Rules

**Do NOT print internal reasoning, `<thinking>` tags, tool-call rationale, JSON assembly logic, signature handling details, or decision-tree logic in the chat.** The BDR should only see clean status updates:

- Brief confirmation when the draft file is saved (with file link)
- The approval prompt ("Ready to create these drafts in Gmail?")
- A one-line status per draft as it's created (e.g., "✓ Draft 1/8: Claire Outram — created")
- Final summary (count created, any skips/failures)

All internal processing — reading settings, assembling HTML bodies, appending signatures, converting line breaks, calling the Gmail MCP tool, handling errors — stays **completely behind the scenes**. Do NOT narrate what you are doing, which tools you are calling, or how you are building each draft.

## Prerequisites

- Emails must have been generated in this session (via `/hqo:research`)
- BDR identity (name, email) must be confirmed
- Gmail connector must be enabled in Cowork
- BDR signature HTML should be saved in `config/settings.json` → `bdr.signature_html` (set during `/hqo:onboard`)

## Steps

1. **Collect** — Gather all generated emails from the current session
2. **Validate** — Check each email:
   - Contact email is valid format
   - Body ends with BDR sign-off (no signature block in body — signature appended separately)
   - Subject line is present and non-empty
3. **Sort** — Order by priority (high → medium → low)
4. **Save Draft File** — Write all validated emails to a JSON file in the workspace:
   - File: `drafts-[company-slug]-[date].json`
   - Contains full metadata (company, BDR, timestamp) + all draft objects + skip list
   - The draft file stores plain-text bodies (signature is appended at Gmail creation time, not in the file)
   - Present the file link to BDR in chat so they can review before Gmail creation
   - Ask BDR: "Ready to create these drafts in Gmail?"
5. **Create Drafts** — After BDR confirms, for each validated email, use the Gmail MCP `create_email_draft` tool:
   - `recipient_email`: contact's email address
   - `subject`: generated subject line
   - `body`: assembled HTML body (see Signature Assembly below)
   - `is_html`: `true` (when signature is present)
   - Do NOT send — drafts only
   - Report each draft as it's created (e.g., "Draft 1/8: Claire Outram — created")
6. **Confirm** — Report final results to BDR in chat:
   - Count of drafts created
   - List any skipped contacts with reasons
   - Remind BDR to review each draft in Gmail before sending

## Signature Assembly

For each draft, assemble the final `body` at creation time:

1. Read `bdr.signature_html` from `config/settings.json`
2. Take the plain-text email body and convert `\n` to `<br>`
3. If `signature_html` is non-empty:
   - Append `<br><br>` after the sign-off as a spacer
   - Append the raw `signature_html` value
   - Set `is_html: true` on the `create_email_draft` call
4. If `signature_html` is empty:
   - Send as plain text (`is_html: false`) — no signature block
   - Warn the BDR: "No signature configured — drafts won't include a signature. Run `/hqo:onboard` to add one."

## Error Handling

- No emails generated → "Run `/hqo:research [company]` first to generate emails"
- Gmail connector not enabled → "Enable the Gmail connector in Cowork settings"
- Single draft fails → Log the error, continue with remaining drafts, report failures at end
- All drafts fail → Check Gmail connector permissions, suggest reconnecting

## Safety

- NEVER auto-send emails. Always create drafts only.
- Always show draft file and get BDR confirmation before creating drafts in Gmail.
- BDR must manually review and send each draft from Gmail.
