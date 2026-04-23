### **Progress log of OpenClaw setup within Docker**


### Reference / Quick-access section

- [LOCALHOST CONTROL URL / GATEWAY DASHBOARD](http://localhost:18789/)
    - Dashboard screenshot
      ![Alt text](dashboard.png)
- GATEWAY TOKEN: aa6b063e906f2f4e3ce6e66669719b78a5838a827f57f3758aa941129c704413█
- Use ```docker ps``` to see health of active docker containers. (Docker image & containers are diff things.)
  - **Docker image** a frozen “snapshot” of your app and everything it needs to run (like a template)
  - **Docker container** a running copy of that image (like the app actually turned on and running)
  - E.G 
  ```cmd
  C:\Users\amodk\Documents\repos\Openclaw-setup>docker ps
  CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS                    PORTS                                                                     NAMES
  08cc50a5471b   openclaw:local   "docker-entrypoint.s…"   18 minutes ago   Up 18 minutes (healthy)   0.0.0.0:18789-18790->18789-18790/tcp, [::]:18789-18790->18789-18790/tcp   openclaw-openclaw-gateway-1
  ```
  - NOTE: ```docker ps``` will only run successfully if docker desktop is actively running. I've configured **Docker Desktop** to startup automatically upon login. 

### Notable work (repo history)

| Date | Work Done (summary) | Work Done (extended description) |
|------|----------------------|----------------------------------|
| 04/14/2026 | **Baseline gateway running + UI reachable**<br>Completed the first working Docker run and confirmed the local Control UI/dashboard was accessible from the host browser. | Baseline OpenClaw-in-Docker setup completed and verified (control dashboard reachable); quick-access notes and open follow-ups captured (e.g. GitHub skill / `gh` still to configure). |
| 04/14/2026 | **README updated with Docker setup + troubleshooting**<br>Consolidated the key setup steps (env, onboarding, compose/run) plus Windows-specific verification and troubleshooting guidance. | Top-level documentation refreshed: progress log surfaced as `README.md`, then expanded with Docker-centric setup, verification, and Windows-oriented guidance. |
| 04/20/2026 | **Repo cleanup: renamed files + clarified log vs overview**<br>Standardized filenames and separated the short `README.md` overview from the detailed `ProgressLog.md`. | Files renamed for consistency across repos; detailed progress kept in this `ProgressLog.md` while `README.md` holds a short overview. |
| 04/21/2026 | **GitHub skill enabled: installed `gh` + authenticated**<br>Installed GitHub CLI inside the running container and logged it in using a classic token, then verified `gh auth status`. | Configured GitHub skill auth inside the running OpenClaw Docker container using a GitHub classic token. Commands: <br><br>**Configuring docker openclaw to use github classic token (PowerShell):**<br>`echo <github classic token> \| docker exec -i openclaw-openclaw-gateway-1 sh -lc "gh auth login --hostname github.com --with-token"`<br><br>**Check if openclaw is authenticated correctly:**<br>`docker exec openclaw-openclaw-gateway-1 sh -lc "gh auth status"`<br><br>**Terminal output:**<br>`github.com`<br>`  ✓ Logged in to github.com as amodkk (/home/node/.config/gh/hosts.yml)`<br>`  ✓ Git operations for github.com configured to use https protocol.`<br>`  ✓ Token: ghp_************************************`<br>`  ✓ Token scopes: project, read:org, repo, workflow` |
| 04/22/2026 | **SearXNG wired as `web_search` (OpenClaw + self-hosted SearXNG in Docker)**<br>Configured the bundled SearXNG provider, fixed networking and JSON access, and verified with `web_search` + direct API calls. | **Config (host file, same data the gateway container uses):** `C:\Users\amodk\.openclaw\openclaw.json` is bind-mounted to `/home/node/.openclaw/` in the gateway—edits on disk are what the container loads.<br><br>**OpenClaw settings added:**<br>• `tools.web.search`: `enabled: true`, `provider: "searxng"` (official id; not a generic `custom` endpoint).<br>• `plugins.entries.searxng`: `enabled: true` (required—the bundled SearXNG plugin is off by default), plus `config.webSearch.baseUrl: "http://searxng:8080"` (origin only; OpenClaw calls `/search` with `format=json` itself).<br>• Optional env: `SEARXNG_BASE_URL` can mirror the same URL for auto-detect, but explicit provider + plugin config was used here.<br><br>**Obstacles → fixes:**<br>1. **Gateway and SearXNG on different Docker networks** — hostname `searxng` did not resolve from the gateway. **Fix:** `docker network connect searxng_default openclaw-openclaw-gateway-1` (re-run after recreating the gateway if needed). *Alternative without extra network:* `baseUrl: http://host.docker.internal:8080` if SearXNG publishes a host port.<br>2. **Bundled plugin disabled** — logs showed `plugins.entries.searxng: … disabled by default` and `WEB_SEARCH_PROVIDER_INVALID_AUTODETECT`. **Fix:** set `plugins.entries.searxng.enabled: true`.<br>3. **SearXNG returned 403 for JSON** — `search.formats` in SearXNG’s `settings.yml` listed only `html`, so `format=json` was forbidden. **Fix:** add `- json` under `search:` → `formats:`, then restart the SearXNG container. **Persistence:** that change was applied inside the SearXNG data volume; bake the same `formats` into any Compose/bind-mounted `settings.yml` so it survives container recreation.<br>4. **“Custom” YAML** (`provider: custom`, `endpoint: …/search`, `params.format`) is **not** the documented OpenClaw shape; use `searxng` + `baseUrl` (and enable the plugin) per [OpenClaw SearXNG docs](https://docs.openclaw.ai/tools/searxng-search).<br><br>**Verification:**<br>• `curl` / Node `fetch` from the gateway to `http://searxng:8080/search?...&format=json` with `Accept: application/json` returns a JSON body (`query`, `results[]`, `suggestions[]`, `unresponsive_engines[]`, etc.).<br>• `openclaw agent --session-id <id> -m "… use web_search …" --json` — first run failed with 403 until JSON was enabled; after fixes, the agent got real search-backed answers. OpenClaw does not print raw SearXNG JSON in chat; inspect via HTTP if you need the full response.<br>• `openclaw doctor` no longer warned about SearXNG config once the plugin was enabled. |
| 04/23/2026 | **Telegram channel: OpenClaw bot DMs + replies (per repo instruction files)**<br>Enabled native `channels.telegram`, token via `.env`, allowlists, streaming off; doc sync + `AGENTS.md` output style. | **Started from (this repo — not auto-read by the gateway at runtime):**<br>• `TELEGRAM CONFIGURATION KEYS.md` — token and ids informed setup; file rewritten to **placeholders + `.env` guidance** (real `TELEGRAM_BOT_TOKEN` in `%USERPROFILE%\.openclaw\.env` only).<br>• `TELEGRAM OUTPUT RULES.md` — state-based / low-noise / rate-minded messaging; the **practical effect** is a new **`## Telegram`** section in **`C:\Users\amodk\.openclaw\workspace\AGENTS.md`** (what the agent session loads). The rules file in this repo has a one-line note pointing to `AGENTS.md` and still holds the full list.<br>• `TELEGRAM INPUT SETUP INSTRUCTIONS.md` — described a **custom** poller + JSON action contract; **on realizing a clash** with OpenClaw (built-in long polling + main agent, not a separate `getUpdates` loop with `{action, instruction}`), the file was **updated at the top** with “How this maps to OpenClaw”; the rest left as *reference* for a possible custom stack.<br><br>**Runtime:** `openclaw.json` `channels.telegram` (enabled, `dmPolicy` + `allowFrom` + `groupAllowFrom` for the owner id, `streaming: "off"`, `groups["*"].requireMention: true`); `TELEGRAM CONFIGURATION RESULT.md` captures the end-to-end summary. Gateway shows Telegram channel provider starting; `openclaw status` — Telegram ON. |

### Revisit in future 

### URGENT (Github configuration)
```bash
◇  Configure skills now? (recommended)
│  Yes
│
◇  Install missing skill dependencies
│  🐙 github
│
◇  Homebrew recommended ──────────────────────────────────────────────────────────╮
│                                                                                 │
│  Many skill dependencies are shipped via Homebrew.                              │
│  Without brew, you'll need to build from source or download releases manually.  │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────╯
│
◇  Show Homebrew install command?
│  Yes
│
◇  Homebrew install ─────────────────────────────────────────────────────╮
│                                                                        │
│  Run:                                                                  │
│  /bin/bash -c "$(curl -fsSL                                            │
│  https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"  │
│                                                                        │
├────────────────────────────────────────────────────────────────────────╯
│
◇  Install failed: github — brew not installed — Homebrew is not installed. Install it from https://brew.sh or install "gh" manually using your system package manager …
Tip: run `openclaw doctor` to review skills + requirements.
Docs: https://docs.openclaw.ai/skills
```




### OPTIONAL
<details>

◆  Gateway bind   
│  ● Loopback (127.0.0.1)  
│  ○ LAN (0.0.0.0)  
│  ○ Tailnet (Tailscale IP)  
│  ○ Auto (Loopback → LAN)  
│  ○ Custom IP




```MD 
### Network Mode Guidance

### Use LAN (recommended if you run OpenClaw in Docker on Windows)
Official Docker documentation defaults to LAN so your browser on the same machine can access:

- http://127.0.0.1:18789  
- http://localhost:18789  

This works with normal port publishing like `-p 18789:18789`.

Loopback inside containers can interfere with this model, so for Docker-based setups with a local control UI, **LAN is the correct choice**.

---

### Use Loopback when
- You are running the gateway directly on the host (not inside Docker)
- You only want it accessible on the current machine
- You do NOT need access from other devices on the network

This is the most restrictive / local-only option.

---

### Use Tailnet when
- You use Tailscale
- You want the gateway accessible via your tailnet IP
- You need secure remote access from other devices or locations

---

### Use Auto when
- You want OpenClaw to automatically choose the appropriate mode
- You are unsure about networking details

Note: For Docker + local UI setups, **LAN is still the most predictable and aligned with defaults**.

---

### Use Custom IP only if
- You specifically need to bind to a particular IP address
- You understand your network routing requirements

This is uncommon for first-time or standard setups.
```


</details>



### MISC / Informative output on setup completion 
```bash
Config warnings:
- plugins.entries.device-pair: plugin disabled (bundled (disabled by default)) but config is present
Config overwrite: /home/node/.openclaw/openclaw.json (sha256 1baf160fdbcfa408ab871f38579af170eb4d87821b2c726ce4caeb9f9fe80ad0 -> aee5188c0cc4974f9c75f831141617428e692240879aff86aec42366ecd1154f, backup=/home/node/.openclaw/openclaw.json.bak)
Config warnings:\n- plugins.entries.device-pair: plugin disabled (bundled (disabled by default)) but config is present
│
◇  Systemd ───────────────────────────────────────────────────────────────────────────────╮
│                                                                                         │
│  Systemd user services are unavailable. Skipping lingering checks and service install.  │
│                                                                                         │
├─────────────────────────────────────────────────────────────────────────────────────────╯
│
◇  Gateway ──────────────────────────────────────────────────────────────────────────────╮
│                                                                                        │
│  Gateway not detected yet.                                                             │
│  Setup was run without Gateway service install, so no background gateway is expected.  │
│  Start now: openclaw gateway run                                                       │
│  Or rerun with: openclaw onboard --install-daemon                                      │
│  Or skip this probe next time: openclaw onboard --skip-health                          │
│                                                                                        │
├────────────────────────────────────────────────────────────────────────────────────────╯
│
◇  Optional apps ────────────────────────╮
│                                        │
│  Add nodes for extra features:         │
│  - macOS app (system + notifications)  │
│  - iOS app (camera/canvas)             │
│  - Android app (camera/canvas)         │
│                                        │
├────────────────────────────────────────╯
│
◇  Control UI ──────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                   │
│  Web UI: http://172.18.0.2:18789/                                                                 │
│  Web UI (with token):                                                                             │
│  http://172.18.0.2:18789/#token=aa6b063e906f2f4e3ce6e66669719b78a5838a827f57f3758aa941129c704413  │
│  Gateway WS: ws://172.18.0.2:18789                                                                │
│  Gateway: not detected (connect failed: SECURITY ERROR: Cannot connect to "172.18.0.2"            │
│  over plaintext ws://. Both credentials and chat data would be exposed to network                 │
│  interception. Use wss:// for remote URLs. Safe defaults: keep gateway.bind=loopback and          │
│  connect via SSH tunnel (ssh -N -L 18789:127.0.0.1:18789 user@gateway-host), or use               │
│  Tailscale Serve/Funnel. Break-glass (trusted private networks only): set                         │
│  OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1. Run `openclaw doctor --fix` for guidance.)                 │
│  Docs: https://docs.openclaw.ai/web/control-ui                                                    │
│                                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────────────────────╯
│
◇  Workspace backup ────────────────────────────────────────╮
│                                                           │
│  Back up your agent workspace.                            │
│  Docs: https://docs.openclaw.ai/concepts/agent-workspace  │
│                                                           │
├───────────────────────────────────────────────────────────╯
│
◇  Security ──────────────────────────────────────────────────────╮
│                                                                 │
│  Running agents on your computer is risky — harden your setup:  │
│  https://docs.openclaw.ai/security                              │
│                                                                 │
├─────────────────────────────────────────────────────────────────╯

◇  Web search ───────────────────────────────────────╮
│                                                    │
│  Web search was skipped. You can enable it later:  │
│    openclaw configure --section web                │
│                                                    │
│  Docs: https://docs.openclaw.ai/tools/web          │
│                                                    │
├────────────────────────────────────────────────────╯
│
◇  What now ─────────────────────────────────────────────────────────────╮
│                                                                        │
│  What now: https://openclaw.ai/showcase ("What People Are Building").  │
│                                                                        │

```