# Getting Started with GoodComms

This guide takes you from download to a running community — whether you're joining someone else's server or setting up your own.

---

## Part 1: The Client (For Users)

### Windows

Two versions are available:

- **Installer (`gc-client_0.9.99_x64-setup.exe`)**: Recommended for most users. Installs the app, adds a desktop shortcut, and handles file associations. To update, just run the new installer.
- **Portable (`.exe`)**: Run without installing — useful for USB drives or restricted environments. To enable Portable Mode, create a folder named `data` in the same directory as the `.exe`. GoodComms will detect it and keep all settings, logs, and cache in that folder instead of AppData.

### Linux

The Linux client is a portable binary — no installation required. Make it executable and run it:

```bash
chmod +x gc-client
./gc-client
```

Screen sharing requires `PipeWire` and `xdg-desktop-portal`. On most modern desktop distros these are already present; on minimal setups you may need to install them.

### Joining a Server

1. Launch the app and click the **+ (Plus)** icon in the sidebar.
2. Enter the address your server owner gave you — a domain (`chat.yourdomain.com`) or an IP address.
3. Register a new account or log in. Accounts are local to each server — there is no global GoodComms account.
4. Enable **Save Password** to get a one-click quick-join button on your next launch.

---

## Part 2: Setting Up a Server (For Owners)

The GoodComms server is a lightweight binary that authenticates sessions, stores messages in SQLite, and relays media packets — it never processes or decodes your data. CPU and RAM requirements are minimal; bandwidth and storage are the real scaling factors.

### Prerequisites

- A Linux server or VPS with **Docker** and **Docker Compose** installed
- A domain pointed at your server
- A reverse proxy — this guide uses [Caddy](https://caddyserver.com), which handles TLS certificate issuance and renewal automatically

> **No domain / LAN setup?** See [Deployment Scenarios: Mode B (Self-Signed)](DEPLOYMENT_SCENARIOS.md) for a simpler standalone setup that doesn't require a proxy or domain.

### Configuration: Using a .env File

Instead of listing every environment variable inline in your compose file, you can put them all in a `.env` file and reference it with one line. A fully commented template is included in the release package as `server.env.example` — copy and edit it:

```bash
cp server.env.example .env
# edit .env with your values
```

Then in your `docker-compose.yml`, replace the `environment:` block with:

```yaml
env_file: .env
```

Both approaches are shown in the sections below — use whichever you prefer.

---

### Step 1: Choose How Your Proxy Reaches GoodComms

How you wire up the proxy depends on whether Caddy runs on the host or inside Docker. Pick the option that matches your setup — the rest of the steps are the same either way.

---

**Option A — Caddy installed on the host (most common for simple VPS setups)**

Expose port 4076 to the host's loopback interface. Your proxy hits it at `localhost:4076`. GoodComms needs no Docker network configuration — it's fully self-contained.

```yaml
# docker-compose.yml
services:
  goodcomms-server:
    image: goodcomms/gc-server:latest
    container_name: gc-server
    restart: unless-stopped
    ports:
      - "127.0.0.1:4076:4076"  # TCP — loopback only, your host proxy connects here
      - "4077:4077/udp"         # Voice (Opus) — must be open in your firewall
      - "4078:4078/udp"         # Video (H.264) — must be open in your firewall
    environment:
      - IP_ADDR=0.0.0.0
      - PORT=4076
      - NO_TLS=true             # The proxy handles TLS — do not remove this
      - TRUST_PROXY=true        # Rate limiting keys on X-Forwarded-For — required behind a proxy (v0.9.99+)
      - DATABASE_PATH=/app/data/goodcomms.db
      - STORAGE_DIR=/app/uploads
      - DRIVE_DIR=/app/drive
      - RETENTION_DAYS=0        # 0 = messages kept forever
      # First run only — remove after logging in:
      # - ADMIN_USER=your_username
      # - ADMIN_PASS=your_secure_password
      - LOG_DIR=/app/logs
    volumes:
      - ./data:/app/data         # Database and certs
      - ./uploads:/app/uploads   # Chat file uploads
      - ./drive:/app/drive       # Server Drive files
      - ./logs:/app/logs         # Server logs
```

```
# Caddyfile
chat.yourdomain.com {
    reverse_proxy localhost:4076
}
```

> **Proxy on a different machine (LAN)?** Replace `127.0.0.1:4076:4076` with `4076:4076` to bind to all interfaces, then point your proxy at this server's LAN IP — e.g. `reverse_proxy 192.168.1.50:4076`. Make sure your firewall blocks port 4076 from the internet — it carries plain HTTP with no TLS.

---

**Option B — Caddy running as a Docker container**

Put both containers on the same Docker network. Caddy resolves `gc-server` by container name — port 4076 does not need to be exposed to the host at all.

Create a shared network if you don't already have one:
```bash
docker network create proxy-network
```

```yaml
# docker-compose.yml
services:
  goodcomms-server:
    image: goodcomms/gc-server:latest
    container_name: gc-server
    restart: unless-stopped
    networks:
      - proxy-network            # Must match the network your Caddy container is on
    ports:
      - "4077:4077/udp"          # Voice (Opus) — must be open in your firewall
      - "4078:4078/udp"          # Video (H.264) — must be open in your firewall
    environment:
      - IP_ADDR=0.0.0.0
      - PORT=4076
      - NO_TLS=true
      - TRUST_PROXY=true         # Required behind a proxy (v0.9.99+)
      - DATABASE_PATH=/app/data/goodcomms.db
      - STORAGE_DIR=/app/uploads
      - DRIVE_DIR=/app/drive
      - RETENTION_DAYS=0
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
    external: true               # Your existing Caddy network
```

```
# Caddyfile
chat.yourdomain.com {
    reverse_proxy gc-server:4076    # Container name resolves on the shared network
}
```

---

If you use **Nginx** or **Traefik** instead of Caddy, the same two options apply — proxy to `localhost:4076` (host proxy) or to `gc-server:4076` (Docker shared network). TLS is always terminated at the proxy; GoodComms always receives plain HTTP on port 4076.

> **Custom ports**: All ports are configurable. If another service already uses 4076, 4077, or 4078, you can change them via `PORT`, `VOICE_PORT`, and `VIDEO_PORT`. When using Docker Compose, update both the environment variable **and** the `ports:` mapping — they must match. The internal `PORT` value is invisible to clients in a reverse proxy setup; only the proxy's public port matters to them.

### Step 2: Open Firewall Ports

Your proxy handles TCP (port 443). Voice and video use direct UDP — they cannot be proxied and must be open at your server firewall and any upstream network firewall (cloud provider security groups, router port forwarding, etc.):

| Port | Protocol | Purpose |
|------|----------|---------|
| 443 | TCP | WebSocket / HTTP (via your proxy) |
| 4077 | UDP | Voice (Opus) |
| 4078 | UDP | Video (H.264) |

> Voice and video will silently fail if UDP 4077/4078 are blocked. This is the most common setup issue.

### Step 3: Bootstrap Your Owner Account

Before first launch, uncomment the `ADMIN_USER` and `ADMIN_PASS` lines in your compose file and fill in your credentials. Then start the server:

```bash
docker compose up -d
```

> **Older systems**: If `docker compose` isn't found, try `docker-compose` (with a hyphen) — the legacy standalone tool. Modern Docker (v20.10+) includes `docker compose` as a built-in plugin.

Open the GoodComms client, connect to your domain, and log in with those credentials. This claims the **Owner** role.

**Immediately after logging in successfully**, stop the server:

```bash
docker compose down
```

Edit your `docker-compose.yml` and remove (or re-comment) the `ADMIN_USER` and `ADMIN_PASS` lines. Then restart:

```bash
docker compose up -d
```

Your account is now stored in the database. Leaving credentials in a config file is a security risk — remove them.

---

## Part 3: Building Your Community

### Creating Channels

In **Admin → Channels**, click **New Channel**.

- **Type**: Toggle **Text**, **Voice**, or both independently. A channel can support simultaneous text and voice.
- **Private**: Hides the channel from everyone except roles you explicitly grant access to.
- **⚠️ Limitation**: A channel's privacy (Public vs. Private) cannot be changed after creation. If you need to change it, delete the channel and create a new one. Name, description, type (text/voice), and position can all be changed after creation.

### Setting Up Roles

In **Admin → Roles**, create roles and assign permissions. Roles use a hierarchy — higher numbers have authority over lower ones.

**Default roles:**

| Role | Hierarchy | Description |
|------|-----------|-------------|
| Owner | 101 | Full control, bypasses all permission checks |
| Administrator | 100 | Broad administrative access |
| Default | 10 | Base role assigned to all new members |

**Example: Setting up a Moderator role**

1. Go to **Admin → Roles** and click **New Role**
2. Name it `Moderator`, set Hierarchy to `50`
3. Grant `manage_messages` and `manage_users`
4. Go to **Admin → Users** and assign the role to a trusted member

The moderator can now delete messages and manage users ranked below them, but cannot touch channels or roles.

### The Server Drive

The **Drive** tab provides private file storage for your community — stored directly on your server hardware, with no third-party cloud involved. Create folders, upload files, and share within your community.

---

## Troubleshooting

**Can't connect to the server**
- Verify your domain resolves to your server's IP (`ping chat.yourdomain.com`)
- Check that your proxy is running and forwarding to port 4076
- Check server logs: `docker compose logs -f goodcomms-server`

**Voice or video not working**
- Almost always a firewall issue — verify UDP 4077 and 4078 (or your custom `VOICE_PORT`/`VIDEO_PORT` if changed) are open at every layer (server firewall, cloud security group, router)
- These ports cannot be proxied; they must be directly reachable

**Self-signed certificate warning**
- Expected when connecting via IP address instead of a domain
- Click **Connect anyway** — this is normal for LAN/IP setups
- See [Deployment Scenarios: Mode B](DEPLOYMENT_SCENARIOS.md) for the full self-signed setup guide

**Second stream not visible to viewers**
- Starting a second stream in the same app session may not work reliably
- Close the client and reopen it before starting a new stream

**GIF search not working**
- GIF search is optional and must be configured by the server owner
- See [GIF_SETUP.md](GIF_SETUP.md) for Giphy and Klipy configuration

**Linux: screen sharing issues**
- Ensure `PipeWire` and `xdg-desktop-portal` are installed and running
- System audio loopback is not supported on Linux
- If the portal picker doesn't appear on a second stream attempt, restart the app

---

## Updating the Server

When a new version is released, updating takes two commands:

```bash
docker compose pull          # Pull the new image
docker compose up -d         # Restart with the new image (zero config changes needed)
```

To pin to a specific version rather than tracking `latest`, set the image tag in your compose file:

```yaml
image: goodcomms/gc-server:0.9.99   # pinned
# or
image: goodcomms/gc-server:latest   # always pulls newest on `docker compose pull`
```

Your database, uploads, and drive files are in the mounted volumes and are untouched by updates.

---

For all deployment configurations (standalone, self-signed, manual TLS, bare binary), see [Deployment Scenarios](DEPLOYMENT_SCENARIOS.md).

For webhook and slash command setup, see [Webhooks & Slash Commands](WEBHOOKS.md).

---

## Getting Help

Found a bug or have a question? Open an issue on GitHub: [github.com/goodcomms/goodcomms/issues](https://github.com/goodcomms/goodcomms/issues)
