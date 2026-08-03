# whatsapp-telegram-bridge
# WhatsApp → Telegram Bridge

> A production-ready Node.js service that monitors WhatsApp groups and automatically republishes messages to a Telegram channel — with formatting conversion, media support, and persistent sessions. Runs 24/7, unattended.

![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-blue)
![Deploy](https://img.shields.io/badge/deploy-Railway-0B0D0E?logo=railway&logoColor=white)

## Why this exists

Communities that publish deals, alerts, or announcements often live on WhatsApp — but Telegram channels are far better for broadcast distribution. Copying content between them by hand is slow, error-prone, and impossible to sustain around the clock. This service removes the human from the loop entirely.

## Features

- **Real-time monitoring** of one or more WhatsApp groups
- **Automatic reposting** to a Telegram channel within seconds
- **Formatting conversion** — WhatsApp markdown (`*bold*`, `_italic_`, `~strike~`) → Telegram HTML
- **Media support** — images and captions forwarded intact
- **Persistent sessions** — survives restarts and redeploys; reconnects automatically (no re-scanning QR codes)
- **Pairing-code authentication** — connect your WhatsApp account with a code instead of a QR scan, ideal for headless servers
- **Cloud-native** — designed for Railway (or any Node host) with volume-backed session storage

## How it works

```
┌──────────────────┐     Baileys      ┌──────────────────┐   Bot API    ┌──────────────────┐
│  WhatsApp group   │ ───────────────▶ │  Node.js service  │ ───────────▶ │ Telegram channel  │
│  (source)         │   ws events      │  · filter groups  │  sendMessage │  (destination)    │
└──────────────────┘                  │  · convert format │              └──────────────────┘
                                      │  · handle media   │
                                      │  · persist session│──▶ volume (/data)
                                      └──────────────────┘
```

1. The service connects to WhatsApp via [Baileys](https://github.com/WhiskeySockets/Baileys) using pairing-code auth.
2. Incoming messages are filtered by group ID.
3. Text is converted from WhatsApp markdown to Telegram-safe HTML; media is re-uploaded.
4. The message is published to the Telegram channel via the official Bot API.
5. Session credentials are stored on a mounted volume, so the connection survives restarts.

## Quick start

```bash
git clone https://github.com/YOUR_USERNAME/whatsapp-telegram-bridge.git
cd whatsapp-telegram-bridge
npm install
cp .env.example .env   # fill in your values
npm start
```

On first run, the service prints a **pairing code**. Open WhatsApp on your phone → *Linked devices* → *Link with phone number* → enter the code. Done — the session is saved and future restarts reconnect automatically.

## Configuration

| Variable | Required | Description |
|---|---|---|
| `TELEGRAM_BOT_TOKEN` | ✅ | Token from [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_CHANNEL_ID` | ✅ | Destination channel ID (e.g. `-100xxxxxxxxxx`); the bot must be an admin |
| `WHATSAPP_GROUP_IDS` | ✅ | Comma-separated list of source group JIDs (e.g. `1203...@g.us`) |
| `WHATSAPP_PHONE_NUMBER` | ✅ | Your number in international format, for pairing-code auth |
| `SESSION_DIR` | — | Where session credentials are stored (default: `./data/session`) |
| `LOG_LEVEL` | — | `info` (default) or `debug` |

> **Tip:** to discover a group's JID, set `LOG_LEVEL=debug` and send a message in the group — the JID appears in the logs.

## Deploying on Railway

1. Create a new Railway project from this repo.
2. Add a **Volume** mounted at `/data` and set `SESSION_DIR=/data/session` — this is what makes the session survive redeploys.
3. Set the environment variables above.
4. Deploy. Grab the pairing code from the deploy logs and link your device.

## Project structure

```
src/
├── index.js          # entrypoint — wires everything together
├── whatsapp.js       # Baileys connection, pairing auth, event handling
├── telegram.js       # Telegram Bot API client
├── formatter.js      # WhatsApp markdown → Telegram HTML
└── media.js          # image/caption handling
```

## Roadmap

- [ ] Multiple destination channels with per-group routing
- [ ] Message filtering rules (keywords, senders)
- [ ] Video and document forwarding
- [ ] Health-check endpoint + uptime alerts

## Disclaimer

This project uses Baileys, an unofficial WhatsApp Web library. It is intended for automating **your own** accounts and communities. Use responsibly and review WhatsApp's Terms of Service before deploying.

## License

MIT — see [LICENSE](LICENSE).

---

Built by **Matheus Perez** · Automation Specialist · [LinkedIn](https://linkedin.com/in/matheusperezaks)
