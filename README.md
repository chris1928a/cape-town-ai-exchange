# Cape Town AI Exchange

A working hub for the Cape Town AI builder community. Tools, quickstarts, meetup notes — open source, fork freely.

---

## Hi crew,

This repo is the front door for everything we share at the AI Exchange. If you are new here, start with one of the quickstarts below — each is a working tool you can deploy in under an hour.

If you build something interesting, open a PR and add it here. The point is to compound: every meetup adds one more thing the next builder can stand on.

---

## Quickstarts (pick one and ship it tonight)

| Tool | What it is | Time to deploy | Cost |
|---|---|---|---|
| **[Erler Brain](quickstarts/erler-brain.md)** | Personal AI Assistant on Telegram + WhatsApp + Google Workspace | ~30 min | ~5 EUR/month |
| **[Sales Leadership Board](quickstarts/sales-board.md)** | Turn CRM call data into individualized SDR coaching | ~1 hour | ~15 EUR/month |
| **[Build your own Claude Code Skill](quickstarts/claude-skills.md)** | Encode your domain expertise as a reusable Claude skill | ~20 min | free |

---

## Tools we share

### [Erler Brain](https://github.com/chris1928a/erler-brain-public)
Your personal Telegram bot that searches your Drive, drafts your emails, and runs on a 5 EUR Lightsail box.
Two-model routing keeps API spend low. Hybrid intent detection (regex + Gemini fallback) keeps it snappy.

### [Sales Leadership Board](https://github.com/chris1928a/sales-leadership-board)
Pulls calls and emails from Close.com or HubSpot every day. Scores each call across 6 dimensions.
Tells each rep what to practice this week. ~15 EUR/month vs. Gong at 9,300 EUR/year for 3 users.

---

## Shared Claude Code Skills

The [`skills/`](skills/) folder is a community library of reusable Claude Code Skills.
Drop one into your `~/.claude/skills/` and it loads automatically when relevant.

Currently shipping: [example-pricing-coach](skills/example-pricing-coach/SKILL.md) (template).
Add yours via PR — see [skills/README.md](skills/README.md).

---

## Meetup notes

Notes from past sessions live in [`meetup-notes/`](meetup-notes/).
Template: [meetup-notes/README.md](meetup-notes/README.md). Add yours after each session.

---

## Contributing

How to add quickstarts, skills, or meetup notes: [CONTRIBUTING.md](CONTRIBUTING.md).
Community norms: [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

The bar is intentionally low: under-30-minute deploy, under-100-EUR/month cost,
sanitized of any client data, MIT or similar license.

---

## Contact

Questions, ideas, want to host the next session: [chris@erlerventures.org](mailto:chris@erlerventures.org)

This is a side project — replies are not instant, but they happen.
