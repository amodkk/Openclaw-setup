# OpenClaw setup (Windows + Docker)

This repository holds **notes and progress** for running [OpenClaw](https://github.com/openclaw/openclaw) in **Docker Desktop on Windows** (WSL 2 backend): isolated runtime, persisted config on the host, and a gateway you reach in the browser.

## What to read here

| File | Description |
|------|--------|
| [ProjectPlan (gemini).md](ProjectPlan%20(gemini).md) | End-to-end plan: prerequisites, clone official repo, `.env`, `docker-setup.sh` vs manual `docker run`, `docker compose up`, control UI on **port 18789**, firewall and Ollama tips. |
| [ProgressLog.md](ProgressLog.md) | What was actually done, quick links (dashboard URL), `docker ps` expectations, Docker Desktop auto-start, open follow-ups (e.g. GitHub skill / `gh` inside the container), and optional **LAN** bind guidance for Docker + local UI. |

## Short operational summary

1. **Docker Desktop** must be running (CLI talks to the engine over the Windows named pipe). With **`restart: unless-stopped`** on the container (or Compose), the gateway typically comes back after reboot **once Docker is up**; starting Docker at login avoids “cannot connect to docker API” and empty `docker ps`.
2. **Control UI**: open [http://127.0.0.1:18789](http://127.0.0.1:18789) or [http://localhost:18789](http://localhost:18789) on the PC. Ignore container-only `172.x.x.x` URLs for the browser; use the published host port instead.
3. **Official workflow** lives in the cloned `openclaw` repo (Compose / setup script there), not in this folder. This repo is the **scratchpad and checklist** next to that work.

For tokens, screenshots, and long verbatim logs, see **ProgressLog.md**.
