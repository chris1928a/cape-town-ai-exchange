# Shared Claude Code Skills

Each subfolder is a Claude Code Skill that any crew member can drop into their own `.claude/skills/`.

## Install one

```bash
git clone https://github.com/chris1928a/cape-town-ai-exchange.git
mkdir -p ~/.claude/skills
cp -r cape-town-ai-exchange/skills/example-pricing-coach ~/.claude/skills/
```

Restart Claude Code. The skill loads when the description matches your prompt.

## Available skills

| Skill | Domain | Author |
|---|---|---|
| [example-pricing-coach](example-pricing-coach/SKILL.md) | B2B SaaS pricing strategy | Chris (template) |
| [invoice-accountant](invoice-accountant/SKILL.md) | Inbound invoice automation: Gmail forward + Chrome-MCP portal scraper + Google Sheet tracker | Chris |

## Add yours

See [CONTRIBUTING.md](../CONTRIBUTING.md#adding-a-skill) for the workflow.

The bar: single-topic, sanitized of personal context, with at least one trigger keyword the description matches reliably.

## What about the Sherpa GTM skill?

That one ships inside the [Erler Brain repo](https://github.com/chris1928a/erler-brain-public/tree/main/skills/sherpa-gtm-sales-intelligence) because it pairs tightly with the [Sales Leadership Board](https://github.com/chris1928a/sales-leadership-board) runtime. Read it as a worked example of a mature skill — it is ~400 lines and shows how to layer methodology, references, and style guidance.
