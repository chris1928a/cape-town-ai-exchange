# Tier 3: Contract-Based Vendors (Reference Only)

## Goal
Recurring payments that do not produce a monthly invoice — rent, salary,
loans, statutory memberships. The skill references these but does not act on
them each month.

## Why these belong in Tier 3 (not Tier 1 or 2)

A vendor goes to Tier 3 if **any** of these is true:
- The payment is governed by a contract that already documents the amount
  (rent, salary, loan repayment, leasing)
- The bookkeeper books from the bank statement directly, without an invoice
  PDF
- The receipt arrives once a year on paper (insurance policies, chamber-of-
  commerce fees)
- The vendor enforces 2FA on every single login and there is no trust-device
  option (this rules out automated portal scraping)

## Categories to expect

### Standing orders / direct debits
Examples: office rent, parking spot rental, gym memberships, club dues.
Action: none. Bank statement + the underlying contract are sufficient
documentation.

### Internal payroll
Examples: managing director salary, employee salaries, social security
contributions.
Action: none in this skill. These are booked from your payroll system, not
from invoices.

### Statutory authorities
Examples: tax authority assessments, chamber of commerce fees, trade office
fees.
Action: receipts arrive by post or in an electronic mailbox. Scan + forward
when they arrive (a few times per year).

### Annual policies
Examples: life insurance, business insurance, professional indemnity.
Action: the policy document arrives once per year on paper or via portal.
Scan + forward when it arrives. The skill flags this in the tracker as
`annual_action: true` so you remember to check.

### Tax / legal advisors
Often these professionals already send their own invoices directly to your
accounting system on your behalf. Confirm with them once — if yes, no Tier 1
filter needed either.

### Photographed paper receipts
Examples: fuel stations, taxi cash payments, parking meters.
Action: photograph the receipt → run `images_to_pdf.py` → forward to the
accountant inbox. The skill helps you batch these at month-end.

## What the skill does for Tier 3

In the master tracker sheet:
- Status column shows `Contract` permanently (not `Pending` / `Late`)
- A Notes column references the contract or document name
- Annual vendors trigger a one-line reminder in their anniversary month:
  "<vendor>'s annual policy should arrive this month — watch the post"

The skill does **not** chase Tier 3 vendors. They cannot fail. There is no
invoice to be late.

## Migrating from Tier 3 → Tier 1

Some Tier 3 vendors can move to Tier 1 if you make a small change:

| Vendor type | Migration trigger |
|---|---|
| Insurance company | Check the insurer's portal — many now offer email delivery of policies |
| Chamber of Commerce | Some chambers email invoices on request |
| Tax authority | Many countries offer an electronic mailbox (e.g. ELSTER in DE) |
| Professional advisors | Ask them to email invoices directly to the accountant inbox |

When migration is possible, move the entry from `tier-3-providers.json` to
`tier-1-providers.json` and add a Gmail filter.

## What NOT to put in Tier 3

- Recurring **SaaS subscriptions** — those belong in Tier 1 (email) or Tier 2
  (portal). They have invoices.
- Recurring **mobility / cloud / dev tool** payments — same. Always have
  invoices, just not always by email.
- **One-off purchases.** If it happens once and never again, do not list it
  anywhere — handle the receipt manually when it arrives.
