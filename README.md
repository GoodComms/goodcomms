# GoodComms v0.9.98

A self-hosted communication platform for communities that value privacy and control. Chat, voice, and screen sharing — native Rust clients, lightweight server, your data on hardware you own.

---

## Why GoodComms?

Most communication platforms — even those that offer "self-hosting" — still tie you to a centralized account system, ship as Electron web apps, or collect data somewhere in the pipeline. GoodComms doesn't do any of that.

- **No cloud accounts.** You register on the server you connect to. That's the only place your account exists — no global GoodComms account, no OAuth, no third-party identity provider.
- **No telemetry.** No analytics, no tracking, no external calls. The only optional network dependency is GIF search, controlled entirely by the server owner.
- **Pure native clients.** Windows and Linux binaries compiled from Rust. No browser engine, no Electron, no runtime. Download and run — the same model as old-school TeamSpeak, before everything moved to the cloud.
- **No surveillance logging.** Server logs contain no IP addresses or account identifiers. Privacy by design.
- **Lightweight server.** The server authenticates sessions, stores messages in SQLite, and routes media packets without ever processing or decoding them. CPU and RAM requirements are minimal; bandwidth and storage are the real scaling factors.

If you have a friend or community member willing to run a server, all your communication stays between you and them — not a corporation in the middle.

---

## Key Features

- **Native Performance**: Chat UI renders on the CPU (tiny-skia). Video pipeline runs on the GPU. Neither interferes with the other.
- **GPU-Accelerated Screen Sharing (Windows)**: Windows Graphics Capture + D3D11 + MFT H.264. Auto-detects NVIDIA, Intel, or AMD hardware. Software fallback on Linux via PipeWire.
- **Crystal Clear Voice**: Opus audio with noise suppression, AGC, push-to-talk, per-user volume, and deafen support.
- **Role-Based Access Control**: Hierarchical role system with channel-level and server-wide permissions. Private channels with explicit access control.
- **Server Drive**: Built-in private file storage running on your own hardware — no third-party cloud.
- **Webhooks & Slash Commands**: Push messages from external services into channels, or trigger external endpoints from chat commands.

---

## For Users — Get Connected

### Step 1: Download and Install

- **Windows (Recommended):** Run `gc-client_0.9.98_x64-setup.exe`. Installs with a desktop shortcut. To update, run the new installer.
- **Windows (Portable):** Run `gc-client.exe` directly. Create a `data/` folder next to the `.exe` to enable Portable Mode — all settings, logs, and cache stay in that folder.
- **Linux:** Portable binary. Requires PipeWire and xdg-desktop-portal for screen sharing.

### Step 2: Add a Server and Log In

1. Launch the app and click the **+** icon in the sidebar. Enter the address your admin gave you (e.g. `chat.example.com` or an IP address) and click **Connect**.
2. Create a new account or log in with existing credentials. Accounts are local to each server — there is no global GoodComms account.
3. Enable **Save Password** to get a one-click quick-join button on your next launch.

### Step 3: Voice Channels

Voice-enabled channels show a **Join Voice** button. Once in voice, right-click any user for per-user volume controls. Your own mic mute and deafen are at the top of the app.

### Step 4: Screen Sharing

Click **Go Live** in the bottom left and choose a window or display. Your stream attaches to the channel you had open. To watch someone else's stream, open their channel and click **Watch** next to their name.

### Chat Formatting

| Format | Syntax |
|--------|--------|
| Bold | `**text**` |
| Italic | `*text*` |
| Strikethrough | `~~text~~` |
| Inline code | `` `code` `` |
| Code block | ` ```code``` ` |
| Slash commands | `/me`, `/shrug`, `/tableflip` |

---

## For Server Owners

### Quick Start — Recommended Setup (Reverse Proxy)

The recommended production setup uses a reverse proxy to handle TLS. GoodComms listens on port 4076; your proxy (Caddy, Nginx, Traefik) forwards HTTPS traffic to it.

How your proxy reaches GoodComms depends on where your proxy runs:

**Option A — Proxy on the host (Caddy installed directly on the server)**

Expose port 4076 to the host's loopback interface. No shared Docker network needed — GoodComms is self-contained and your proxy hits it via `localhost`.

```yaml
services:
  goodcomms-server:
    image: goodcomms/gc-server:latest
    container_name: gc-server
    restart: unless-stopped
    ports:
      - "127.0.0.1:4076:4076"  # TCP — host-only, your proxy connects here
      - "4077:4077/udp"         # Voice (Opus)
      - "4078:4078/udp"         # Video (H.264)
    environment:
      - IP_ADDR=0.0.0.0
      - PORT=4076
      - NO_TLS=true
      - DATABASE_PATH=/app/data/goodcomms.db
      - STORAGE_DIR=/app/uploads
      - DRIVE_DIR=/app/drive
      # First run only — remove after logging in:
      # - ADMIN_USER=your_username
      # - ADMIN_PASS=your_secure_password
      - LOG_DIR=/app/logs
    volumes:
      - ./data:/app/data
      - ./uploads:/app/uploads
      - ./drive:/app/drive
      - ./logs:/app/logs
```

```
# Caddyfile
chat.yourdomain.com {
    reverse_proxy localhost:4076
}
```

> **Proxy on a different LAN machine?** Replace `127.0.0.1:4076:4076` with `4076:4076` to bind to all interfaces, then point your proxy at this server's LAN IP (e.g. `reverse_proxy 192.168.1.50:4076`). Ensure your firewall blocks port 4076 from the internet — it's plain HTTP.

**Option B — Proxy in Docker (Caddy running as a container)**

Add both containers to the same Docker network. Caddy reaches GoodComms by container name — no port needs to be exposed to the host at all.

```yaml
services:
  goodcomms-server:
    image: goodcomms/gc-server:latest
    container_name: gc-server
    restart: unless-stopped
    networks:
      - proxy-network        # Must match the network your Caddy container is on
    ports:
      - "4077:4077/udp"      # Voice (Opus)
      - "4078:4078/udp"      # Video (H.264)
    environment:
      - IP_ADDR=0.0.0.0
      - PORT=4076
      - NO_TLS=true
      - DATABASE_PATH=/app/data/goodcomms.db
      - STORAGE_DIR=/app/uploads
      - DRIVE_DIR=/app/drive
      # First run only — remove after logging in:
      # - ADMIN_USER=your_username
      # - ADMIN_PASS=your_secure_password
      - LOG_DIR=/app/logs
    volumes:
      - ./data:/app/data
      - ./uploads:/app/uploads
      - ./drive:/app/drive
      - ./logs:/app/logs

networks:
  proxy-network:
    external: true            # Your existing Caddy network
```

```
# Caddyfile
chat.yourdomain.com {
    reverse_proxy gc-server:4076    # Container name resolves on the shared network
}
```

> For standalone (no domain / LAN) and manual TLS setups, see [Deployment Scenarios](docs/DEPLOYMENT_SCENARIOS.md).

### First-Time Setup (Owner Bootstrap)

1. Uncomment `ADMIN_USER` and `ADMIN_PASS` in your compose file and set your credentials before first launch.
2. Start the server: `docker compose up -d`
3. Connect with the client and log in to claim **Owner** status.
4. **Security**: Stop the server (`docker compose down`), remove the `ADMIN_USER` and `ADMIN_PASS` lines, then restart (`docker compose up -d`). Your account is now in the database — credentials in a config file are a security risk.

> **Firewall**: UDP ports 4077 and 4078 must be open. TCP is handled by your proxy. Without the UDP ports, voice and video will not work.

### Configuration Reference

All options can be set via CLI flag or environment variable. Environment variables take precedence.

| Feature | CLI Flag | Env Variable | Default |
| :--- | :--- | :--- | :--- |
| Bind IP | `-i`, `--ip` | `IP_ADDR` | `127.0.0.1` |
| Main Port (TCP) | `-p`, `--port` | `PORT` | `443` |
| HTTP Port | `--http-port` | `HTTP_PORT` | `80` |
| Admin User | `-a`, `--admin` | `ADMIN_USER` | *(first run only)* |
| Admin Password | `-w`, `--password` | `ADMIN_PASS` | *(first run only)* |
| Voice Port (UDP) | `-v`, `--voice-port` | `VOICE_PORT` | `4077` |
| Video Port (UDP) | `--video-port` | `VIDEO_PORT` | `4078` |
| Database Path | `--db-path` | `DATABASE_PATH` | `goodcomms.db` |
| File Upload Dir | — | `STORAGE_DIR` | `uploads` |
| Server Drive Dir | — | `DRIVE_DIR` | `drive` |
| Message Retention | — | `RETENTION_DAYS` | `0` *(disabled)* |
| TLS Certificate | — | `TLS_CERT_PATH` | *(auto self-signed)* |
| TLS Private Key | — | `TLS_KEY_PATH` | *(auto self-signed)* |
| Disable TLS | `--no-tls` | `NO_TLS` | `false` |
| Log Directory | `--log-dir` | `LOG_DIR` | `logs` |

All ports are fully configurable. When using Docker Compose, changing a port requires updating **both** the environment variable **and** the `ports:` mapping — they must match. For example, to run the voice relay on port 5000:

```yaml
environment:
  - VOICE_PORT=5000
ports:
  - "5000:5000/udp"   # was 4077:4077/udp
```

This applies to `PORT`, `VOICE_PORT`, and `VIDEO_PORT`. The internal `PORT` value is particularly useful if another service already occupies 4076. Note that for reverse proxy setups, the internal `PORT` is invisible to clients — only the proxy's public-facing port matters to them.

**CLI example:**
```bash
./gc-server --port 8443 --admin myname --password mysecret --no-tls
```

### Administration

#### Roles and Permissions

GoodComms uses a hierarchical role system. Roles with a higher Hierarchy number have authority over lower ones.

Default roles:
- **Owner (Hierarchy 101)**: Bypasses all permission checks.
- **Administrator (Hierarchy 100)**: Broad administrative access.
- **Default (Hierarchy 10)**: Base role for all new members.

Permissions span channel-level controls (view, send, voice, manage messages, manage users, webhooks) and server-wide controls (manage channels, manage roles, drive read/write/manage). See [GETTING_STARTED.md](docs/GETTING_STARTED.md) for a moderator role walkthrough.

#### Security Hardening

- **Bootstrapping**: No default passwords — owner credentials must be set explicitly and removed after first login.
- **Revocation**: JWT sessions are invalidated immediately on logout or credential change.
- **Privacy**: All media, avatars, and drive files require a valid authentication token — no public routes.
- **Hardening**: Rate limiting, SSRF protection for link previews, parameterized queries throughout.

---

## Known Limitations

### All Platforms
- **Screen sharing**: Starting a second stream in the same app session may not be visible to viewers. Restart the client before streaming again if this occurs.
- **Privacy Settings**: A channel cannot be switched between Public and Private after it is created. If you need to change a channel's privacy, please delete it and create a new one.

### Linux
- Screen sharing: System audio loopback is not supported.
- Audio: Per-user volume sliders can only lower volume, not boost.

---

## Technical Specifications

- **Clients**: Native Windows and Linux binaries (Rust). Chat UI: tiny-skia (CPU). Video pipeline: Direct3D 11 + Windows Graphics Capture + MFT H.264 (GPU, Windows); PipeWire + OpenH264 (software, Linux).
- **Server**: Single Rust binary. Axum HTTP/WebSocket + UDP relay. SQLite via sqlx. Pure packet relay — zero media processing server-side.
- **Video**: Simulcast SFU with FEC parity recovery for single-fragment packet loss. Quality tiers: Source / 1080p / 720p.
- **Audio**: Opus codec. Jitter buffer, noise suppression (nnnoiseless), AGC, push-to-talk.
- **Protocol**: V5 binary header (20 bytes) for low-latency media relay.

---

## Further Reading

- [Getting Started](docs/GETTING_STARTED.md) — detailed walkthrough for users and server owners
- [Deployment Scenarios](docs/DEPLOYMENT_SCENARIOS.md) — reverse proxy, self-signed TLS, and manual TLS
- [Webhooks & Slash Commands](docs/WEBHOOKS.md) — integrating bots and external services
- [GIF Search Setup](docs/GIF_SETUP.md) — optional Giphy/Klipy configuration

---

## Getting Help

Found a bug or have a question? Open an issue: [github.com/goodcomms/goodcomms/issues](https://github.com/goodcomms/goodcomms/issues)

---

*GoodComms v0.9.98 — Engineered for Privacy. Built with Rust.*
