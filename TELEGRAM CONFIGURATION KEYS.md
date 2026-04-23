# Telegram secrets and OpenClaw placement

**Do not commit real bot tokens to git.** If a token was ever committed, rotate it in [@BotFather](https://t.me/BotFather) and update your local env.

## Where OpenClaw reads the token

1. **Recommended:** environment variable for the gateway process:
   - **Windows (this setup):** `%USERPROFILE%\.openclaw\.env`
   - **Variable name:** `TELEGRAM_BOT_TOKEN`
   - **Value:** the token from BotFather (format `123456789:AAH...`)

2. **Alternative:** set `channels.telegram.botToken` in `openclaw.json` (discouraged for shared backups—prefer `.env`).

OpenClaw loads `~/.openclaw/.env` automatically. See: [Env vars (OpenClaw FAQ)](https://docs.openclaw.ai/help/faq#env-vars-and-env-loading)

## Your allowlisted Telegram user

Direct messages are restricted to the numeric user id configured under:

- `channels.telegram.dmPolicy`: `allowlist`
- `channels.telegram.allowFrom`: `[ "<your_telegram_user_id>" ]`

**Finding your id:** DM your bot, then `openclaw logs --follow` in the gateway and read `from.id`, or use `getUpdates` as in the [Telegram channel docs](https://docs.openclaw.ai/channels/telegram#finding-your-telegram-user-id).

## YAML example (other tools)

For *non*–OpenClaw HTTP webhooks, a generic `sendMessage` pattern:

```yaml
telegram:
  type: http
  method: POST
  url: "https://api.telegram.org/bot<YOUR_TOKEN>/sendMessage"
  body:
    chat_id: "YOUR_CHAT_ID"
    text: "{{text}}"
```

OpenClaw does **not** use this shape; it uses the Bot API through the built-in Telegram channel. Use `openclaw.json` + `TELEGRAM_BOT_TOKEN` as above.
