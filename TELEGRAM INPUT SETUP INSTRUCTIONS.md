### How this maps to OpenClaw (read this first)

OpenClaw already implements the Telegram **input** path: the gateway uses **long polling** (via grammY) against the Bot API, not a separate 5-second `getUpdates` loop you maintain yourself. Inbound DMs and authorized group messages are turned into the normal agent session; replies go back on the same channel.

- **Token + access:** configure `TELEGRAM_BOT_TOKEN` in `~/.openclaw/.env` and `channels.telegram` in `openclaw.json` (see [Telegram channel](https://docs.openclaw.ai/channels/telegram)). Use `dmPolicy: "allowlist"` and `allowFrom: [ "<your user id>" ]` so only your account can command the bot by DM.
- **The JSON command contract below** (`update_goal`, `pause`, `resume`, …) is **not** the built-in OpenClaw behavior; the model receives your natural language as a normal chat. To match that contract exactly you would add a custom skill/plugin or a thin interpreter—out of scope for “stock” OpenClaw.

The sections below are retained as a **target architecture / reference** for a custom service; treat OpenClaw’s native channel as the supported integration for “message in → agent → message out”.

---

### INPUT CHANNEL (Telegram polling) — *reference / custom design*

Implement a background loop that continuously polls Telegram for new messages using:

GET https://api.telegram.org/bot
<TOKEN>/getUpdates?offset=<last_update_id>

Polling requirements:
Poll every 5 seconds
Maintain last_update_id to avoid reprocessing messages
Extract:
message text
chat_id
update_id
Ignore non-message updates
Ignore duplicate or already-processed updates
Only process messages from the configured allowed chat_id

### MESSAGE INTERPRETATION (LLM step)

Every incoming message must be sent to the already-initialized OpenAI client for interpretation.

Use the following system prompt:

"""
You are a command interpreter for an autonomous coding agent.

Convert natural language instructions into structured JSON commands.

Allowed actions:

update_goal
pause
resume
stop
status

Return ONLY valid JSON in this format:
{
"action": "...",
"instruction": "..."
}

Rules:

If user is unclear, use action: "status"
Do not include any explanation or extra text
"""

User input = raw Telegram message text.

IMPORTANT:

The OpenAI client is already initialized in the runtime environment.
Do NOT attempt to load, configure, or request API keys.

### EXECUTION LOGIC

After receiving JSON from the LLM:

If action == "update_goal":
→ update the agent’s current objective using instruction
If action == "pause":
→ pause current execution loop
If action == "resume":
→ resume execution
If action == "stop":
→ stop current task completely
If action == "status":
→ send current agent state back via Telegram output channel

### OUTPUT CHANNEL INTEGRATION

Reuse the existing Telegram sendMessage function.

After executing any command:

Send a confirmation message back to the user via Telegram
Keep messages short and state-based (not verbose logs)

Examples:

"Paused current task"
"Updated goal"
"Resuming execution"
"Current status: running task X"

### SAFETY RULE

Only accept commands from the configured allowed chat_id.

Ignore all other users.

⏱ PERFORMANCE RULES
Polling interval: 5 seconds
Do not spam Telegram
Only send messages on:
command execution
status requests
errors
Do not send intermediate reasoning steps
Debounce duplicate updates and ignore already-processed update_id values

### ARCHITECTURE SUMMARY

Final system behavior:

Telegram message →
5s polling loop →
OpenAI interpretation (using pre-initialized client) →
Structured action →
Agent execution →
Telegram confirmation

END OF INSTRUCTIONS