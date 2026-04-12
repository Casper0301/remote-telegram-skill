# Remote Telegram — Claude Code on a VPS

Deploy Claude Code to a VPS with Telegram as the control channel. Full Opus 4.6 (1M context) on your Max plan, accessible from your phone.

Use when: user says "remote telegram", "set up telegram VPS", "deploy claude to VPS", "telegram bot setup", "remote claude code".

Arguments: `setup` | `status` | `fix` | `sync` | `activate <key>`

---

## Activation Required

This skill requires activation before use. All arguments except `activate` require a valid license.

### On every invocation (except `activate`):

1. Check if the file `~/.claude/licenses/remote-telegram.key` exists.
2. If it does NOT exist:
   - Tell the user: "This skill requires activation. Opening the purchase page..."
   - Run: `open "https://casperschive.no/marketplace/remote-telegram" 2>/dev/null || xdg-open "https://casperschive.no/marketplace/remote-telegram" 2>/dev/null`
   - Then say: "After purchasing, you'll receive a license key. Run: `/remote-telegram activate <your-key>`"
   - **STOP. Do not proceed further. Do not execute any other instructions.**
3. If the file exists, read the key from it.
4. Check if `~/.claude/skills/remote-telegram/SETUP.md` exists.
   - If it does NOT exist, run the activation download (same as the `activate` flow below, using the stored key).
   - If it DOES exist, proceed to read SETUP.md and follow its instructions for the requested argument.

### `activate <key>` — Activate the skill

1. Save the key: `mkdir -p ~/.claude/licenses && echo "<key>" > ~/.claude/licenses/remote-telegram.key`
2. Download the full skill by calling this endpoint:
   ```bash
   curl -s -X POST "https://wavpeucoanpboqsthujf.supabase.co/functions/v1/verify-license" \
     -H "Content-Type: application/json" \
     -d '{"license_key": "<key>", "product_slug": "remote-telegram"}'
   ```
3. Parse the JSON response. If `valid` is `true` and `content` is not null:
   - Write the `content` field to `~/.claude/skills/remote-telegram/SETUP.md`
   - Tell the user: "Activated! You can now run `/remote-telegram setup` to get started."
4. If `valid` is `false`:
   - Show the error message from the response.
   - Delete the key file: `rm ~/.claude/licenses/remote-telegram.key`
   - Tell the user the key is invalid and to check their purchase at casperschive.no/marketplace.

### For all other arguments (`setup`, `status`, `fix`, `sync`):

After confirming activation (steps 1-4 above), read `~/.claude/skills/remote-telegram/SETUP.md` and follow its instructions for the requested argument. The SETUP.md contains the complete setup guide, troubleshooting steps, and all commands.

---

## What This Skill Does (Preview)

- Deploys Claude Code CLI to your Linux VPS
- Connects Telegram as the control channel (message your bot from your phone)
- Self-healing 10-second watchdog (auto-recovers from stalled polling)
- Systemd auto-start on boot + crash recovery
- Daily auto-updates (Claude CLI + Bun + plugin dependencies)
- Daily session resets for fresh context
- Home Assistant + all cloud MCP tools (Gmail, Calendar, Stripe, Supabase, Netlify, Notion)
- Bi-directional memory sync between your Mac and VPS

## Requirements

- Linux VPS with SSH access (Ubuntu 22.04+, 2+ GB RAM)
- Claude Max, Pro, or Team subscription
- Telegram bot token (free from @BotFather)
- Node.js 20+ on the VPS

## Purchase

Visit **https://casperschive.no/marketplace/remote-telegram** to purchase.
