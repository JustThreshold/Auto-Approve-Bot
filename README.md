<div align="center">
  # ⚡ Telegram Auto-Approval & Join Request Bot
  
  **A production-grade, asynchronous Telegram bot built with [Kurigram](https://github.com/KurimuzonAkuma/kurigram) and Async MongoDB (Motor) for automated join request approvals, multi-step scheduling, unified channel customization, tier-based quota management, and Fernet-encrypted MTProto session storage.**

  <p>
    <a href="https://github.com"><img src="https://img.shields.io/badge/Python-3.10%2B-blue.svg?style=flat-square&logo=python" alt="Python 3.10+"/></a>
    <a href="https://github.com/KurimuzonAkuma/kurigram"><img src="https://img.shields.io/badge/Framework-Kurigram%20(Pyrogram)-orange.svg?style=flat-square" alt="Kurigram"/></a>
    <a href="https://www.mongodb.com"><img src="https://img.shields.io/badge/Database-MongoDB%20(Motor)-green.svg?style=flat-square&logo=mongodb" alt="MongoDB"/></a>
    <a href="https://www.docker.com"><img src="https://img.shields.io/badge/Docker-Ready-2496ED.svg?style=flat-square&logo=docker" alt="Docker"/></a>
    <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square" alt="License"/></a>
  </p>
</div>

---

## 🌟 Key Feature Pillars

### 1. ⚡ Instant & Pending Backlog Approvals
- **Token-Bucket Queue Manager (`core/queue_manager.py`)**: High-throughput join request processing without triggering Telegram `FloodWait` (420) exceptions.
- **`/approveall [chat_id] [limit]`**: Safely approve backlogged join requests with precise limit caps, live progress updates, and graceful cancellation.
- **`/queue [chat_id]`**: Real-time position tracking, remaining jobs ahead count, and dynamic ETA calculation.

### 2. 📅 Scheduled Approvals Engine
- **Multi-Step Wizard (`plugins/schedule.py`)**: 5-step inline setup for target channel selection, approval limit, execution time, and timezone.
- **Persistent MongoDB Store (`core/scheduler.py`)**: Jobs persist across bot restarts and are automatically claimed and executed upon waking.
- **Auto-Recovery**: Interrupted in-flight jobs are safely reset to pending on restart via `core/recovery.py`.

### 3. 🛠️ Unified Channel Control Center (`/managechnls`)
- **Per-Channel Toggles**: Auto-approval, CAPTCHA verification (1-Click, Math, Emoji), Avatar requirement, and Combot Anti-Spam (CAS) checks.
- **Rich-Media Welcome & Goodbye Editor**: Text, photos, videos, animations/GIFs, dynamic template variables (`{mention}`, `{first_name}`, `{chat_title}`, `{date}`), and custom inline URL buttons.
- **Live Preview in DM**: Test welcome and goodbye formats instantly without publishing to public channels.
- **Global Channel Defaults (`/defaults`)**: Configure a master template and push it to all managed channels in one tap.

### 4. 💎 Plan & Quota Management (`/plan`)
- **Tiered Plans**: `FREE`, `PRO`, `BUSINESS`, and `UNLIMITED` with channel limits and daily/weekly/monthly quota tracking.
- **Feature Gating**: Premium features (goodbye messages, scheduling, broadcast suite) gated cleanly via `core/quota.py`.
- **Automatic Quota Resets**: Daily counters reset automatically at `QUOTA_RESET_HOUR` with lifetime metrics preserved.

### 5. 🔐 Cryptographic Session Security (`/login`, `/logout`, `/sessions`)
- **Fernet Symmetric Encryption (`core/session_manager.py`)**: Telegram session strings are encrypted with AES-128-CBC before database writes.
- **Zero Plaintext Leakage Guarantee**: Decryption occurs strictly in-memory during MTProto client execution; plain session strings never hit database documents or application logs.

---

## 📂 Project Architecture

```
Auto-Approve-Bot/
├── main.py                      # Application entry point, supervisor loop & graceful shutdown
├── config.py                    # Environment settings, access control & key validation
├── database.py                  # Async MongoDB driver (Motor)
├── helpers.py                   # Keyboards, formatters, rate limiters & UI utilities
│
├── core/                        # Central Engine & Shared Services
│   ├── cache.py                 # In-memory TTL cache for channel settings & permissions
│   ├── permissions.py           # Unified multi-level access control & admin checks
│   ├── queue_manager.py         # Central token-bucket approval queue & live ETA tracker
│   ├── quota.py                 # Plan tiers, usage metrics & automatic reset logic
│   ├── recovery.py              # Crash recovery, DB latency monitor & watchdog
│   ├── scheduler.py             # Persistent MongoDB schedule runner & timezone engine
│   └── session_manager.py       # Fernet-encrypted user session manager
│
├── plugins/                     # Modular Command & Callback Handlers
│   ├── admin.py                 # Super admin moderation (/ban, /unban, /fsub)
│   ├── broadcast.py             # High-speed broadcast suite with live progress
│   ├── captcha.py               # DM CAPTCHA verification callbacks
│   ├── chat_added.py            # Bot chat join/leave listener
│   ├── join_request.py          # Real-time ChatJoinRequest handler
│   ├── leave.py                 # ChatMemberLeft / Kicked goodbye dispatcher
│   ├── logs.py                  # Live log viewer & diagnostic file exporter
│   ├── managechnls.py           # Unified Channel Control Center (/managechnls)
│   ├── mass.py                  # Bulk backlog actions & CSV export
│   ├── pending_request.py       # /approveall & /queue commands
│   ├── plan.py                  # /plan, /quota usage dashboard & tier display
│   ├── schedule.py              # /schedule, /schedules automation wizard
│   ├── session.py               # /login, /logout, /sessions user client manager
│   ├── start.py                 # /start, /help & main menu navigation
│   ├── stats.py                 # /stats, /ping health diagnostics
│   └── user_defaults.py         # /defaults master template manager
│
├── tests/                       # Complete Unit & Integration Test Suite
│   ├── test_managechnls.py
│   ├── test_permissions.py
│   ├── test_queue_eta.py
│   ├── test_queue_manager.py
│   ├── test_quota.py
│   ├── test_quota_reset.py
│   ├── test_recovery.py
│   ├── test_scheduler.py
│   ├── test_scheduler_persistence.py
│   └── test_session_manager.py
│
├── Dockerfile                   # Production container definition
├── docker-compose.yml           # Multi-container orchestration (Bot + MongoDB)
├── requirements.txt             # Python dependencies
└── .env.example                 # Environment configuration template
```

---

## 🚀 Quick Deployment Guide

### Option 1: Docker Compose (Recommended)

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Yahiko5679/Auto-Approve-Bot.git
   cd Auto-Approve-Bot
   ```

2. **Configure `.env` file:**
   ```bash
   cp .env.example .env
   # Edit .env with your BOT_TOKEN, API_ID, API_HASH, OWNER_ID, and SESSION_ENCRYPTION_KEY
   ```

3. **Launch the stack:**
   ```bash
   docker-compose up -d --build
   ```

4. **Monitor live logs:**
   ```bash
   docker-compose logs -f bot
   ```

---

### Option 2: Direct VPS / Local Deployment (Python 3.10+)

1. **Create and activate a virtual environment:**
   ```bash
   python -m venv .venv
   source .venv/bin/activate   # Linux/macOS
   .\.venv\Scripts\activate    # Windows
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run test suite to verify installation:**
   ```bash
   python -m unittest discover tests
   ```

4. **Start the bot:**
   ```bash
   python main.py
   ```

---

## ⚙️ Environment Variables Reference

| Variable | Required | Default | Description |
| :--- | :---: | :---: | :--- |
| `BOT_TOKEN` | **Yes** | — | Telegram Bot Token from [@BotFather](https://t.me/BotFather) |
| `API_ID` | **Yes** | — | Telegram API ID from [my.telegram.org](https://my.telegram.org/apps) |
| `API_HASH` | **Yes** | — | Telegram API Hash from [my.telegram.org](https://my.telegram.org/apps) |
| `OWNER_ID` | **Yes** | — | Telegram User ID of the master bot owner |
| `ADMINS` | No | `""` | Super admin user IDs (space or comma separated) |
| `MONGO_URL` | **Yes** | — | MongoDB Connection URI |
| `DATABASE_NAME` | No | `AutoApproveBot` | MongoDB Database Name |
| `MAX_APPROVALS_PER_SECOND` | No | `25` | Token bucket approval throughput limit |
| `ENABLE_CAS_CHECK` | No | `true` | Enable Combot Anti-Spam (CAS) lookup |
| `START_PIC` | No | `""` | Image URL used for the `/start` welcome banner |
| `LOG_LEVEL` | No | `INFO` | Logging verbosity (`DEBUG`, `INFO`, `WARNING`, `ERROR`) |
| `SCHEDULER_TIMEZONE` | No | `UTC` | Default timezone for scheduled approval jobs |
| `DEFAULT_PLAN` | No | `FREE` | Default plan tier assigned to new users |
| `QUOTA_RESET_HOUR` | No | `0` | Local hour (0-23 UTC) at which daily quotas reset |
| `CACHE_TTL_SECONDS` | No | `300` | TTL in seconds for channel settings and permission cache |
| `SESSION_ENCRYPTION_KEY` | **Yes** | — | 32-byte url-safe Fernet encryption key for user sessions |
| `PORT` | No | `8080` | Port for health check web server (Render compatibility) |

---

## 📖 Bot Commands Reference

| Command | Permission | Description |
| :--- | :--- | :--- |
| `/start` | All Users | Open the main interactive navigation menu and quick start guide |
| `/help` | All Users | Step-by-step tutorial on adding the bot to channels and groups |
| `/managechnls` | Channel Admins | Unified Channel Control Center (toggles, welcome/goodbye, live preview) |
| `/defaults` | Channel Admins | Master channel settings template: configure once, push to all channels |
| `/approveall [chat] [limit]` | Channel Admins | Bulk approve pending join request backlog with limit cap |
| `/queue [chat]` | Channel Admins | Live approval queue status, position, ETA, and cancellation |
| `/schedule` | Channel Admins | 5-step wizard to automate future join request approval runs |
| `/schedules` | Channel Admins | List and cancel upcoming scheduled approval tasks |
| `/plan` | All Users | View current plan tier, feature checklist, and usage quota bars |
| `/login` | All Users | Connect user session via phone OTP or session string |
| `/logout` | All Users | Revoke and securely delete stored user session |
| `/sessions` | All Users | View connected session status and masked phone number |
| `/stats` | All Users | Global bot statistics: approved requests, users, and channels |
| `/ping` | All Users | System diagnostics: DB latency, queue size, scheduler status |
| `/broadcast` | Master Admin | Broadcast messages to all bot users with live progress meter |
| `/logs` | Master Admin | View recent logs or export full application log file |
| `/ban [user_id]` | Master Admin | Ban a malicious user from interacting with the bot |
| `/unban [user_id]` | Master Admin | Remove a user ban |
| `/fsub [chat_id]` | Master Admin | Configure force-subscription channel requirement |

---

## 📜 License

Distributed under the **MIT License**. See `LICENSE` for more information.
