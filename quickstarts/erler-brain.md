# Quickstart: Erler Brain

Deploy your own personal AI assistant on Telegram in about 30 minutes. Total ongoing cost: ~5 EUR/month for the VPS, plus AI API usage (~3-10 EUR/month at personal volume).

> Repo: [chris1928a/erler-brain-public](https://github.com/chris1928a/erler-brain-public)

---

## What you will end up with

- A Telegram bot only you can talk to
- Search your Google Drive: `/drive contract Q3`
- Read your Gmail inbox: `/email is:unread`
- Draft replies in your tone (Claude Sonnet for important ones, Gemini Flash for the rest)
- Search a personal RAG over markdown / PDF docs you index yourself
- Optional: WhatsApp inbound via self-hosted Evolution API

---

## What you need before you start

- AWS Lightsail account (or any 1 GB Linux VPS — DigitalOcean, Hetzner work too)
- Telegram account
- Anthropic API key: [console.anthropic.com](https://console.anthropic.com)
- Google AI Studio key for Gemini: [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
- 30 minutes

---

## Steps

### 1. Create the bot (5 min)
- Message [@BotFather](https://t.me/BotFather) → `/newbot` → copy the token
- Get your numeric Telegram ID: send any message to your bot, then visit
  `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`

### 2. Spin up Lightsail (5 min)
- Lightsail console → Ubuntu 22.04 → 5 USD plan → static IP
- SSH in

### 3. Install + configure (15 min)
```bash
sudo apt install -y python3.11 python3.11-venv git
git clone https://github.com/chris1928a/erler-brain-public.git
cd erler-brain-public
python3.11 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env                              # fill in your tokens
cp config/rules.yaml.example config/rules.yaml    # who gets Claude vs Gemini
cp config/context.md.example config/context.md    # tell it about you
python main.py                                    # boot it
```

In Telegram: `/start` → you should see the help text.

### 4. Run as a service (5 min)
Follow the systemd snippet in [SETUP.md → Step 7](https://github.com/chris1928a/erler-brain-public/blob/main/SETUP.md#step-7-run-as-a-systemd-service).

---

## What to customize

- **`config/context.md`** — who you are, your ventures, your writing style. The Brain uses this as the system prompt for Claude.
- **`config/rules.yaml`** — per-contact routing rules. High-stakes contacts get Claude Sonnet drafts you approve; everything else can auto-reply with Gemini.
- **`bot/handlers/`** — drop a new handler file to add a new command (`/myhandler`). The pattern is in `ARCHITECTURE.md`.

---

## What it costs to run

- AWS Lightsail (1 GB): 5 USD/month
- Anthropic Claude Sonnet: ~3 EUR/month at personal volume (only high-stakes routes hit it)
- Google Gemini Flash: practically free under 1M tokens/day
- Telegram: free

**Total: ~5-15 EUR/month** depending on how chatty you are.

---

## When to NOT use this

- You need multi-tenancy (this is built for one user)
- You need 99.9% SLA (Lightsail goes down sometimes — restart your VPS)
- You need data residency outside the EU (default region is eu-central-1, change if needed)

---

## Help

Bugs / questions: open an issue on [chris1928a/erler-brain-public](https://github.com/chris1928a/erler-brain-public/issues).

If you ship a meaningful improvement (new handler, better RAG, bug fix), PRs welcome.
