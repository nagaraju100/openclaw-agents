# OpenClaw 3-Agent Setup

3 specialized AI agents running on MacBook Air M4 inside Docker containers, each with its own personality (SOUL.md), isolated memory, and connected to Telegram.

## Agents

| Agent | Name | Role | Port | Telegram Bot |
|-------|------|------|------|-------------|
| Scheduler | Sched | Calendar, meetings, daily priorities | 18800 | OpenClawTBot1 |
| Finance | Finchai | Zerodha portfolio, SIPs, NRI tax | 18801 | OpenClawTBot2 |
| News | Newsy | AI news digest, Data Engineering trends | 18802 | OpenClawTBot3 |

## Prerequisites

- macOS (MacBook Air M4 recommended)
- Docker Desktop (latest)
- OpenClaw installed: `npm install -g openclaw`
- Telegram bots created via [@BotFather](https://t.me/BotFather)
- OpenCode Go subscription (get key from [opencode.ai](https://opencode.ai))

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/nagaraju100/openclaw-agents
cd openclaw-agents
```

### 2. Create your `.env` file

```bash
cp .env.example .env
nano .env
```

Fill in your values:

```env
OPENCODE_GO_API_KEY=your-opencode-go-key-here

SCHEDULER_GATEWAY_TOKEN=your-scheduler-token-here
FINANCE_GATEWAY_TOKEN=your-finance-token-here
NEWS_GATEWAY_TOKEN=your-news-token-here

TELEGRAM_BOT_TOKEN_SCHEDULER=your-bot1-token-here
TELEGRAM_BOT_TOKEN_FINANCE=your-bot2-token-here
TELEGRAM_BOT_TOKEN_NEWS=your-bot3-token-here
```

### 3. Create per-agent config

For each agent, create the `.openclaw` directory with your config:

```bash
for agent in scheduler finance news; do
  mkdir -p agents/$agent/.openclaw/agents/main/agent
  mkdir -p agents/$agent/.openclaw/workspace
done
```

Copy your OpenClaw auth files into each agent:

```bash
for agent in scheduler finance news; do
  cp ~/.openclaw/agents/main/agent/auth-profiles.json agents/$agent/.openclaw/agents/main/agent/
  cp ~/.openclaw/agents/main/agent/models.json agents/$agent/.openclaw/agents/main/agent/
done
```

Copy each agent's openclaw.json and SOUL.md:

```bash
for agent in scheduler finance news; do
  cp agents/$agent/openclaw.json agents/$agent/.openclaw/openclaw.json
  cp agents/$agent/SOUL.md agents/$agent/.openclaw/workspace/SOUL.md
done
```

> Update `agents/*/openclaw.json` with your real Telegram bot tokens and gateway tokens before copying.

### 4. Launch all agents

```bash
docker pull ghcr.io/openclaw/openclaw:latest
docker-compose up -d
docker ps  # Should show 3 containers Up
```

### 5. Connect to Web UI

Open each agent's dashboard in your browser:

| Agent | URL |
|-------|-----|
| Scheduler | http://localhost:18800 |
| Finance | http://localhost:18801 |
| News | http://localhost:18802 |

Use the tokenized URL for easy login:
```
http://localhost:18800/#token=YOUR_SCHEDULER_GATEWAY_TOKEN
```

On first connect, approve the device pairing:
```bash
docker exec openclaw-scheduler openclaw devices list
docker exec openclaw-scheduler openclaw devices approve <REQUEST_ID>
```

### 6. Connect Telegram

Send any message to each bot in Telegram. You'll receive a pairing code:

```
OpenClaw: access not configured.
Pairing code: XXXXXXXX
```

Approve it:
```bash
docker exec openclaw-scheduler openclaw pairing approve telegram XXXXXXXX
docker exec openclaw-finance openclaw pairing approve telegram XXXXXXXX
docker exec openclaw-news openclaw pairing approve telegram XXXXXXXX
```

## Daily Management

```bash
# Start all agents
docker-compose up -d

# Stop all agents
docker-compose down

# Restart one agent
docker restart openclaw-scheduler

# View live logs
docker logs -f openclaw-news

# Check all status
docker ps
```

## Project Structure

```
openclaw-agents/
├── docker-compose.yml          # Main config for all 3 agents
├── .env                        # Your secrets (gitignored)
├── .env.example                # Template for .env
└── agents/
    ├── scheduler/
    │   ├── SOUL.md             # Sched personality
    │   ├── openclaw.json       # Agent config template
    │   ├── data/               # Agent memory (gitignored)
    │   └── .openclaw/          # Runtime config (gitignored)
    ├── finance/
    │   ├── SOUL.md             # Finchai personality
    │   ├── openclaw.json
    │   ├── data/
    │   └── .openclaw/
    └── news/
        ├── SOUL.md             # Newsy personality
        ├── openclaw.json
        ├── data/
        └── .openclaw/
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Container exits immediately | Check SOUL.md exists in `.openclaw/workspace/` |
| `HTTP 401: Invalid API key` | Update `auth-profiles.json` with correct OpenCode Go key |
| Telegram conflict (409) | Disable Telegram in local OpenClaw: set `"enabled": false` in `~/.openclaw/openclaw.json` |
| Web UI origin not allowed | Add `http://localhost:PORT` to `controlUi.allowedOrigins` in `openclaw.json` |
| Pairing required | Use tokenized URL: `http://localhost:PORT/#token=YOUR_TOKEN` |

## Credits

Built by [NagarajuAI](https://youtube.com/@NagarajuAI) · April 2026
