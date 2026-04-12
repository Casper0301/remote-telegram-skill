# Remote Telegram for Claude Code

Control Claude Code from your phone via Telegram. Full Opus 4.6 with 1M context on your Max plan — no computer needed.

```
You (phone) → Telegram → VPS → Claude Code (Opus 4.6, 1M context)
                                    ↓
                              Your Max plan
                              Home Assistant
                              Gmail, Calendar
                              Supabase, Stripe
                              Netlify, Notion...
```

## What This Is

A Claude Code skill that deploys a fully autonomous Claude Code instance to your VPS, connected to Telegram as a control channel. Message your bot, Claude responds — with full tool access, memory, and your Max plan subscription.

Run `/remote-telegram setup` and the skill walks you through everything.

## Why

Claude Code is powerful but tied to your desk. This skill puts it in your pocket:

- **Fix code from your phone** — push a hotfix while you're at dinner
- **Control your smart home** — "turn off the garage lights" via Telegram
- **Check deployments** — "what's the Netlify build status for autopromo?"
- **Run queries** — "check Supabase for signups today"
- **Manage projects** — works with all your cloud MCP tools

All running on your existing Max plan. No API costs. No extra subscriptions.

## Features

**Self-healing infrastructure**
- 10-second watchdog monitors Telegram polling via TCP connection state
- Auto-recovers from stalled connections in under 30 seconds
- Systemd service with auto-start on boot and crash recovery

**Production-grade stability**
- Watchdog pattern inspired by [OpenClaw](https://github.com/openclaw/openclaw)'s polling-session supervisor
- Never steals messages during health checks (TCP-based, not getUpdates-based)
- Handles all known failure modes: stalled polling, dead processes, auth expiry, plugin conflicts

**Daily maintenance, zero effort**
- Auto-updates Claude Code CLI, Bun, and plugin dependencies daily
- Fresh session every morning (prevents context bloat)
- Bi-directional memory sync between your Mac and VPS

**Full tool access**
- Cloud MCP tools via claude.ai proxy (Stripe, Gmail, Calendar, Supabase, Netlify, Notion)
- Home Assistant via Nabu Casa (or any network-accessible HA instance)
- Any MCP server you can run on a Linux box
- Git repos — clone and work on code remotely

**Smart access control**
- Telegram allowlist locked to your user ID
- No pairing codes exposed to strangers
- OAuth credentials stored securely (never in env vars)

## Architecture

```
┌─────────────┐
│  Your Phone  │
│  (Telegram)  │
└──────┬───────┘
       │
       ▼
┌─────────────────────┐
│  Telegram Bot API    │
│  (Telegram Cloud)    │
└──────┬──────────────┘
       │ grammy long-polling
       ▼
┌──────────────────────────────────┐
│  VPS (always on)                 │
│                                  │
│  ┌───────────────────────┐       │
│  │ bun server.ts         │       │
│  │ (Telegram MCP server) │       │
│  └──────┬────────────────┘       │
│         │ stdio/MCP              │
│  ┌──────▼────────────────┐       │
│  │ claude                │       │
│  │ Opus 4.6 (1M context) │       │
│  │ Max plan              │       │
│  └──────┬────────────────┘       │
│         │                        │
│  ┌──────▼────────────────┐       │
│  │ 10s Watchdog          │       │
│  │ TCP connection monitor │       │
│  └───────────────────────┘       │
│                                  │
│  systemd: auto-start, restart    │
│  timers: update, session reset   │
└──────────────────────────────────┘
       │
       ▼ cloud proxy
┌──────────────────────┐
│  claude.ai MCP tools  │
│  Stripe, Gmail, etc.  │
└───────────────────────┘
```

## What's Included

The skill handles the entire setup:

| Step | What it does |
|------|-------------|
| Dependencies | Installs Claude CLI, Bun, symlinks for global access |
| Authentication | Browser OAuth flow with full MCP scopes |
| Plugin install | Telegram plugin via Claude's official system |
| Bot config | Token, access control, allowlist lockdown |
| Startup script | Network wait, dep install, Claude launch |
| Watchdog | 10s TCP-based polling monitor, auto-restart |
| Systemd services | Main service + update timer + session reset timer |
| Local cleanup | Disables local Telegram plugin to prevent conflicts |
| Memory sync | Bi-directional rsync between Mac and VPS |
| MCP servers | Optional Home Assistant and custom MCP setup |

Plus a `status` command to check health, a `fix` command for common issues, and full documentation of all 7 known failure modes with solutions.

## Requirements

- Linux VPS with SSH access (Ubuntu 22.04+, 2+ GB RAM recommended)
- Claude Max, Pro, or Team subscription
- Telegram bot token (free, takes 30 seconds from @BotFather)
- Node.js 20+ on the VPS

## Installation

**1. Get the skill**

Purchase at **[casperschive.no/marketplace](https://casperschive.no/marketplace)** to get access to the installation repo.

**2. Install into Claude Code**

```bash
# Clone the private repo (you get access after purchase)
git clone https://github.com/casperschive/remote-telegram-pro.git ~/.claude/skills/remote-telegram
```

**3. Run the setup**

```
/remote-telegram setup
```

The skill guides you through everything interactively. Takes about 10 minutes.

## Pricing

**One-time purchase. No subscription. No API costs.**

Your Telegram bot runs on your existing Claude Max plan — the same "unlimited" usage you already pay for, now accessible from your phone.

**[Get it at casperschive.no/marketplace →](https://casperschive.no/marketplace)**

## FAQ

**Does this use API credits?**
No. It authenticates with your Claude Max/Pro subscription via OAuth. Same billing as Claude Desktop — no per-token charges.

**What happens if my VPS reboots?**
The systemd service auto-starts on boot. The startup script waits for network connectivity and reinstalls plugin dependencies before launching. Recovery is fully automatic.

**What if the bot stops responding?**
The built-in watchdog checks every 10 seconds. If polling stalls, it kills and restarts Claude within ~25 seconds. You probably won't even notice.

**Can I use this with Claude Desktop at the same time?**
Yes, but the Telegram plugin must be disabled locally. Two pollers on the same bot token will fight — only one can receive messages. The skill handles this during setup.

**What MCP tools work on the VPS?**
All cloud-proxied tools (Stripe, Gmail, Calendar, Supabase, Netlify, Notion, etc.) work automatically. Local-only tools (Apple Notes, computer-use) don't. Any MCP server that runs on Linux can be added.

**Can multiple people use the same bot?**
Yes — add their Telegram user IDs to the allowlist in `access.json`. Each message goes to the same Claude session.

## Support

Questions? Issues? Open a [GitHub issue](https://github.com/casperschive/remote-telegram-skill/issues) or reach out at [casperschive.no](https://casperschive.no).

---

Built by [Casper Schive](https://casperschive.no) — from 14 hours of debugging grammy polling stalls so you don't have to.
