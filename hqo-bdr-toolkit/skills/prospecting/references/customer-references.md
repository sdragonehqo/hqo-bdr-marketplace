# Customer Reference Rules

## Dynamic Matching (Primary Method)

Query HubSpot for companies where `lifecyclestage = customer` AND `city` or `state` matches the target company's markets.

Match the customer's asset class to the target's asset class before including in any outreach.

## Preferred Customers

Use when a market-match exists. These are deeper relationships with stronger proof points.

| Asset Class | Reference Customers | Notes |
|-------------|-------------------|-------|
| Commercial Office | Vornado, Hines | Hines works as national fallback for any asset class |
| Luxury Residential | Related Companies | Only for luxury resi or mixed-use with resi component |
| Mixed-Use only | Jamestown | Do NOT use for pure office or pure residential |

## Matching Rules

1. **Best match:** Same city + same asset class → use local customer name
2. **Good match:** Same state + same asset class → use regional customer name
3. **Fallback:** No local match → use Hines (national footprint, works for most asset classes)
4. **Never:** Mismatched asset class (e.g., don't cite Jamestown for a pure office deal)

## Usage in Emails

- Reference the customer's peer group, not the company by name when possible ("other owner-operators in Boston")
- If naming a customer, frame as social proof: "At [Customer], the asset management team uses..."
- Never imply endorsement or quote customers directly
- One customer reference per email maximum
