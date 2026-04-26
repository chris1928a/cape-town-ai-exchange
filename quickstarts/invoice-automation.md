# Quickstart: Invoice Automation Pipeline

> A working day to set up. After that, ~90% of vendor invoices land in your
> accountant's inbox without anyone forwarding them by hand.
>
> Cost: free (Gmail filters + Google Sheet) + Claude Code Max.
> Optional: ~10 EUR/month for InvoiceFetcher as a safety net.

---

## What you get

- Vendors that email receipts → forwarded automatically to your accountant
  inbox the moment they arrive.
- Vendors with login-only portals → Claude logs in monthly, downloads the PDF,
  forwards it.
- Fixed-contract payments (rent, salary, statutory fees) → referenced in one
  place, no recurring action.
- One Google Sheet shows the bookkeeper which receipts are in and which are
  out — read-only link, no signup.
- One consolidated WhatsApp / Telegram message at month-end if anything needs
  your attention.

## Why it works

The realisation: most SMBs have ~20 recurring vendors. Roughly 60% email
receipts (Anthropic, Stripe, Google Workspace, Microsoft, GitHub, etc.). About
30% only show receipts in a portal (Uber, OpenAI Platform, AWS Billing). Only
10% truly need ongoing human attention (2FA, paper post, photographed fuel
receipts).

The pipeline matches that distribution: email vendors get a Gmail filter,
portal vendors get a Chrome-MCP recipe, contract vendors get a one-line
reference. Each tier uses the right tool for the job — no vendor is harder
than it needs to be.

---

## Prerequisites

- Claude Code with Claude-in-Chrome MCP enabled
- A Gmail account where vendor receipts arrive
- An accountant inbox you can forward to (DATEV upload mail, QuickBooks
  inbox, Xero email, or a generic mailbox)
- A Google Sheet you can use as the master tracker
- Chrome logged into the portals you want to scrape (one-time)

## Step 1 — Inventory (30 min)

Pull the last 12 months of bank transactions. Group by recurring payee. For
each group, classify:

```
Tier 1 — search inbox: have they emailed me a receipt in the last 90 days?
Tier 2 — no email; do they have a billing page behind login?
Tier 3 — fixed contract (rent, salary, loan, statutory fee)
```

Single-shot vendors skip the system. Anything you bought once, handle
manually.

## Step 2 — Install the skill (5 min)

```bash
git clone https://github.com/chris1928a/cape-town-ai-exchange.git
mkdir -p ~/.claude/skills
cp -r cape-town-ai-exchange/skills/invoice-accountant ~/.claude/skills/
```

Restart Claude Code. The skill loads when you ask about invoices, receipts,
or accounting.

## Step 3 — Configure your vendors (15 min)

In `~/.claude/skills/invoice-accountant/`, copy the example files to your
real config files:

```bash
cd ~/.claude/skills/invoice-accountant/data
cp tier-1-providers.example.json tier-1-providers.json
cp tier-3-providers.example.json tier-3-providers.json

cd ../recipes
cp portal-recipes.example.json portal-recipes.json
```

Edit each one to match your inventory from Step 1. Drop vendors you do not
have, add ones you do. Keep the schema.

## Step 4 — Verify the accountant forwarding address (10 min, one-time)

In Gmail Settings → Forwarding and POP/IMAP, add your accountant inbox as a
forwarding address. Gmail will send a verification email — the accountant
clicks the confirmation link inside their portal.

You only do this once. After that, every filter you create can forward to
this address.

## Step 5 — Set up Tier 1 Gmail filters (45 min)

For each vendor in `tier-1-providers.json`, ask the skill:

> Set up a Gmail filter for Anthropic — search `from:receipts+anthropic@stripe.com`,
> apply label `Forwarded`, forward to my accountant inbox.

The skill drives Chrome through the Gmail filter UI. Repeat for each Tier 1
vendor. After about 45 minutes you have ~15 filters in place — and they will
keep working forever.

Add the catch-all `from:receipts+*@stripe.com has:attachment` as a safety
net — it catches every Stripe-based SaaS that you have not mapped explicitly.

## Step 6 — Test Tier 2 recipes one at a time (1-2 hours)

For each portal in `portal-recipes.json`, log into Chrome manually first,
click "Trust this device" on any 2FA prompt, then ask the skill:

> Test the OpenAI Platform recipe — dry run, just download last month's
> invoice without forwarding.

If the PDF lands in `~/Downloads/Invoices/openai-platform/`, you are good.
Then ask:

> Run the recipe for real this time, forward to my accountant.

If a recipe fails, the skill tells you why (login expired, selector not
found, captcha). Fix the cause once, the recipe is stable from there.

## Step 7 — Set up the master tracker (15 min)

Create a new tab in your accounting Google Sheet called `Invoice-Tracker`.
Ask the skill:

> Populate the tracker with one row per vendor from my Tier 1 / 2 / 3 files.
> Add conditional formatting: green for Done, yellow for Pending, red for
> Late.

Then share the sheet read-only and send the link to your bookkeeper.

## Step 8 — Schedule the monthly run (5 min)

```
/schedule cron "0 9 3 * *" "Run invoice-accountant: monthly fetch and update tracker"
```

This fires on the 3rd of each month at 9 AM. The run:
1. Validates Tier 1 filters fired in the previous month
2. Walks every Tier 2 recipe, downloads, forwards
3. Sends a single consolidated message if anything needs your attention
4. Updates the tracker sheet

The schedule only fires when your laptop is awake and Claude Desktop is
open — but the skill catches up automatically the next time you reconnect.

---

## Maintenance

The system needs ~5 minutes of attention per month:
- If a Tier 2 recipe breaks (portal redesign), update the selectors in the
  recipe JSON.
- If a vendor changes their billing email sender, update the Tier 1 filter
  query.
- If you add a new vendor, add a row to the right tier file and (for Tier 1)
  create a Gmail filter for it.

Everything else is automatic.

## What about new SaaS that uses Stripe?

You probably do not need to do anything. The catch-all
`from:receipts+*@stripe.com has:attachment` already forwards their receipts.
Add a vendor-specific entry only if you want a cleaner subject line in the
accountant inbox.

## What about photographed paper receipts?

Drop them into a folder like `~/Downloads/PaperReceipts/`. Once a month, ask
the skill to convert and forward — it merges them into one PDF per vendor and
sends to the accountant inbox.

## What if I travel a lot and my laptop is closed?

Two options:
1. The schedule waits and catches up when you reconnect. Receipts forwarded
   one week late are still on time for monthly accounting.
2. Use a third-party safety net like InvoiceFetcher.com (~10 EUR/month) for
   the most critical Tier 2 portals. They scrape server-side and forward to
   DATEV directly. The skill orchestrates around them.

---

## Source

The skill: [`skills/invoice-accountant/`](../skills/invoice-accountant/SKILL.md)

Built and refined inside one accountant's workflow over a quarter of monthly
runs. Numbers based on real vendor counts: 16 email vendors, 10 portal vendors,
14 contract vendors. Runtime: ~90% zero-touch after setup.
