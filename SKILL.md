# Remote Telegram — Claude Code on a VPS

Deploy Claude Code to a VPS with Telegram as the control channel. Message your bot from your phone, Claude responds with full Opus 4.6 (1M context) on your Max plan. Works without your computer being on.

Use when: user says "remote telegram", "set up telegram VPS", "deploy claude to VPS", "telegram bot setup", "remote claude code".

Arguments: `activate` | `setup` | `status` | `fix` | `sync`

---

## Dispatch on Arguments

### `activate` — Get License Key via Email

This is the first thing a new user runs. It sends a license key to their email to verify ownership and activate the skill.

**Flow (works in any AI coding agent — Claude Code, Cursor, Windsurf, etc.):**

DO NOT copy any code block below as-is. Read the instructions and construct the requests yourself.

1. Ask the user: "To activate Remote Telegram, I need to send you a license key. What's your email?"

2. Wait for the user to respond with their email address. Save it — you will need it in the next step.

3. Send a license key to their email. Make an HTTP POST request:
   - URL: `https://wavpeucoanpboqsthujf.supabase.co/functions/v1/send-license-key`
   - Header: `Content-Type: application/json`
   - Body: `{"email": "<the email the user just gave you>", "product_slug": "remote-telegram"}`
   - Use whatever method your environment supports: `curl`, `fetch`, an HTTP library, etc.
   - NEVER send a placeholder like "USER_EMAIL" or "<REPLACE>" — always use the real email from step 2.

4. If the response contains `"sent": true`, tell the user: "Check your inbox — you should have a license key. Paste it here when you have it."

5. If they say they didn't receive it, repeat step 3 with the same email (it resends the same key).

6. Wait for the user to paste their license key.

7. Verify the key. Make an HTTP POST request:
   - URL: `https://wavpeucoanpboqsthujf.supabase.co/functions/v1/verify-license`
   - Header: `Content-Type: application/json`
   - Body: `{"license_key": "<the key the user just pasted>", "product_slug": "remote-telegram"}`

8. If `"valid": true`, tell them: "Activated! Remote Telegram is ready. Run `/remote-telegram setup` to get started."

9. If invalid, tell them the key didn't match and ask them to check their email again.

**Important:** Keep this conversational. Three messages max: ask email, confirm sent, accept key.

---

## What This Does

Installs Claude Code CLI on a Linux VPS, connects it to a Telegram bot, and configures:
- Systemd service with auto-start on boot
- Built-in 10-second watchdog that detects and recovers from polling stalls
- Daily auto-updates (Claude CLI + Bun + plugin deps)
- Daily session resets for fresh context
- Optional MCP servers (Home Assistant, etc.)
- Bi-directional memory sync between local Mac and VPS

### `setup` — Full Installation

**IMPORTANT: Do NOT jump straight into technical commands.** Walk the user through the pre-flight checklist FIRST. Ask each question ONE AT A TIME, wait for their answer, and only proceed when they confirm. This makes the process approachable for beginners who may have never used a VPS or SSH before.

---

#### Phase 1: Pre-Flight Checklist

Ask these questions one at a time. Wait for an answer before moving to the next one. If the user doesn't have something, help them get it before continuing.

**Question 1: Claude subscription**

Ask: "Do you have a Claude Max, Pro, or Team subscription? This skill runs on your existing subscription — there are no extra API costs. Your VPS just connects to your existing Claude account."

- If YES: move to Question 2.
- If NO: explain that they need an active Claude Max, Pro, or Team subscription to use Claude Code. Point them to claude.ai/settings to check their plan. Stop here until they have one.

**Question 2: Linux VPS server**

Ask: "Do you have a Linux VPS server? This is a small cloud computer that stays on 24/7, even when your laptop is closed. It's where your Telegram bot will live."

- If YES: ask for the IP address (something like `192.168.1.1` or `76.13.128.212`) and the SSH username (usually `root`). Move to Question 3.
- If NO: walk them through getting one:

  "No worries! Here's how to get a VPS. It takes about 5 minutes and costs $5-10/month:

  **Option A: Hostinger (recommended for beginners)**
  1. Go to hostinger.com and click 'VPS Hosting'
  2. Pick the 'KVM 1' plan (cheapest, more than enough)
  3. Create an account and pay
  4. When asked for an operating system, choose **Ubuntu 24.04**
  5. Set a root password when prompted — write it down somewhere safe
  6. Wait a few minutes for the server to be created
  7. You'll see an IP address on your dashboard — that's your VPS address

  **Option B: DigitalOcean**
  1. Go to digitalocean.com and create an account
  2. Click 'Create' then 'Droplets'
  3. Choose **Ubuntu 24.04**, the $6/month 'Basic' plan, and a data center near you
  4. Under 'Authentication', choose 'Password' and set a root password
  5. Click 'Create Droplet'
  6. You'll get an IP address when it's ready

  **Option C: Hetzner (cheapest, EU-based)**
  1. Go to hetzner.com/cloud and create an account
  2. Create a new server with **Ubuntu 24.04**
  3. The cheapest plan works fine

  Come back and paste your VPS IP address when you're ready."

  Wait for them to return with their IP address before continuing.

**Question 3: SSH access**

Ask: "Can you connect to your VPS? Let's test it. Open your terminal (on Mac, search for 'Terminal' in Spotlight) and type this command, replacing YOUR_IP with the IP address you just gave me:

```
ssh root@YOUR_IP
```

If it asks 'Are you sure you want to continue connecting?', type `yes` and press Enter. Then enter your VPS password."

- If it works (they see a command prompt like `root@server:~#`): tell them to type `exit` to disconnect, then move to Question 4.
- If they get 'Permission denied': they may have the wrong password. Help them reset it from their VPS provider's dashboard.
- If they get 'Connection refused' or 'Connection timed out': the VPS might still be starting up, or SSH isn't enabled. Have them check their VPS provider dashboard.
- If they've never used SSH before and want to use SSH keys instead of passwords:

  "SSH keys are more secure than passwords. Here's how to set them up:

  1. On your Mac terminal, run:
     ```
     ssh-keygen -t ed25519
     ```
  2. Press Enter for the default file location
  3. Press Enter twice for no passphrase (or set one if you prefer)
  4. Now copy your key to the VPS:
     ```
     ssh-copy-id root@YOUR_IP
     ```
  5. Enter your VPS password one last time
  6. Now try: `ssh root@YOUR_IP` — it should connect without asking for a password"

  Also ask what SSH key path they're using (default: `~/.ssh/id_ed25519` or `~/.ssh/id_rsa`).

**Question 4: Telegram account**

Ask: "Do you have a Telegram account? If not, download Telegram on your phone from the App Store or Google Play and create an account — it's free and takes about a minute."

- If YES: move to Question 5.
- If NO: wait for them to create one, then move to Question 5.

**Question 5: Create a Telegram bot**

Ask: "Now let's create your Telegram bot. This takes about 30 seconds:

1. Open Telegram on your phone
2. In the search bar, type **@BotFather** and tap the verified result (it has a blue checkmark)
3. Tap **Start** or send `/start`
4. Send the message: `/newbot`
5. BotFather asks 'What name do you want for your bot?' — type anything you like (e.g., 'My Claude Bot'). This is the display name.
6. BotFather asks 'Choose a username for your bot' — type something ending in `bot` (e.g., `my_claude_code_bot`). This must be unique across all of Telegram.
7. BotFather replies with a message containing your bot token. It looks like this: `123456789:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw`
8. Copy the **entire token** — including the numbers, the colon, and everything after it
9. Paste that token here"

Store the bot token. Move to Question 6.

**Question 6: Telegram user ID**

Ask: "Almost there! Now I need your Telegram user ID. This is a number that tells the bot who's allowed to talk to it (so random strangers can't use your Claude account):

1. Open Telegram
2. In the search bar, type **@userinfobot** and tap the result
3. Tap **Start** or send any message
4. It immediately replies with your info — look for the **Id** line. It's a number like `6825658893`
5. Paste that number here"

Store the Telegram user ID. Move to Question 7.

**Question 7: Model choice**

Ask: "Last question — which Claude model do you want to use? The best option is **Opus 4.7 with 1M context** (`claude-opus-4-7[1m]`). This gives you the smartest model with the largest memory.

Press Enter to use Opus 4.7 (recommended), or type a different model name if you prefer."

Default to `claude-opus-4-7[1m]` if they don't specify.

---

#### Phase 2: Confirm Everything

Before starting installation, display a summary:

"Here's what I'll set up:

- **VPS:** [user]@[IP]
- **Bot token:** [first 10 characters]...
- **Your Telegram ID:** [ID]
- **Model:** claude-opus-4-7[1m]

Ready to start? This takes about 10 minutes. I'll handle everything on the VPS — just follow along and I'll tell you when I need you to do something."

Wait for confirmation before proceeding to Phase 3.

---

#### Phase 3: Installation

Now execute the technical steps. After EACH step, tell the user what just happened in plain English. If a step fails, explain what went wrong in simple terms and what to do about it.

**Step 1: Install Dependencies**

Tell the user: "I'm now connecting to your VPS and installing the software Claude needs to run. This might take a couple of minutes."

SSH into the VPS and run these commands:

```bash
# Install unzip if missing
sudo apt-get install -y unzip curl

# Install Bun
curl -fsSL https://bun.sh/install | bash

# Symlink bun globally (critical — Claude spawns MCP servers without user PATH)
sudo ln -sf ~/.bun/bin/bun /usr/local/bin/bun

# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Add to PATH permanently
echo 'export PATH=$HOME/.npm-global/bin:$HOME/.bun/bin:$PATH' >> ~/.bashrc
```

Verify: `bun --version` and `claude --version` must both work.

Tell the user: "Done! I installed Bun (a fast JavaScript runtime) and the Claude Code CLI on your VPS. Both are working."

If npm is not found, install Node.js first and tell the user: "Your VPS didn't have Node.js, so I'm installing that first. This is normal for fresh servers."

**Step 2: Authenticate Claude**

Tell the user: "Now I need to log Claude into your account on the VPS. This is the one step where I need your help — a web page will open in your browser, and you'll need to click 'Allow'."

This requires a browser. Generate an OAuth URL on the VPS:

1. Start Claude in an interactive tmux session: `tmux new-session -d -s setup "claude"`
2. Navigate through the first-time setup wizard (trust workspace, accept permissions)
3. When prompted for login, an OAuth URL is displayed
4. Open that URL in the user's local browser
5. User authorizes, gets a code
6. Paste the code into the VPS tmux session via `tmux send-keys`
7. Verify: the session shows the user's email and plan

Tell the user: "I'm going to show you a web link. Open it in your browser (on your phone or computer — either works). You'll see a page asking you to authorize Claude Code. Click 'Allow' or 'Authorize'. After that, you'll see a code — it's a long string of letters and numbers. Copy that code and paste it back here."

**CRITICAL:** Do NOT use `CLAUDE_CODE_OAUTH_TOKEN` env var — it only has `user:inference` scope, which prevents plugins from loading. The browser OAuth stores full-scope credentials in `~/.claude/.credentials.json`.

After login, set the workspace trust flag so it doesn't prompt on restart:

```python
import json
with open("/home/USER/.claude.json") as f:
    d = json.load(f)
d["projects"]["/home/USER"]["hasTrustDialogAccepted"] = True
with open("/home/USER/.claude.json", "w") as f:
    json.dump(d, f)
```

Tell the user: "Claude is now logged in on your VPS using your subscription. I also told it to trust the workspace so it won't ask again after reboots."

**Step 3: Configure Settings**

Tell the user: "I'm setting up Claude's preferences — telling it which model to use and enabling the Telegram plugin."

Write `~/.claude/settings.json`:

```json
{
  "model": "claude-opus-4-7[1m]",
  "skipDangerousModePermissionPrompt": true,
  "enabledPlugins": {
    "telegram@claude-plugins-official": true
  }
}
```

Note: `claude-opus-4-7[1m]` enables the 1M context window. Without `[1m]`, you get the default context size.

Tell the user: "Done! Claude is configured to use [chosen model] with the Telegram plugin enabled."

**Step 4: Install Telegram Plugin**

Tell the user: "Now I'm installing the Telegram plugin. This is what lets Claude receive and send Telegram messages."

The plugin MUST be installed through Claude's own plugin system — manual file copies don't work (the plugin system validates marketplace origin).

1. Start an interactive Claude session in tmux
2. Type: `/plugin install telegram@claude-plugins-official`
3. Select "Install for you (user scope)"
4. Type: `/reload-plugins`
5. Verify: "1 plugin, 1 plugin MCP server" in the output
6. Exit the session

Tell the user: "Telegram plugin installed and verified! Claude can now talk to Telegram."

**Step 5: Configure Telegram Bot**

Tell the user: "I'm connecting your bot token and locking it to your Telegram ID so only you can use it."

```bash
# Create channel config directory
mkdir -p ~/.claude/channels/telegram

# Write bot token
echo "TELEGRAM_BOT_TOKEN=YOUR_TOKEN_HERE" > ~/.claude/channels/telegram/.env
chmod 600 ~/.claude/channels/telegram/.env

# Write access control (locked to user's Telegram ID)
cat > ~/.claude/channels/telegram/access.json << 'EOF'
{
  "dmPolicy": "allowlist",
  "allowFrom": ["USER_TELEGRAM_ID"],
  "groups": {},
  "pending": {}
}
EOF
```

Tell the user: "Your bot token is saved and access is locked to your Telegram account. Nobody else can use your bot."

**Step 6: Create Startup Script**

Tell the user: "I'm creating a startup script that launches Claude and keeps it healthy. It includes a watchdog that automatically restarts Claude if it ever gets stuck."

Write `/home/USER/claude-telegram.sh`:

```bash
#!/bin/bash
export PATH=/usr/local/bin:$HOME/.npm-global/bin:$HOME/.bun/bin:$HOME/.local/bin:$PATH
export HOME=/home/USER
LOG=$HOME/watchdog.log

# Wait for network
for i in {1..30}; do
    curl -s --max-time 3 https://api.telegram.org > /dev/null 2>&1 && break
    sleep 2
done

# Ensure plugin deps installed (survives reboots)
PLUGIN_DIR=$(python3 -c "
import json
with open('$HOME/.claude/plugins/installed_plugins.json') as f:
    d = json.load(f)
tg = d.get('plugins',{}).get('telegram@claude-plugins-official',[])
if tg: print(tg[0]['installPath'])
" 2>/dev/null)
[ -n "$PLUGIN_DIR" ] && [ -d "$PLUGIN_DIR" ] && cd "$PLUGIN_DIR" && bun install --no-summary 2>/dev/null
cd $HOME

# Launch Claude in background
claude --channels plugin:telegram@claude-plugins-official --dangerously-skip-permissions &
CLAUDE_PID=$!

# Wait for bun MCP server to start
for i in {1..30}; do
    pgrep -f 'bun server.ts' > /dev/null 2>&1 && break
    sleep 1
done

# === 10-SECOND WATCHDOG ===
# Detects polling stalls via TCP connection check (never steals messages).
# Grammy long-polls Telegram API = always has an ESTAB connection.
# Two consecutive failures (20s) confirms a real stall.
STALL_COUNT=0
while kill -0 $CLAUDE_PID 2>/dev/null; do
    sleep 10

    BUN_PID=$(pgrep -f 'bun server.ts' | head -1)
    if [ -z "$BUN_PID" ]; then
        echo "$(date): bun dead" >> $LOG
        kill $CLAUDE_PID 2>/dev/null
        wait $CLAUDE_PID 2>/dev/null
        exit 1
    fi

    ESTAB=$(ss -tnp 2>/dev/null | grep "pid=$BUN_PID" | grep -c ESTAB)
    if [ "$ESTAB" -eq 0 ]; then
        STALL_COUNT=$((STALL_COUNT + 1))
        if [ $STALL_COUNT -ge 2 ]; then
            echo "$(date): polling stall ($STALL_COUNT consecutive), restarting" >> $LOG
            kill $CLAUDE_PID 2>/dev/null
            wait $CLAUDE_PID 2>/dev/null
            exit 1
        fi
    else
        STALL_COUNT=0
    fi
done

echo "$(date): claude exited" >> $LOG
exit 1
```

Make executable: `chmod +x /home/USER/claude-telegram.sh`

Tell the user: "Startup script created. It waits for the internet, installs any missing dependencies, launches Claude, and then watches it every 10 seconds to make sure it's still working. If it ever gets stuck, the watchdog automatically restarts everything."

**Step 7: Create Systemd Services**

Tell the user: "I'm setting up the system services. These make sure Claude starts automatically when the VPS boots up, updates itself daily, and gets a fresh session each morning."

Main service (`/etc/systemd/system/claude-telegram.service`):

```ini
[Unit]
Description=Claude Code Telegram Channel
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
User=USER
Group=USER
WorkingDirectory=/home/USER
Environment=PATH=/usr/local/bin:/home/USER/.npm-global/bin:/home/USER/.bun/bin:/usr/bin:/bin
Environment=HOME=/home/USER
Environment=TERM=xterm-256color
ExecStart=/usr/bin/tmux new-session -d -s telegram -x 120 -y 40 /home/USER/claude-telegram.sh
ExecStop=/usr/bin/tmux kill-session -t telegram
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Daily update (`/etc/systemd/system/claude-update.service` + timer):

```ini
# claude-update.service
[Unit]
Description=Update Claude Code CLI and plugins

[Service]
Type=oneshot
ExecStart=/home/USER/claude-update.sh
```

```ini
# claude-update.timer
[Unit]
Description=Daily Claude Code update

[Timer]
OnCalendar=*-*-* 04:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

Daily session reset (`/etc/systemd/system/claude-session-reset.service` + timer):

```ini
# claude-session-reset.service
[Unit]
Description=Reset Claude Code Telegram session

[Service]
Type=oneshot
ExecStart=/usr/bin/systemctl restart claude-telegram.service
```

```ini
# claude-session-reset.timer
[Unit]
Description=Daily fresh session

[Timer]
OnCalendar=*-*-* 06:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

Update script (`/home/USER/claude-update.sh`):

```bash
#!/bin/bash
export PATH=/usr/local/bin:$HOME/.npm-global/bin:$HOME/.bun/bin:/usr/bin:/bin
export HOME=/home/USER
LOG=$HOME/claude-update.log
echo "$(date): Starting update" >> $LOG
npm install -g @anthropic-ai/claude-code >> $LOG 2>&1
$HOME/.bun/bin/bun upgrade >> $LOG 2>&1
PLUGIN_DIR=$(python3 -c "
import json
with open('$HOME/.claude/plugins/installed_plugins.json') as f:
    d = json.load(f)
tg = d.get('plugins',{}).get('telegram@claude-plugins-official',[])
if tg: print(tg[0]['installPath'])
" 2>/dev/null)
[ -n "$PLUGIN_DIR" ] && cd "$PLUGIN_DIR" && $HOME/.bun/bin/bun install --no-summary >> $LOG 2>&1
echo "$(date): Update complete" >> $LOG
systemctl restart claude-telegram.service
```

Enable everything:

```bash
sudo systemctl daemon-reload
sudo systemctl enable claude-telegram.service claude-update.timer claude-session-reset.timer
sudo systemctl start claude-telegram.service claude-update.timer claude-session-reset.timer
```

Tell the user: "Three things are now set up:
1. **Main service** — Claude starts automatically when the VPS boots and restarts if it crashes
2. **Daily update** — at 4:00 AM, it updates Claude and the plugins to the latest version
3. **Daily session reset** — at 6:00 AM, it restarts Claude for a fresh context window"

**Step 8: Disable Local Telegram Plugin**

Tell the user: "One important step — if you have the Telegram plugin on your computer too, it will fight with the VPS for messages (Telegram only delivers each message to one listener). I'm disabling it on your local machine so the VPS gets all messages."

**CRITICAL:** If Claude Desktop or local Claude Code has the Telegram plugin, it will steal all bot messages (Telegram only delivers to one poller). The local plugin MUST be disabled.

In local `~/.claude/settings.json`, set:
```json
"enabledPlugins": {
  "telegram@claude-plugins-official": false
}
```

Also remove from local `~/.claude/plugins/installed_plugins.json` and delete the cache:
```bash
rm -rf ~/.claude/plugins/cache/claude-plugins-official/telegram
rm -rf ~/.claude/channels/telegram
```

Kill any running local pollers: `pkill -f 'bun.*telegram'`

Tell the user: "Local Telegram plugin disabled. All messages will go to your VPS now."

**Step 9: Test**

Tell the user: "Everything is installed! Let's test it. Open Telegram on your phone and send a message to your bot (@[bot_username]). It should respond within a few seconds."

Also check VPS status:
```bash
ssh USER@VPS_IP "tmux capture-pane -t telegram -p | grep -E 'Listening|telegram'"
```

If the bot responds: "Your bot is running! You can now message it from anywhere — your phone, tablet, or desktop Telegram app. Claude is always on, even when your computer is off."

If the bot doesn't respond within 30 seconds: "It looks like something isn't working yet. Let me diagnose the issue..." Then run through the `fix` argument logic to identify and resolve the problem.

**Step 10: Optional — Memory Sync**

Ask the user: "Would you like to set up memory sync? This keeps your VPS Claude and your local Claude in sync — things either one learns get shared with the other. It's optional but recommended."

If YES, set up bi-directional sync between local Claude memory and VPS:

Create `~/.claude/sync-to-vps.sh` locally:
```bash
#!/bin/bash
VPS="USER@VPS_IP"
KEY="$HOME/.ssh/YOUR_KEY"
LOCAL_MEMORY="$HOME/.claude/projects/-Users-USERNAME-Projects/memory/"
REMOTE_MEMORY="/home/USER/.claude/projects/-home-USER/memory/"
rsync -avz --update -e "ssh -i $KEY" "$LOCAL_MEMORY" "$VPS:$REMOTE_MEMORY" 2>/dev/null
rsync -avz --update -e "ssh -i $KEY" "$VPS:$REMOTE_MEMORY" "$LOCAL_MEMORY" 2>/dev/null
```

Schedule via cron or launchd to run every 15 minutes.

Tell the user: "Memory sync is set up. Every 15 minutes, your local and VPS Claude will exchange memories."

**Step 11: Optional — Additional MCP Servers**

Ask the user: "Do you want to connect any extra tools to your VPS Claude? For example, Home Assistant for smart home control, or other MCP servers?"

If YES, add extra MCP servers by creating a JSON config and passing `--mcp-config`:

```json
{
  "mcpServers": {
    "home-assistant": {
      "command": "/home/USER/.local/bin/uvx",
      "args": ["ha-mcp@latest"],
      "env": {
        "HOMEASSISTANT_URL": "https://your-ha-instance.ui.nabu.casa/",
        "HOMEASSISTANT_TOKEN": "your-long-lived-token"
      }
    }
  }
}
```

Update the startup script's claude command:
```bash
claude --channels plugin:telegram@claude-plugins-official --mcp-config /home/USER/extra-mcp.json --dangerously-skip-permissions
```

---

### `status` — Check VPS Health

SSH into the VPS and report:
1. Service status: `systemctl status claude-telegram`
2. Bot process: `ps aux | grep 'bun.*server'`
3. Model and plan: capture tmux pane, grep for Opus/Sonnet/Max
4. Timer status: `systemctl list-timers | grep claude`
5. Recent watchdog events: `tail -5 ~/watchdog.log`
6. Local telegram pollers: `pgrep -f 'bun.*telegram'` (should be 0)

### `fix` — Diagnose and Repair

Common issues and fixes:

1. **Bot not responding, VPS bun alive** → Stalled polling. Restart service: `sudo systemctl restart claude-telegram.service`

2. **Bot not responding, no bun process** → Plugin deps missing after update/reboot. Install deps and restart:
   ```bash
   cd PLUGIN_DIR && bun install && sudo systemctl restart claude-telegram.service
   ```

3. **Bot not responding, local pollers exist** → Local Desktop stealing updates. Kill local pollers and disable plugin:
   ```bash
   pkill -f 'bun.*telegram'
   # Edit local settings.json: telegram@claude-plugins-official: false
   # Remove from local installed_plugins.json
   # Delete: rm -rf ~/.claude/plugins/cache/claude-plugins-official/telegram
   ```

4. **409 Conflict in logs** → Two pollers fighting. Check both local AND VPS for bun processes. Kill the wrong one.

5. **Auth expired (401 errors)** → Re-authenticate via browser OAuth in tmux session. Never use CLAUDE_CODE_OAUTH_TOKEN env var.

6. **Plugin shows "0 enabled"** → Plugin not installed through Claude's system. Run `/plugin install telegram@claude-plugins-official` in an interactive session.

### `sync` — Force Memory Sync

Run the sync script immediately:
```bash
~/.claude/sync-to-vps.sh
```

Report what was synced.

---

## Known Failure Modes (Hard-Won Lessons)

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| Grammy polling stalls | bun process alive but HTTP long-poll breaks silently | 10s watchdog detects via TCP connection check, auto-restarts |
| Local Desktop steals messages | Telegram delivers to one poller only — local plugin races VPS | Disable local plugin in settings.json, delete cache |
| Plugin "0 enabled" on VPS | Manual file copy doesn't register with plugin system | Must use `/plugin install` in interactive Claude session |
| CLAUDE_CODE_OAUTH_TOKEN breaks plugins | Setup token only has `user:inference` scope, missing `user:mcp_servers` | Don't set the env var; let Claude use stored `.credentials.json` |
| Missing deps after reboot | bun node_modules not persisted or wrong platform | Startup script runs `bun install` before every launch |
| Channel name mismatch | `--plugin-dir` tags plugin as "inline", not matching `plugin:X@marketplace` | Always install via Claude's plugin system, never --plugin-dir |
| Plugin re-enables locally | Claude auto-installs plugins on session start | Set `false` in enabledPlugins + delete cache + delete channels dir |

## Architecture

```
Phone (Telegram)
    |
    v
Telegram Bot API (cloud)
    |
    v (grammy long-polling)
VPS: bun server.ts (MCP server)
    |
    v (stdio/MCP protocol)
VPS: claude (CLI, Max plan, Opus 4.6 1M)
    |
    v (cloud proxy)
claude.ai MCP servers (Stripe, Gmail, Supabase, etc.)
```

Watchdog runs alongside Claude in the same startup script, checking TCP connections every 10 seconds. If grammy's long-poll connection drops, the watchdog kills Claude, systemd restarts everything in ~5 seconds.

