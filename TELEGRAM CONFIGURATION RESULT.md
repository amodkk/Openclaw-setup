# Telegram configuration result (OpenClaw)

This document records how Telegram was hooked up: native OpenClaw channel, env-based token, allowlists, workspace output rules, and follow-ups.

---

## OpenClaw native Telegram (how it works)

- **Inbound:** The gateway runs the official **Telegram channel** (long polling via grammY), **not** a separate 5s `getUpdates` loop. Your bot token is read from the environment.
- **Outbound:** Replies go back on **Telegram** for the same chat (per [channel docs](https://docs.openclaw.ai/channels/telegram)).
- **Your “JSON command interpreter” spec** (`update_goal` / `pause` / `resume` / …) is **not** how stock OpenClaw works: messages are handled by the **normal agent** with your model. That contract would need a custom skill/plugin or a separate service. A short **“OpenClaw mapping”** block at the top of `TELEGRAM INPUT SETUP INSTRUCTIONS.md` spells this out.

---

## What was configured (live)

| Item | Location |
|------|----------|
| Bot token | `C:\Users\amodk\.openclaw\.env` → `TELEGRAM_BOT_TOKEN=...` (loaded by the gateway; **not** in the git repo) |
| Channel + access | `C:\Users\amodk\.openclaw\openclaw.json` → `channels.telegram` |

### `openclaw.json` settings

- `enabled: true`
- `dmPolicy: "allowlist"` and `allowFrom: ["8693242868"]` (only this Telegram user for DMs)
- `groupAllowFrom: ["8693242868"]` so **group** commands are not open to every member (addresses the security audit that appeared before this)
- `groups["*"].requireMention: true` for groups
- `streaming: "off"` to avoid live “typing preview” style updates (closer to “state change only” output style)

### Logs / status after restart

- `gateway/channels/telegram` → `[default] starting provider`
- `openclaw status` → **Telegram | ON | OK** (token from env)

---

## Output rules (`TELEGRAM OUTPUT RULES.md`)

OpenClaw does not expose a separate “rate limit 10–30s” switch in config; behavior is mostly **model + instructions**. A **Telegram** section was added to `C:\Users\amodk\.openclaw\workspace\AGENTS.md` so the agent keeps Telegram replies **short, state-based, and low-noise**, aligned with that rules file. `TELEGRAM OUTPUT RULES.md` points at that section.

---

## Repo and secrets

- **`TELEGRAM CONFIGURATION KEYS.md`** was rewritten: **no live token** in the file; it documents using `%USERPROFILE%\.openclaw\.env` instead. If a token was ever in git or a public paste, **rotate it in BotFather** and update `.env`.
- **`TELEGRAM INPUT SETUP INSTRUCTIONS.md`** / **`TELEGRAM OUTPUT RULES.md`**: OpenClaw notes and cross-links as described in those files.

---

## What you should do manually

1. In Telegram, open a **DM with your bot** and send a message (you’re allowlisted by user id).
2. If DMs are ignored, confirm your user id: `openclaw logs --follow` in the container and check `from.id` ([OpenClaw docs – finding your Telegram user ID](https://docs.openclaw.ai/channels/telegram#finding-your-telegram-user-id)), then fix `allowFrom` / `groupAllowFrom` if needed.
3. **Optional:** tighten permissions for `openclaw.json` (e.g. `chmod 600` inside the container / correct Windows ACLs) — `openclaw status` may still report the config file as world-writable until fixed.
