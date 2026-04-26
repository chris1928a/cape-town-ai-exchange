# Tier 2: Portal Auto-Download via Claude-in-Chrome

## Goal
Vendors that do not send email receipts — the skill logs into their portal each
month, downloads the PDFs, and forwards them to the accountant inbox.

## Prerequisites (one-time per vendor)

1. **Log in once in Chrome.** Use the Chrome profile that Claude-in-Chrome MCP
   controls. Click "Trust this device" / "Remember me" on the 2FA step.
2. **Verify the cookie persists.** Close Chrome, reopen, navigate to the
   billing page. If you still see the page (not a login screen), you are good.
3. **Test the recipe manually before scheduling.** Run it once with `--dry-run`
   and check the PDF lands in `~/Downloads/Invoices/<vendor>/`.

If the vendor enforces 2FA on every login (rare but happens — many banks, AWS
root account), it does not belong in Tier 2. Move it to Tier 3.

## Recipe schema

```json
{
  "vendor-id": {
    "name": "Display name",
    "category": "AI/SaaS|Mobility|Cloud|...",
    "cycle": "monthly|quarterly|irregular",
    "url": "https://vendor.com/billing/history",
    "login_required": true,
    "login_method": "Description for the human",
    "trust_device_supported": true,
    "steps": [
      {"action": "navigate", "url": "..."},
      {"action": "wait_for", "selector": "..."},
      {"action": "for_each", "selector": "...", "do": [...]}
    ],
    "datev_subject_template": "Vendor {month}/{year}",
    "fallback_behavior": "tier3_human_ping"
  }
}
```

## Step actions

| action | parameters | purpose |
|---|---|---|
| `navigate` | `url` | Chrome navigates to the URL |
| `wait_for` | `selector` | Wait until element is on page (10s timeout) |
| `click` | `selector` | Click an element |
| `select` / `filter_period` | `value` | Set date filter (e.g. last_month) |
| `for_each` | `selector`, `do: [...]` | Loop over matching elements |
| `download_pdf` | `save_as` (optional) | Trigger PDF download |
| `download_pdf_or_html_to_pdf` | — | If no direct PDF, render the page as PDF |
| `merge_pdfs` | `to` | Merge multiple downloads into a single file |
| `screenshot_to_pdf` | — | Last resort: screenshot the page as PDF |
| `click_and_download_pdf` | `selector_within` | Convenience: click a link and capture the resulting download |

## Run flow (monthly)

For each recipe in `recipes/portal-recipes.json`:

1. **Pre-check the tracker sheet.** If status for this month is already `Done`,
   skip.
2. **Navigate** to `recipe.url`.
3. **Detect login state.** If `wait_for` selector does not appear within 10s,
   assume the cookie expired → trigger Tier 3 ping, mark as `Awaiting-Human`.
4. **Walk the steps** sequentially. Use `browser_batch` where possible to
   reduce round-trips.
5. **Sweep the download folder.** New PDFs go into
   `~/Downloads/Invoices/<vendor-id>/<YYYY-MM>/`.
6. **Merge if needed.** Vendors with one receipt per item (Uber, Apple) get
   merged to a single monthly PDF.
7. **Forward to accountant.** Create a Gmail draft with subject from
   `datev_subject_template`, attach the PDF using Chrome `file_upload`, send.
8. **Update the tracker sheet.** Status `Done`, forward date set, PDF path
   logged in the notes column.

## Failure handling

The skill triggers `fallback_behavior: tier3_human_ping` on any of:

1. Login page detected after navigate
2. 2FA prompt visible (selector match on `Enter code`, `verification`, `OTP`)
3. Captcha (selector match on `recaptcha`, `hCaptcha`, `challenge`)
4. `wait_for` selector times out (portal redesigned)
5. PDF download fails (network, permissions)

The Tier 3 action sends a single message to the human:

```
Receipt for <vendor> needs your login.
Link: <portal-url>
Reply 'done' once you are signed in — I will pick up from there.
```

The tracker sheet status moves to `Awaiting-Human`. On the next run (manual
trigger or scheduled), the skill retries the recipe.

## Recipe maintenance

When a portal redesigns and your recipe breaks:

1. Open the portal manually in Chrome, navigate to the receipts page.
2. Ask the skill: `read this page and propose new selectors for the receipt
   button and the date filter`.
3. Update the recipe JSON with the proposed selectors.
4. Run a single-vendor test: `run recipe <vendor-id> --dry-run`.
5. If the test passes, the next monthly run picks up the change.

The skill prefers, in order: `data-testid` → `aria-label` → semantic selectors
(`button[type=submit]`) → text content (`button:contains('Download')`).
Avoid CSS classes — they change every redesign.

## Recipes shipped as examples

`recipes/portal-recipes.example.json` includes starter recipes for:
- OpenAI Platform
- Anthropic Console
- Uber Business
- Apple (reportaproblem.apple.com)
- AWS Billing Console
- Stripe Dashboard (for your own Stripe account fees)

Adapt these to your real vendors. The selectors will drift over time — the
schema is what stays stable.
