# PAIOS — Personal AI Operating System

**Built by Ade Cobbs · Washington, DC · 2026**

PAIOS is a self-hosted AI command center built around an Azure VM called **Olympus** and an AI agent named **Hermes**. It automates my job search, manages my AZ-104 study progress, syncs everything to my Obsidian vault, and gives me a voice-controlled Jarvis interface accessible from any device.

---

## 🌐 Live

**War Room (permanent URL):**
```
https://olympus-warroom.centralus.cloudapp.azure.com
```
Works on desktop, iPhone, and iPad. Add to iPhone home screen: Safari → Share → Add to Home Screen.

---

## Architecture

```
Your Devices (iPhone / Mac / iPad)
         ↓ HTTPS (Let's Encrypt SSL)
┌─────────────────────────────────────────────┐
│          Azure VM — hermes-vm               │
│          Ubuntu 24.04 · D2s v3              │
│                                             │
│  nginx (443) ──► olympus_bridge.py (8080)   │
│                    ├── Advisory Board (OpenRouter LLMs)
│                    ├── ElevenLabs TTS       │
│                    ├── OpenAI Realtime      │
│                    ├── Tool Registry        │
│                    ├── Long-term Memory     │
│                    └── Tier 1 Self-building │
│                                             │
│  job_hunter.py  (cron 8am)                  │
│  job_applier.py (cron 9am)                  │
│  hermes_scanner.py (cron hourly + 7am)      │
└─────────────────────────────────────────────┘
         ↕ cloudflared tunnel
┌─────────────────────────────────────────────┐
│              Mac (launchd)                  │
│  obsidian_receiver.py → iCloud → Obsidian   │
└─────────────────────────────────────────────┘
```

---

## War Room Features

### ⚡ Jarvis — Voice Command Center
OpenAI Realtime API (gpt-4o-realtime-preview) via WebRTC. Fully hands-free with server-side VAD. Voice: `ash`.

On every activation, Jarvis loads your last 4 Obsidian daily notes **and** long-term memory, then gives a personalized 3-sentence brief covering open tasks, job pipeline, and a specific recommendation.

On disconnect: auto-summarizes the session and appends it to today's daily note.

**16 tools:**

| Tool | What it does |
|---|---|
| `get_status` | VM health, disk, memory |
| `get_jobs` | Today's job leads + pipeline stats |
| `save_note` | Creates note in Obsidian Inbox/ |
| `append_daily` | Appends to today's daily note |
| `search_vault` | Full-text search across vault |
| `read_note` | Reads specific note by path |
| `log_study` | AZ-104 study tracker with start/end duration |
| `web_search` | Tavily real-time web search |
| `get_file_context` | Reads uploaded file by name/query |
| `route_to_board` | Routes to board persona in their ElevenLabs voice |
| `run_code` | Executes Python, bash, or **Claude Code CLI** on the VM |
| `read_vm_file` | Reads any file under /opt/olympus/ |
| `write_vm_file` | Writes/appends to any file under /opt/olympus/ |
| `list_vm_files` | Lists directory under /opt/olympus/ |
| `deploy_self` | Restarts olympus-bridge (hot reload) |
| `get_memory` | Retrieves all facts from long-term memory |

---

### 🧠 Long-term Memory
Hermes persists facts across sessions in `/opt/olympus/hermes_memory.json`. Memory is auto-injected into every system prompt and Jarvis opening brief. Just say *"remember that..."* in any chat or voice session.

```
remember(key, value) → writes to hermes_memory.json
recall(key | "all")  → reads back
forget(key)          → removes a key
```

---

### 🤖 Claude Code Integration
`run_code` supports `lang: "claude"` — Hermes can delegate complex coding tasks to the Claude Code CLI running on the VM. Background mode (`background: true`) fires the task async and returns a job ID; poll `GET /api/jobs/<id>` for the result.

```json
{"lang": "claude", "code": "Add a new endpoint to olympus_bridge.py that...", "background": true}
```

---

### 🎙️ Advisory Board
5-member AI board with distinct ElevenLabs voices. Smart routing picks the 1-3 most relevant personas per message.

| Persona | Voice | Expertise |
|---|---|---|
| Hermes | Adam | Command AI, logistics, job pipeline |
| The General | Drew | Career strategy, competitive positioning |
| The Recruiter | Rachel | Resumes, ATS, interviews, callbacks |
| The Architect | Josh | Azure, AZ-104, M365, cloud infra |
| The Coach | Matilda | Mindset, habits, accountability |

Modes: Smart routing · Full Board (all 5) · Hands-free VAD

---

### 🎬 Weekly Board Meeting
One button, structured 5-agenda meeting. Live pipeline data fetched. All 5 personas speak in their own voice. Full transcript saved to Obsidian daily note.

---

### 🎯 Interview Prep
Enter a role. The Recruiter asks 5 tailored questions, gives real-time feedback after each, and delivers a 3-part final eval (strongest moment, biggest gap, one drill). Saved to Obsidian `Interview Prep/`.

---

### 📊 Pipeline Dashboard
Live view of `applications.json` — totals, success rate, platform breakdown (LinkedIn/Indeed/remote), full filterable table.

---

### 💬 Chat — Tool Registry
Every `/chat` request runs against a real tool execution loop. The LLM can call registered tools (web search, file context, memory, etc.), get results, and loop up to 5 rounds before returning a final answer. Tool pills appear in the UI showing what was called.

---

## Job Automation

| Time | Script | Does |
|---|---|---|
| 8:00 AM | `job_hunter.py` | Scrapes Remotive, RemoteOK, Arbeitnow. Scores jobs against my profile (Azure/M365/IT). Generates cover letters. Saves `jobs_today.json`. |
| 9:00 AM | `job_applier.py` | Auto-applies to jobs scoring ≥60 via Playwright. LinkedIn Easy Apply, Indeed Easy Apply, generic forms. Max 10/day. Logs to `applications.json`. |

---

## Gmail / Telegram Scanner

```
cron (hourly) → hermes_scanner.py → gmail_scanner.py → telegram_sender.py → @OlympusBot
```

Alerts for: interview invites, job offers, rejections, recruiter outreach, follow-up reminders, 7am morning brief.

---

## Obsidian Vault Sync

- **Vault:** "Ade's Brain" — iCloud synced
- No Obsidian plugin required — direct filesystem write via `obsidian_receiver.py`
- Cloudflared tunnel (Mac) → auto-URL-sync on restart

What lands automatically:
- Jarvis voice notes → `Inbox/`
- Session summaries → daily note `## Jarvis Session`
- Study logs → daily note `## AZ-104 Study`
- Board meeting transcripts → daily note `## Weekly Board Meeting`
- Interview sessions → `Interview Prep/`

---

## Tier 1 Self-Building

Hermes can read, edit, and restart its own code:

```
read_vm_file  → inspect olympus_bridge.py
write_vm_file → patch it
deploy_self   → restart bridge → new code is live
```

All file ops restricted to `/opt/olympus/` via `os.path.realpath()`.

---

## Stack

| Layer | Tech |
|---|---|
| VM | Azure Standard D2s v3 · Ubuntu 24.04 |
| Web server | nginx + Let's Encrypt SSL |
| Bridge API | Python HTTPServer (port 8080) |
| LLM (chat/board) | OpenRouter → LLaMA 3.3 70b (default) / Nous Hermes 3 405b |
| LLM (code) | Claude Code CLI (`@anthropic-ai/claude-code`) |
| Voice | OpenAI Realtime API (gpt-4o-realtime-preview) via WebRTC |
| TTS | ElevenLabs eleven_turbo_v2_5 |
| Web search | Tavily (1000 queries/month) |
| Job scraping | Remotive · RemoteOK · Arbeitnow · LinkedIn |
| Browser automation | Playwright + Chromium |
| Notes | Obsidian "Ade's Brain" via iCloud |
| Mac services | launchd |

---

## Deploy

```zsh
# Deploy bridge + War Room to VM (~15 seconds)
zsh ~/Documents/Claude/Projects/Olympus/deploy_warroom.sh

# SSH into VM
ssh olympus
```

---

## Roadmap

- [ ] Google Calendar — track interviews, study sessions in morning brief
- [ ] Anki flashcard generator from AZ-104 study notes
- [ ] Telegram → Jarvis two-way (reply to trigger actions remotely)
- [ ] Named Cloudflare tunnel for permanent vault URL
- [ ] GitHub PAT on VM — Hermes can push commits
- [ ] Vercel auto-deploy — chat-to-production pipeline
- [ ] Dev Mode model routing — Claude Opus for heavy code tasks
