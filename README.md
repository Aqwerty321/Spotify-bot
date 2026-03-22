# Spotify Bot

A Discord music bot that streams **directly** from **Spotify to Discord**. The only interface is Spotify itself. You control playback entirely through the Spotify app.

**Note**: A Spotify Premium account is required. This is a limitation of [librespot](https://github.com/librespot-org/librespot), the Spotify library this bot uses.

![Demo](https://raw.githubusercontent.com/codetheweb/aoede/main/.github/demo.gif)

## How It Works

1. The bot follows a specific Discord user (you)
2. When you join a voice channel, the bot enables Spotify Connect
3. A device (e.g. "Aoede") appears in your Spotify app's device list
4. You select it and play music -- audio streams directly to the voice channel
5. When you leave the voice channel, the bot disconnects and the device disappears

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose
- A Discord bot token ([how to create one](#1-create-a-discord-bot))
- A Spotify Premium account
- `avahi-daemon` installed and running on the host (for Spotify Connect discovery)

## Setup

### 1. Create a Discord Bot

1. Go to https://discord.com/developers/applications
2. Click **New Application**, give it a name
3. Go to **Bot** tab, click **Reset Token**, and copy the token
4. Go to **OAuth2 > URL Generator**
   - Under Scopes, check **bot**
   - Under Permissions, check **Connect**, **Speak**, **Use Voice Activity**
5. Open the generated URL in your browser and invite the bot to your server

### 2. Get Your Discord User ID

1. In Discord, go to **Settings > Advanced** and enable **Developer Mode**
2. Right-click your own username and click **Copy User ID**

### 3. Install Avahi (if not already installed)

The bot uses mDNS for Spotify Connect discovery. This requires the Avahi daemon on the host:

```bash
sudo apt-get install -y avahi-daemon
sudo systemctl enable --now avahi-daemon
```

### 4. Clone and Configure

```bash
git clone https://github.com/Aqwerty321/Spotify-bot.git
cd Spotify-bot
mkdir aoede-cache
```

Create a `.env` file:

```
DISCORD_TOKEN=your_bot_token_here
DISCORD_USER_ID=your_user_id_here
CACHE_DIR=/data
SPOTIFY_BOT_AUTOPLAY=true
SPOTIFY_DEVICE_NAME=Aoede
```

| Variable | Required | Description |
|----------|----------|-------------|
| `DISCORD_TOKEN` | Yes | Your Discord bot token |
| `DISCORD_USER_ID` | Yes | Your Discord user ID -- the bot follows this user |
| `CACHE_DIR` | Yes | Path inside the container for cached credentials (use `/data`) |
| `SPOTIFY_BOT_AUTOPLAY` | No | Auto-play similar tracks when queue ends (default: `false`) |
| `SPOTIFY_DEVICE_NAME` | No | Name shown in Spotify's device list (default: `Aoede`) |

### 5. Build and Start

The bot must be built from source to include DAVE (Discord Audio Voice Encryption) support:

```bash
docker compose build    # ~5-15 min first time
docker compose up
```

### 6. First-Time Spotify Authentication

On first run, there are no cached Spotify credentials. The bot will start a Spotify Connect discovery service. You'll see output like:

```
No cached credentials found, re-authentication required
Starting Discovery service...
Device name: Aoede
Waiting for Spotify app to connect...
```

To authenticate:

1. Open **Spotify** on your phone or desktop
2. Tap the **device picker** (speaker icon at the bottom)
3. Look for your device name (e.g. **"Aoede"**) and select it
4. The bot authenticates and saves credentials to `aoede-cache/credentials.json`

You'll see confirmation:

```
Credentials verified!
Authentication complete!
```

Future restarts use the cached credentials automatically -- no re-authentication needed.

### 7. Run in Background

Once authentication is complete, restart the bot in detached mode:

```bash
docker compose up -d
```

The bot will:
- Run in the background (no terminal needed)
- Auto-restart on crash (`restart: unless-stopped`)
- Auto-start when Docker/your machine boots

### Managing the Bot

```bash
docker compose logs -f      # watch live logs
docker compose restart      # restart the bot
docker compose down         # stop the bot
docker compose build        # rebuild after code changes
```

## What Happens at Runtime

1. **Bot starts** and connects to Discord using your bot token
2. **Bot appears offline** in Discord until the followed user joins a voice channel
3. **User joins voice** -- bot enables Spotify Connect, device appears in Spotify
4. **User selects device in Spotify** -- Spotify Connect session established
5. **User plays music** -- bot joins the voice channel and streams audio
6. **User changes tracks** -- audio switches seamlessly
7. **User leaves voice** -- bot disconnects from voice and disables Spotify Connect

## Building from Source (without Docker)

Requirements: Rust 1.85+, automake, autoconf, cmake, libtool

```bash
sudo apt-get install pkg-config libssl-dev libavahi-compat-libdnssd-dev libasound2-dev cmake
cargo build --release
```

Set the environment variables from the table above and run `./target/release/aoede`.

## Troubleshooting

- **Bot joins but no audio**: Make sure you built from source (`docker compose build`). Pre-built images may not include DAVE (E2EE) support, which Discord requires for voice.
- **"DNS-SD Error: Unknown"**: Avahi daemon is not running. Install and start it: `sudo systemctl start avahi-daemon`
- **"Permission denied" on credentials**: Make sure `aoede-cache/` is owned by uid 1000: `chown -R 1000:1000 aoede-cache/`
- **Device not showing in Spotify**: The bot and your Spotify app must be on the same network. Check that avahi-daemon is running and the container has the D-Bus/Avahi socket mounts.
- **"Bad credentials" error**: Delete `aoede-cache/credentials.json` and re-run the discovery flow.
- **Credentials expired**: Delete `aoede-cache/credentials.json` and restart the bot to re-authenticate.

## Credits

Based on [Aoede](https://github.com/codetheweb/aoede) by codetheweb, with the [foylaou/aoede-Foy-](https://github.com/foylaou/aoede-Foy-) fork.
