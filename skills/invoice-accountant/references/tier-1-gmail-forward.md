# Tier 1: Gmail Auto-Forward to Accountant

## Goal
Vendors that send receipts by email — forward each match automatically to your
accountant inbox via Gmail filters. Zero touch after setup.

## Accountant inbox examples

| System | Pattern |
|---|---|
| DATEV Unternehmen Online | `<unique-id>@uploadmail.datev.de` |
| QuickBooks Online | `<your-company>@qbo.intuit.com` |
| Xero | `inbox-<your-org>@xerofiles.com` |
| FreshBooks | `<unique-token>@hub.freshbooks.com` |
| Generic | a dedicated mailbox you forward to (e.g. `bookkeeping@yourbusiness.com`) |

## Prerequisites (one-time)

1. **Verify the forward address.** In Gmail Settings → Forwarding and POP/IMAP →
   "Add a forwarding address" → enter accountant inbox.
2. Gmail sends a verification email to that inbox. The accountant (or you, if
   you have access) clicks the confirmation link.
3. The "Forward to:" filter action will not appear until verification completes.

**Common pitfall:** if the accountant inbox is hosted by a SaaS that does not
expose the verification email to a human (rare), you may need to ask their
support to confirm the forward.

## Filter setup, per vendor

For each entry in `data/tier-1-providers.json`:

1. Open `https://mail.google.com/mail/u/0/#settings/filters`.
2. Click "Create new filter".
3. Paste the `filter_query` into the search field (typically
   `from:billing@vendor.com has:attachment`).
4. Click "Search" and verify the matches are correct.
5. Click "Create filter with this search".
6. Tick:
   - [x] Skip Inbox (Archive)
   - [x] Apply Label: `Forwarded` (create the label first)
   - [x] Forward to: `<accountant-email>`
   - [x] Also apply to matching existing conversations
7. Click "Create filter".

## Catch-all safety net

Before you map every specific vendor, add this generic filter:

```
from:receipts+*@stripe.com has:attachment
```

This catches every Stripe-based SaaS receipt (Anthropic, OpenAI, Notion,
Perplexity, and many others all bill via Stripe). It is a useful default — you
can refine into vendor-specific filters over time.

## Special case: vendors that only invoice on request

Some vendors (small consultants, agencies, freelancers) only send an invoice
when you ask. For these:

- Mark them with `trigger_reminder: true` in `tier-1-providers.json`
- The skill sends a reminder email (and optionally WhatsApp) on the 3rd of each
  month
- When the vendor replies with the PDF, your existing Gmail filter forwards it
  to the accountant automatically

Reminder template:
```
Hi <name>,

Could you send me the invoice for <service> for <month/year>?

Billing address: <your-business>
VAT ID: <your-vat-id>

Thanks!
```

## Validation (monthly)

The skill checks on the 5th of each month:
- Search Gmail for label `Forwarded` after first-of-month
- Compare against expected list from `tier-1-providers.json`
- Any monthly vendor missing a forward → mark as `Late` in the tracker
- Any quarterly/yearly vendor in the right cycle window → check too

If items are still missing after the 10th, the skill nudges you with a
consolidated list.

## Gmail limits to know

- No hard daily forward limit for filter-based forwards
- Multiple filters can forward to the same address (no conflict)
- Verified forward addresses stay valid for ~1 year, then need re-verification
- Filter cap is 5000 — you will not approach this even with 50 vendors
