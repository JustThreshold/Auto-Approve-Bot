# 🗂️ Universal Project Structure — Auto-Approve Bot

This document outlines the directory structure and component map for the Auto-Approve Telegram bot, supporting the core feature pillars: **Pending Backlog Approvals**, **Scheduled Approvals**, **Unified Channel Customization**, **Plan & Quota Management**, and **Secure Encrypted Session Storage**.

```
Auto-Approve-Bot/
├── main.py                      # Application entry point, supervisor loop & graceful shutdown
├── config.py                    # Environment settings, access control & key validation
├── database.py                  # Async MongoDB driver (Motor)
├── helpers.py                   # Keyboards, formatters, rate limiters & UI utilities
│
├── core/                        # Central Engine & Shared Services
│   ├── __init__.py
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
│   ├── __init__.py
│   ├── test_managechnls.py      # /managechnls permissions, leave handler & UI tests
│   ├── test_permissions.py      # Multi-tier permission & MTProto admin check tests
│   ├── test_queue_eta.py        # Token-bucket ETA calculation & queue position tests
│   ├── test_queue_manager.py    # Queue lifecycle, limit caps & cancellation tests
│   ├── test_quota.py            # Plan tiers, usage tracking & feature gates
│   ├── test_quota_reset.py      # Daily/weekly/monthly quota reset logic tests
│   ├── test_recovery.py         # DB reconnect resilience & in-flight recovery tests
│   ├── test_scheduler.py        # Scheduler timezone parsing & execution tests
│   ├── test_scheduler_persistence.py # mongomock scheduler persistence across restarts
│   └── test_session_manager.py  # Fernet cryptographic round-trip & log audit tests
│
├── Dockerfile                   # Production container definition
├── docker-compose.yml           # Multi-container orchestration (Bot + MongoDB)
├── requirements.txt             # Python dependencies
└── .env.example                 # Environment configuration template
```

## MongoDB Collections (via `database.py`)

| Collection      | Purpose                                                        |
|-----------------|----------------------------------------------------------------|
| `channels`      | Per-channel settings (auto-approve, captcha, welcome/goodbye)  |
| `users`         | Registered bot users (for broadcast & identification)          |
| `join_requests` | Historical join request logs & metrics                         |
| `schedules`     | Scheduled approval jobs: chat_id, limit, run_at, tz, status    |
| `plans`         | Plan tier definitions: limits + feature flags                  |
| `quotas`        | Per-user daily/weekly/monthly usage counters + reset timestamps|
| `sessions`      | Encrypted per-user Telegram session strings (Fernet ciphertext)|
| `user_defaults` | Master channel configuration templates per admin user          |
| `stats`         | Global/per-chat analytics & approval volume counters           |
