# Codex Task Brief — Day 1–2: Minimal Jarvis Loop

**Owner / approver:** TT (Mark) — only TT merges.
**Writer:** Codex. **Reviewer:** Claude (read-only).
**Branch:** `feat/d1-core-loop` → Pull Request. Do not push to `main`.

## Goal
Discord message from TT → route to a model tier → call model → reply in Discord.
Nothing more.

## Environment
- Python 3.13, plain venv (no Docker yet)
- Config: `config/models.yaml`, `config/rules.yaml` (already in repo — read, do not edit)
- Secrets from `.env` (never committed)

## Files to create
```
requirements.txt          # openai, discord.py, python-dotenv, pyyaml, apscheduler — pinned with ==
.env.example              # OPENROUTER_API_KEY=, DISCORD_BOT_TOKEN=, OWNER_DISCORD_ID=
app/__init__.py
app/main.py               # loads .env + config, starts Discord bot
app/config.py             # loads and validates YAML; fails fast on missing keys
app/llm/client.py         # the ONLY module that calls a model
app/core/router.py        # picks tier from routing rules in models.yaml
app/interface/discord_bot.py
tests/test_router.py
tests/test_client.py      # mock the HTTP call; no real API calls in tests
```

## Behaviour
1. Bot responds **only to OWNER_DISCORD_ID** (DM or a mention). Ignore everyone else silently.
2. `router.py`: message starting with `!deep` → `standard` tier (strip the prefix); otherwise `default_tier`. Tiers with `enabled: false` must never be selected.
3. `client.py`: use the `openai` SDK with `base_url` and key from the provider in `models.yaml`. Read `max_tokens`, `temperature`, timeout from config. Request OpenRouter usage data in the response so cost is available (needed for budget in Day 3–4).
4. Error handling per `routing.fallback`: retry once on the same tier, then reply with a short error. Never escalate to a more expensive tier automatically.
5. Reply in Discord with the model's answer. Split replies over 2000 characters.
6. Print one console line per request: tier, model, tokens in/out, cost. **Never print keys, tokens, or full message text.**

## Hard constraints
- No model call anywhere except `app/llm/client.py`.
- No hard-coded model IDs, URLs, or keys in Python files.
- No new dependencies beyond `requirements.txt` without asking.
- Do not create or modify anything in `config/`.
- Out of scope (later days): approval, budget, kill switch, SQLite logging, tools, scheduler.

## Acceptance criteria (TT checks before merge)
- [ ] `pip install -r requirements.txt` works on a clean Python 3.13 venv
- [ ] `python -m app.main` starts and the bot shows online in Discord
- [ ] TT sends "hello" → reply from the `cheap` tier
- [ ] TT sends "!deep explain X" → reply from the `standard` tier
- [ ] A message from another account gets no reply
- [ ] `pytest` passes
- [ ] `git grep` finds no keys or tokens in the repo
- [ ] Console output contains no secrets

## Before TT runs it for the first time
- Set a hard credit limit in the OpenRouter dashboard (budget code arrives Day 3–4)
- Verify every model ID in `models.yaml` on openrouter.ai/models
