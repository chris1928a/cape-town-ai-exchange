# Master Tracker — Google Sheet

## Goal
One sheet that shows the bookkeeper, in real time, which receipts arrived for
which month and which are still outstanding. Read-only link, no account
required.

## Why a Sheet (not Notion, not a database)

- The bookkeeper already lives in spreadsheets and they likely cannot edit your
  Notion workspace without an account
- Read-only sharing in Google Sheets needs no signup
- Every accountant on earth knows how to read a sheet
- You can paste the link into an email and they have it forever

## Schema

| Col | Field | Type | Example |
|---|---|---|---|
| A | Vendor | string | Anthropic API |
| B | Vendor-ID | string | anthropic |
| C | Tier | int (1/2/3) | 1 |
| D | Cycle | enum | monthly / quarterly / yearly / irregular |
| E | Last-Received | date | 2026-04-03 |
| F-Q | Status-YYYY-MM | enum (12 columns, one per month of current year) | Done |
| R | Forward-Date (latest) | date | 2026-04-03 |
| S | Portal-URL (Tier 2 only) | url | https://platform.openai.com/account/billing/history |
| T | Filter-Query (Tier 1 only) | string | from:receipts+anthropic@stripe.com |
| U | Notes | string | "..." |

## Status values (cells F-Q)

| Status | Meaning | Color |
|---|---|---|
| `Done` | Receipt received and forwarded to accountant | green |
| `Forwarded` | Auto-forward triggered, in flight | light green |
| `Pending` | Expected but not yet received | yellow |
| `Late` | Past day-5 of the month, still missing | orange |
| `Awaiting-Human` | Tier 2 fallback active, needs login | red |
| `Reminder-Sent` | Tier 1 reminder email sent (request-based vendors) | blue |
| `Contract` | Tier 3 static, no monthly action | gray |
| `Annual` | Tier 3 yearly, not relevant this month | gray |
| `n/a` | No invoice expected (standing orders, payroll) | gray |

## Conditional formatting

Apply to cells F-Q (status columns):

| Condition | Format |
|---|---|
| Text equals "Done" or "Forwarded" | Green background |
| Text equals "Pending" | Yellow background |
| Text equals "Late" or "Awaiting-Human" | Red background |
| Text equals "Reminder-Sent" | Blue background |
| Text equals "Contract" / "Annual" / "n/a" | Gray background |

## Sharing with the bookkeeper

1. File → Share → "Anyone with the link" → "Viewer"
2. Copy link, paste into a one-time email to the bookkeeper:
   > "This sheet shows what receipts you can expect for each month. Green =
   > already in your inbox. Red = something I owe you. Bookmark it."
3. Optional: give the bookkeeper edit access to the Notes column only (using
   sheet-protected ranges) so they can flag issues back to you.

## Update logic (the skill writes here)

| Trigger | Action |
|---|---|
| Tier 1 forward fires (label `Forwarded` applied to a new email) | Find the matching vendor by filter query, set this month's status to `Forwarded`, update Last-Received |
| Tier 2 recipe completes successfully | Set this month's status to `Done`, set Forward-Date, log PDF path in Notes |
| Tier 2 recipe fails | Set status to `Awaiting-Human`, trigger Tier 3 ping |
| Day-5 sweep | For every monthly vendor with status not in {Done, Forwarded, n/a, Contract}, set to `Late` |
| Annual cycle anniversary | Flag in Notes: "Annual receipt expected this month" |

## Querying the sheet (skill-internal)

The skill reads the sheet via the Google Sheets `gviz` CSV endpoint:
```
https://docs.google.com/spreadsheets/d/<sheet-id>/gviz/tq?tqx=out:csv&gid=<tab-gid>
```
This works because the user is logged into their Google account in the same
Chrome session that Claude-in-Chrome controls.

For writes, the skill drives Chrome directly:
- Click target cell
- Type the new value
- Press Tab/Enter to commit

For bulk updates (e.g. month-rollover), the skill builds a TSV payload and
pastes into cell A2 via the JavaScript clipboard API.

## Initial population

Run once at setup. The skill reads `tier-1-providers.json`,
`tier-3-providers.json`, and `recipes/portal-recipes.json`, writes one row per
vendor with the right Tier, Cycle, and reference fields.

After that, the sheet is the authoritative state — you can edit a vendor's
notes there and the skill will pick up the change on its next read.
