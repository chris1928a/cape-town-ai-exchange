# Quickstart: Build a Claude Code Skill

A Claude Code Skill is a markdown file plus optional reference docs that any Claude Code session can load on-demand to gain domain expertise. It is the cleanest way to package "how I think about X" so you can reuse it across projects — and share it with your team.

> 20 minutes start to working skill. No build step, no SDK, no deploy.

---

## What a skill IS (and is not)

A skill is **NOT** a fine-tuned model, an API, or a plugin. It is a markdown file with a YAML frontmatter that tells Claude:
1. **When** to load it (the `description` field — Claude matches your prompts against it)
2. **What** to do once loaded (the body — methodology, examples, constraints, references)

When the description matches, the body gets injected into Claude's context. That is the whole mechanism.

---

## Where skills live

```
.claude/skills/
   my-skill/
      SKILL.md             # the main file (required)
      references/          # optional supporting docs
         example-1.md
         framework.md
```

Project-scoped: `.claude/skills/` in your repo
User-scoped: `~/.claude/skills/` (works across all projects)

---

## Minimal SKILL.md

```markdown
---
name: pricing-coach
description: |
  Pricing strategy expert for B2B SaaS founders. Use this skill whenever the user
  asks about pricing tiers, willingness-to-pay research, value metrics, packaging,
  expansion revenue, or how to raise prices without losing customers. Triggers on:
  pricing model, price increase, packaging, value metric, willingness to pay,
  expansion ARR, NRR, pricing audit.
---

# Pricing Coach

You are a pricing strategist for B2B SaaS. Default to value-based pricing over
cost-plus. Always ask about the value metric before suggesting tier structures.

## Methodology
1. Identify the value metric (the one number that grows with customer value)
2. Map current pricing to value metric — is it aligned?
3. Find the willingness-to-pay ceiling (Van Westendorp, conjoint, or interviews)
4. Design 3 tiers: bait, target, premium (target captures 80% of revenue)

## Common mistakes I check for
- Per-seat pricing on a non-collaborative product (penalizes adoption)
- "Custom pricing" on the lowest tier (signals you have not done the work)
- Annual prepay discount > 20% (gives away too much)
```

That is it. Drop that file in `.claude/skills/pricing-coach/SKILL.md`, restart Claude Code, and ask *"help me think about pricing for my product"* — the skill loads automatically.

---

## Real example: Sherpa GTM

A more mature skill (~400 lines) lives at
[chris1928a/erler-brain-public/skills/sherpa-gtm-sales-intelligence](https://github.com/chris1928a/erler-brain-public/tree/main/skills/sherpa-gtm-sales-intelligence).

Worth reading before you write your own — note especially:
- How the description packs in trigger keywords (line 2-18 of SKILL.md)
- How references are linked from the body for deep dives
- The "Hinweise für Antworten" section that tells Claude how to *style* its answers

---

## Writing tips

1. **Description is everything.** It is what Claude uses to decide whether to load the skill. List concrete trigger phrases — verbatim. Example: include both "objection handling" and "Einwandbehandlung" if you work bilingual.

2. **Body is methodology, not data.** Put the *how-I-think-about-it* in SKILL.md. Put case-studies, lookup tables, and reference material in `references/`.

3. **Lead with constraints.** Tell Claude what NOT to do. ("Never recommend cost-plus pricing." "Always ask about the value metric first.") Constraints carry more signal than positive instructions.

4. **One skill, one topic.** A skill that handles pricing AND fundraising AND hiring will not match well on any of them. Three skills > one mega-skill.

5. **Test the trigger.** After installing, try 5 prompts in different phrasings. If 4 of 5 do not load the skill, your description needs more trigger keywords.

---

## Sharing skills with the crew

Once you have one that works, drop a PR on [chris1928a/cape-town-ai-exchange](https://github.com/chris1928a/cape-town-ai-exchange) adding it to a shared `skills/` folder. Then anyone in the crew can `git clone` and `cp -r` into their `.claude/skills/`.

The end state: a community library of ~20 skills, each one packaging an expert's mental model, freely composable.

---

## Help

- Official docs: [docs.claude.com/en/docs/claude-code/skills](https://docs.claude.com/en/docs/claude-code/skills)
- Issues / questions: open one on [chris1928a/cape-town-ai-exchange](https://github.com/chris1928a/cape-town-ai-exchange/issues)
