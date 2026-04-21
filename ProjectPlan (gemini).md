# Setting up OpenClaw on Docker (Windows)

Using Docker on Windows is the recommended way to run OpenClaw: you get an isolated environment and reliable auto-start behavior.

---

## 1. Prerequisites

| Item | Notes |
|------|--------|
| **Docker Desktop** | Install from [Docker Desktop](https://www.docker.com/products/docker-desktop/). Enable the **WSL 2** backend during setup. |
| **Git** | Install [Git for Windows](https://git-scm.com/download/win) to clone the repository. |
| **Hardware** | At least **4 GB RAM** (8 GB recommended) and **1–2 vCPUs**. |

---

## 2. Initial configuration

### Clone the repository

Open **PowerShell** or **Windows Terminal** and run:

```powershell
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### Create a config directory

Persist data outside the container by creating a local folder:

```powershell
mkdir C:\openclaw
```

### Environment variables

Use the **repository root** (same folder as `.env.example`, after `cd openclaw`). 

That `.env` is what Docker Compose and the setup script use. The `C:\openclaw` folder is separate: it is bind-mounted to `/home/node/.openclaw` for persistent OpenClaw data (`openclaw.json`, etc.), not for copying `.env.example` into.

Copy the example and edit it with your API keys (e.g. Anthropic or OpenAI):

```powershell
copy .env.example .env
notepad .env
```

---

## 3. Deployment

You can use the official automated script or run Docker manually. 

### Option A — Automated (recommended)

Run `docker-setup.sh` from a bash-compatible shell (**Git Bash** or **WSL**):

```bash
./docker-setup.sh
```

This builds images, runs onboarding, and **normally finishes by starting the gateway** (`docker compose up -d`). If **`docker ps`** is empty afterward, start the stack yourself from the repo root (see below).

### Option B — Manual `docker run`

In PowerShell, launch the latest image with persistence:

```powershell
docker run -d --restart unless-stopped --name openclaw `
  -v C:\openclaw:/home/node/.openclaw `
  -p 18789:18789 `
  ghcr.io/openclaw/openclaw:latest `
  node openclaw.mjs gateway --allow-unconfigured
```

---

## 4. Onboarding and verification

1. **Wizard** — Complete the onboarding prompts (model, channels, etc.) until you see **onboarding complete**.

2. **Start the gateway (right after onboarding)** — Having the **image** in Docker Desktop is not enough; a **container** must be running. From the **repository root** (`openclaw` folder, same place as `docker-compose.yml`):

   ```powershell
   docker compose up -d
   ```

   If `docker-compose.extra.yml` exists (often created by the setup script), use:

   ```powershell
   docker compose -f docker-compose.yml -f docker-compose.extra.yml up -d
   ```

   You only need this again after you stop the stack or remove containers—not on every reboot (Compose usually uses `restart: unless-stopped` so the gateway comes back when Docker starts).

   **Option B only:** if you use **`docker run`** instead of Compose, skip `docker compose up` and ensure your **`openclaw`** container is running (`docker ps`).

3. **Confirm** — You should see **`openclaw-gateway`** with port **18789** published:

   ```powershell
   docker ps
   ```

4. **Web UI** — Open [**http://127.0.0.1:18789**](http://127.0.0.1:18789) (or `localhost`). If onboarding printed a URL with a **`172.x.x.x`** host, ignore that for the browser on your PC—use **127.0.0.1** and keep the **`#token=...`** fragment if one was shown.

---

## 5. Windows-specific troubleshooting

### Firewall

If the UI does not load, add an inbound rule (run **PowerShell as Administrator**):

```powershell
netsh advfirewall firewall add rule name="OpenClaw" dir=in action=allow protocol=TCP localport=18789
```

### Local models (Ollama)

To use Ollama on the host alongside Docker, add `--add-host=host-gateway:host-gateway` to your `docker run` command so the container can reach services on the host.
