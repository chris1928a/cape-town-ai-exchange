# Invoice Accountant Skill

Automates inbound vendor invoices for a small business so the bookkeeper gets
~90% of receipts on time without anyone forwarding them by hand.

## What it does

- **Tier 1 (Gmail Auto-Forward):** vendors that email receipts → Gmail filters
  forward them to your accountant inbox automatically.
- **Tier 2 (Portal Auto-Download):** vendors with login-only receipts →
  Claude-in-Chrome MCP logs in, downloads, and forwards them.
- **Tier 3 (Contracts):** fixed monthly payments without invoices → reference
  the contract, no recurring action.
- **Master Tracker (Google Sheet):** every vendor in one sheet, status updated
  automatically, bookkeeper sees live state via read-only link.

## Why this works

About 60% of typical SMB vendors send email receipts. Another 30% have web
portals. Only 10% need ongoing human attention (2FA, paper post, photographed
fuel receipts).

This skill makes the 60% zero-touch, the 30% scheduled, and surfaces the 10%
as a single weekly notification — instead of 20 different ones.

## Install

```bash
git clone https://github.com/chris1928a/cape-town-ai-exchange.git
mkdir -p ~/.claude/skills
cp -r cape-town-ai-exchange/skills/invoice-accountant ~/.claude/skills/
```

Restart Claude Code. The skill loads when you ask about invoices, receipts,
billing, or anything related to your monthly accounting cycle.

## What you fill in

The skill ships with `.example.json` files. You configure once for your
business:

1. **`data/tier-1-providers.json`** — your email-based vendors. For each: the
   Gmail filter query (`from:billing@vendor.com has:attachment`) and the
   subject template for the accountant forward.
2. **`data/tier-3-providers.json`** — your contract-based payments (rent,
   salary, statutory fees) with contract references.
3. **`recipes/portal-recipes.json`** — for each Tier 2 portal: the URL,
   click-path, and PDF-download steps. Examples shipped for Anthropic Console,
   OpenAI Platform, Uber Business, Apple, AWS — adapt to your vendors.
4. **Accountant forwarding email** — DATEV upload mail, QuickBooks inbox, or
   whatever your accounting system provides. Configure once in `SKILL.md`.

## Setup time

- **30 minutes** for the initial vendor inventory (read 3 months of bank
  statements, classify each recurring payee).
- **1 hour** to set up Gmail filters for Tier 1 vendors (15-20 vendors typical).
- **2-4 hours** to write Tier 2 recipes (one per portal you actually use).
- **15 minutes** to create the tracker sheet and share read-only with bookkeeper.

Total: about a working day. After that: zero recurring work for 90% of receipts.

## Cost

- Gmail filters: free
- Claude-in-Chrome MCP: included with Claude Code Max
- Google Sheet: free
- Optional fallback (InvoiceFetcher.com or similar): ~10 EUR/month if you have
  many portals where Chrome cookies expire

## Limits

- Scheduled runs only fire when your laptop is awake and Claude Desktop is open.
  If you travel, the run waits and catches up when you reconnect.
- Vendors with mandatory 2FA on every login (some banks, AWS without trusted
  device) need to stay in Tier 3.
- Marketplace vendors (e.g. Amazon Marketplace third-party sellers) often do
  not provide downloadable invoices — order confirmation HTML is the best you
  get.

## See also

- [Tier 1 setup runbook](references/tier-1-gmail-forward.md)
- [Tier 2 recipe spec](references/tier-2-portal-recipes.md)
- [Tier 3 contract list](references/tier-3-vertrag.md)
- [Master tracker schema](references/sheet-tracker.md)
- [Quickstart deploy guide](../../quickstarts/invoice-automation.md)
