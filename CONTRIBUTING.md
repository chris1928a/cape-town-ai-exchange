# Contributing

The Cape Town AI Exchange grows by what each crew member ships and shares. This is the practical guide for adding your work to the repo.

---

## What belongs here

| Type | Where it goes | Bar |
|---|---|---|
| **Tool you built** | New file in `quickstarts/` | Under-30-min deploy, under-100-EUR/month, MIT or similar permissive license |
| **Reusable Claude Skill** | New folder in `skills/your-skill/` | Single-topic, sanitized, with description + body + at least one example |
| **Meetup notes** | New file in `meetup-notes/YYYY-MM-DD-topic.md` | Honest takeaways, names with consent, no client confidential |
| **Reference material** | Link from a quickstart, do not duplicate | Permanent URL preferred (GitHub, papers, blog posts) |

What does NOT belong: vendor pitches, gated content, anything that requires a paid SaaS to evaluate, anything that leaks client data.

---

## Adding a quickstart

1. Copy `quickstarts/erler-brain.md` as a template
2. Replace each section with your tool's specifics
3. Test that someone NEW can follow the steps without asking you
4. Add a row in the table in `README.md`
5. Open a PR titled `quickstart: <your tool name>`

The acceptance criterion is: another crew member can deploy your tool from the quickstart alone, without any DM follow-up.

---

## Adding a skill

1. Read `quickstarts/claude-skills.md` if you have not built a skill before
2. Create `skills/<skill-name>/SKILL.md` with a clear description + methodology body
3. Optionally add `skills/<skill-name>/references/` for deeper material
4. Look at `skills/example-pricing-coach/` for a working reference
5. Open a PR titled `skill: <name>`

Skills should encode *one expert mental model*. Three small skills > one mega-skill.

---

## Adding meetup notes

Format: `meetup-notes/YYYY-MM-DD-topic-shortname.md`

Suggested structure (see `meetup-notes/README.md` for the full template):
- Date, attendees (first name + handle, with consent), topic
- 3-5 takeaways
- Tools demoed with links
- Open questions for next time

Honest and short beats polished and long.

---

## PR workflow

1. Fork the repo
2. Branch off `main`: `git checkout -b quickstart/your-tool`
3. Commit your changes (one logical change per PR)
4. Push and open a PR against `chris1928a/cape-town-ai-exchange:main`
5. Tag a reviewer if you want fast feedback (otherwise expect 2-5 days)

PRs that touch other people's tools should ping the original author first.

---

## What gets rejected

- Tools that need a paid trial to evaluate
- Skills that are personal-context-heavy (use your own `.claude/skills/` instead)
- Meetup notes with client names without consent
- Anything that leaks API keys, tokens, or .env-style secrets (see `.gitignore`)

If a PR gets rejected, the maintainer will explain why in the PR. Most rejections are about scope, not quality — small, focused PRs are easier to ship than mega-PRs.

---

## Code of conduct

Be useful. Be honest. Do not waste people's time. We are a small crew without a moderation team — keep it like that by being the kind of contributor you want others to be.
