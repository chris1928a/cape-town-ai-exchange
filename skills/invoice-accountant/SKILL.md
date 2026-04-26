---
name: invoice-accountant
description: |
  Automates the monthly inbound-invoice pipeline for a small business: routes
  email-based receipts to the accountant via Gmail forward filters, downloads
  portal-only receipts via authenticated Chrome automation, tracks every
  expected vendor in a Google Sheet, and reminds the human only on edge cases.
  Triggers on: invoice, receipt, billing, accounting, missing invoices,
  monthly invoice run, Gmail filter, DATEV upload, accountant inbox,
  bookkeeping, vendor receipts, expense report, fetch invoices, missing
  receipts, portal login receipts, OpenAI receipt, Anthropic receipt, AWS
  receipt, Stripe receipt, "what's missing this month".
  Skip for: tax filing, balance sheet analysis, P&L reporting, invoice
  generation (outbound).
---

# Invoice Accountant — Inbound Invoice Pipeline

You orchestrate a 3-tier pipeline that delivers ~90% of inbound vendor invoices
to the bookkeeper or accounting system without human action, and surfaces only
the genuine exceptions.

## Default stance

- **Tier 1 beats Tier 2.** If a vendor sends an email, never scrape their portal.
  Email forwarding is free, robust, and survives portal redesigns.
- **Tier 3 is contracts.** A monthly rent payment with no invoice does not need
  automation — it needs a one-line reference to the contract.
- **Single source of truth.** Every expected vendor lives in one Google Sheet.
  Status updates write back to the sheet so the bookkeeper sees live state.
- **The human only acts on exceptions** — failed logins, expired cookies, new
  vendors. Everything else runs without them noticing.

## Architecture

```
Vendor sends email → Gmail filter → forward to accountant inbox      [TIER 1]
Vendor only has portal → Chrome MCP login → download PDF → forward    [TIER 2]
Vendor is a contract → reference document, no monthly action          [TIER 3]
                          ↓
              All three write status to one Google Sheet
                          ↓
              Bookkeeper has read-only link, sees live state
```

## Methodology

### Step 1: Inventory every vendor

Pull the last 12 months of bank transactions (CSV export from your bank or
accounting tool). Group recurring payees. For each one, classify:

- **Tier 1** if the vendor sends an email receipt (search inbox for their domain
  in the last 90 days)
- **Tier 2** if the receipt only exists behind a login portal
- **Tier 3** if it is a fixed contract (rent, salary, loan, statutory fee)

Single-shot vendors (one purchase, will not repeat) skip the system — handle
them manually.

### Step 2: Set up the accountant forwarding address

For DATEV (German market): each client has a unique email like
`<client-id>@uploadmail.datev.de`. Verify it as a Gmail forwarding address
once — the accountant clicks the confirmation link inside their portal.

For other accounting systems (QuickBooks, Xero, FreshBooks): most have an
inbox-style upload address. Same pattern.

### Step 3: Tier 1 — one Gmail filter per vendor domain

```
Filter query: from:billing@vendor.com has:attachment
Action: Skip Inbox + Apply Label "Forwarded" + Forward to <accountant-email>
```

The catch-all filter `from:receipts+*@stripe.com has:attachment` catches every
Stripe-based SaaS at once — useful default before you map specific vendors.

### Step 4: Tier 2 — recipes per portal

A recipe is a JSON click-path. Schema:

```json
{
  "vendor-id": {
    "url": "https://vendor.com/billing/history",
    "steps": [
      {"action": "navigate", "url": "..."},
      {"action": "wait_for", "selector": "..."},
      {"action": "for_each", "selector": ".invoice-row", "do": [
        {"action": "click_and_download_pdf"}
      ]}
    ],
    "datev_subject_template": "Vendor {month}/{year}",
    "fallback_behavior": "tier3_human_ping"
  }
}
```

The skill walks the recipe via Claude-in-Chrome MCP. Chrome stays logged in
between runs (persistent cookies). On failure (login expired, captcha, selector
broken), the skill pings the human via WhatsApp/Telegram with the portal URL.

### Step 5: Tier 3 — reference document

A flat list: vendor name, contract reference, what bookkeeper needs (none for
fixed payments, scanned PDF for annual policies). This file barely changes.

### Step 6: Master tracker sheet

One row per vendor. Columns: vendor, tier, cycle, last received, status per
month, accountant-forward date, portal URL, notes. Skill writes status changes
back. Bookkeeper has read-only link.

## Common mistakes (call these out)

- **Trying to automate Tier 1 with Playwright instead of Gmail filters.**
  Gmail filters survive forever; Playwright scripts break monthly.
- **Building portal scrapers for vendors that already email receipts.**
  Always check Tier 1 first.
- **Per-vendor recipes for one-off purchases.** If you bought from them once,
  do not write a recipe — just forward the email manually.
- **Storing credentials in the recipe.** Never. Chrome session cookies only.
  If a vendor has 2FA every login, it belongs in Tier 3 (human-assisted).
- **Forgetting to verify the accountant forwarding address.** Gmail will silently
  drop the filter action until verified. Test with one filter first.
- **Hiding the missing-invoice list from the bookkeeper.** They already know
  what is missing — they are chasing you for it. Give them the sheet read-only
  and chase yourself.

## Failure modes and recovery

| Symptom | Cause | Fix |
|---|---|---|
| Filter shows in Gmail but nothing forwards | Forwarding address not verified | Verify once, future filters work |
| Recipe redirects to login page | Cookie expired | Open portal manually, "trust this device" |
| 2FA prompt every run | Vendor disabled trust-device | Move to Tier 3, manual fetch |
| Selector not found | Portal redesigned | Update recipe (read_page on new layout, swap selectors) |
| PDF download saves as HTML | Vendor uses HTML invoice page | Add `download_pdf_or_html_to_pdf` step |
| Monthly run misses a vendor | Vendor changed sender domain | Update filter query, re-test |

## When asked to add a new vendor

Before writing code:
1. Search the inbox for their domain in last 90 days. If found → Tier 1.
2. Check if their billing page has a "send invoices to email" setting.
   If yes → enable it, then Tier 1.
3. Only if neither works → Tier 2 recipe. Test with one month of history first.
4. If 2FA blocks every login → Tier 3, with WhatsApp reminder cadence.

## What to put in `data/`

- `tier-1-providers.json` — list of email-based vendors with `filter_query`,
  expected sender, label, accountant subject template.
- `tier-3-providers.json` — list of contract-based vendors with contract
  reference and any annual action (e.g., "policy arrives by post in January").

## What to put in `recipes/`

- `portal-recipes.json` — keyed by vendor-id, value is the click-path. See
  schema in `references/tier-2-portal-recipes.md`.

## What to put in `references/`

Detailed runbooks, one per tier plus the tracker. The skill loads these on
demand when the human asks for setup help.
