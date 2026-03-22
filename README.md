This is a Discord music bot that streams **directly** from **Spotify to Discord**. The only interface is Spotify itself.

> **Warning: Authentication Update (2024)**  
> Spotify has deprecated username/password authentication. This fork includes support for **cached credentials** to fix authentication issues. See the [Authentication Setup](#authentication-setup) section below.

**Note**: A Spotify Premium account is currently required. This is a limitation of librespot, the Spotify library used by Aoede. [Facebook login is not supported](https://github.com/librespot-org/librespot/discussions/635).

![Demo](https://raw.githubusercontent.com/codetheweb/aoede/main/.github/demo.gif)

## Use Cases

- Small servers with friends
- Discord Stages, broadcasting music to your audience

## Usage

x86 and arm64 Docker images are provided.
Linux x86_64 binaries and macOS ARM binaries are also available.

### Notes:
⚠️ Aoede only supports bot tokens. Providing a user token will not work.

Aoede will appear offline until you join a voice channel it can access.

### Docker Compose (Recommended):

Docker image sources:
- ghcr.io/foylaou/aoede-foy:latest
- s225002731650/aoede-foy:latest
- `:latest`: Latest version
- `:v0.10.4`: Specific version

```yaml
version: '3.8'

services:
  aoede:
    image: s225002731650/aoede-foy:latest
    container_name: aoede-bot
    restart: unless-stopped
    
    volumes:
      # host_path:container_path
      - /home/Share/aoede-Foy-/aoede-cache:/data

    environment:
      - DISCORD_TOKEN=${DISCORD_TOKEN}
      - DISCORD_USER_ID=${DISCORD_USER_ID}
      - SPOTIFY_DEVICE_NAME=${SPOTIFY_DEVICE_NAME:-Aoede Bot}
      - SPOTIFY_BOT_AUTOPLAY=${SPOTIFY_BOT_AUTOPLAY:-false}
      - CACHE_DIR=/data
    
    # Optional: Logging configuration
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
````

### Pre-built Binaries:

Pre-built binaries are available on the [Releases page](https://github.com/foylaou/aoede-Foy-/releases). Download the binary for your platform, then in your terminal:

# Linux && macOS

```bash
chmod +x aoede-linux-x86_64
DISCORD_TOKEN=your_token \
DISCORD_USER_ID=your_id \
CACHE_DIR=cache \
SPOTIFY_BOT_AUTOPLAY=true \
SPOTIFY_DEVICE_NAME="MUSIC BOT" \
./aoede-linux-x86_64
```

# Windows PowerShell

```shell
$env:DISCORD_TOKEN = ""
$env:SPOTIFY_DEVICE_NAME = ""
$env:DISCORD_USER_ID = 
$env:CACHE_DIR = ""
$env:SPOTIFY_BOT_AUTOPLAY = true
C:\aoede.exe
```

### Building from Source:

Requirements:

* automake
* autoconf
* cmake
* libtool
* Rust
* Cargo

Additionally, the following system libraries are needed (on Debian/Ubuntu):

```bash
sudo apt-get install pkg-config libssl-dev libavahi-compat-libdnssd-dev libasound2-dev cmake
```

Run `cargo build --release`. This will produce a binary at `target/release/aoede`. Set the required environment variables (see the Docker Compose section) and run the binary.

### Configuration Options

#### config.toml (Recommended)

```toml
# Required
discord_token = "your_discord_bot_token"
discord_user_id = 123456789

# Cached credentials directory
cache_dir = "aoede-cache"

# Optional
spotify_bot_autoplay = false
spotify_device_name = "Aoede"
```

#### Environment Variables (Alternative)

| Variable               | Required    | Description                                     |
| ---------------------- | ----------- | ----------------------------------------------- |
| `DISCORD_TOKEN`        | Yes         | Your Discord bot token                          |
| `DISCORD_USER_ID`      | Yes         | The Discord user ID to follow                   |
| `CACHE_DIR`            | Recommended | Directory containing cached Spotify credentials |
| `SPOTIFY_BOT_AUTOPLAY` | No          | Enable autoplay (true/false)                    |
| `SPOTIFY_DEVICE_NAME`  | No          | Custom device name (default: "Aoede")           |

*Username/password are only needed if not using cached credentials. Environment variables override config.toml values.

### Authentication Setup

On first run (or if no cached credentials exist), Aoede will start a Spotify Connect discovery service:

1. Open your Spotify app (phone or desktop)
2. Look for the device name (default: "Aoede") in your device list
3. Select the device to authenticate
4. Credentials will be cached to `{CACHE_DIR}/credentials.json` for future use

### Migrating from Username/Password

If you previously used username/password authentication:

1. Remove `SPOTIFY_USERNAME` and `SPOTIFY_PASSWORD` environment variables
2. Add a `CACHE_DIR` environment variable pointing to your credentials directory
3. On next startup, follow the discovery authentication flow above

### Troubleshooting

* **"Bad credentials" error**: Use cached credentials instead of username/password
* **"No cached credentials found"**: Ensure `credentials.json` is in your cache directory, or let the bot run through the discovery flow
* **Device not showing in Spotify**: Ensure the bot and Spotify are on the same network
* **Credentials expired**: Re-run the credential generation process

## Credits

This project is based on Aoede by codetheweb.
