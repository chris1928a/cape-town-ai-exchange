# Quickstart: Sales Leadership Board

Turn your team's CRM call data into individualized SDR coaching, automated training diagnostics, and ICP validation. Replaces ~9,000 EUR/year of Gong with ~15 EUR/month of API costs.

> Repo: [chris1928a/sales-leadership-board](https://github.com/chris1928a/sales-leadership-board)

---

## What you will end up with

- A Google Sheet with 8 tabs: Master Dashboard, Daily KPIs, Call Analysis, Email Analysis, Lost Reasons, Q&A scripts (44 LARC objection handlers), Leaderboards, individual coaching tabs per rep
- Daily Telegram briefing: who is on track, who needs intervention, what each rep should practice this week
- A Gap-Detection Matrix that turns "the team is underperforming" into 9 specific diagnoses (Effort Gap, Skill Gap, Plateau, etc.)
- Closed-loop coaching: schwächste Dimension this week → specific framework recommendation → re-measured next week

---

## What you need before you start

- Close.com OR HubSpot account with API access
- A Google account (for the Sheet output)
- Anthropic API key (call scoring)
- Deepgram API key (call transcription) — only if you have call recordings
- A small VPS or GitHub Action runner (~5 EUR/month, or free on a raspberry pi)
- ~1 hour for the first deploy

---

## Steps

### 1. Clone and configure (15 min)
```bash
git clone https://github.com/chris1928a/sales-leadership-board.git
cd sales-leadership-board
pip install -r requirements.txt
cp .env.example .env                # API keys
nano config/targets.json            # rep names, ramping phases, KPI thresholds
```

### 2. First test run (10 min)

Pick the entry point that matches your CRM:
```bash
python main_closecom.py    # if you use Close.com
# OR
python main_hubspot.py     # if you use HubSpot
```

This will:
- Pull yesterday's calls / emails / meetings from your CRM
- Score them against your `targets.json`
- Write the result to your configured Google Sheet
- (Skip Deepgram + Claude if you have not set those keys; you still get raw KPIs)

### 3. Add scoring + coaching (20 min)

Once the basic flow works, layer in:
- **Deepgram** for call transcription (~7 EUR/month at 18 hrs of calls)
- **Claude** for scoring (6 dimensions per call, ~3 EUR/month at 1,500 calls)

The scoring logic is in `scoring/call_scorer.py` — adapt the rubric to your sales motion.

### 4. Schedule it (15 min)

Pick one:
- **Cron on a VPS:** `0 18 * * 1-5 cd /path/to/repo && python main_closecom.py`
- **GitHub Actions:** see `.github/workflows/daily.yml.example` (free for public repos, $0.008/min for private)
- **AWS Lambda:** the original deploy target if you prefer serverless

---

## What to customize

- **`config/targets.json`** — your reps, their ramping phase (day 0-30, 30-60, 60-90, 90+), and the KPI targets per phase. Without this, every rep gets the same default targets.
- **`scoring/gap_matrix.py`** — the 2-axis matrix (Activity vs Outcomes) and 9 diagnoses. Tune the thresholds to your sales motion.
- **`training/objection_library.py`** — 10 LARC scripts for the most common objections. Edit them for your offering.

---

## What it costs to run

| Component | Cost |
|---|---|
| VPS or GitHub Actions | ~5 EUR/month (or free on Pi) |
| Deepgram (transcription) | ~7 EUR/month for 18 hrs of calls |
| Claude API (scoring) | ~3 EUR/month for 1,500 calls |
| Google Sheets API | free (60 req/min limit) |
| Telegram Bot | free |
| **Total** | **~15 EUR/month** |

vs. Gong at ~9,300 EUR/year for a team of 3.

---

## The Sherpa Skill (the brain behind the board)

The Sales Board is the runtime; the *reasoning* is captured in a Claude Code Skill called
[sherpa-gtm-sales-intelligence](https://github.com/chris1928a/erler-brain-public/tree/main/skills/sherpa-gtm-sales-intelligence)
in the Erler Brain repo.

If you load the skill into Claude Code, you can ask it things like *"score this call transcript on
my 6 dimensions"* or *"what is the gap diagnosis for these KPIs"* and get the same logic the
runtime uses, but on demand.

---

## When to NOT use this

- Your team is under 2 SDRs (overhead exceeds value)
- You do not record calls (you lose the most valuable signal — call scoring)
- You already have Gong or Chorus and the budget is fine (this is a cost-cutting alternative, not a feature-equivalent product)

---

## Help

Issues + PRs: [chris1928a/sales-leadership-board](https://github.com/chris1928a/sales-leadership-board)
