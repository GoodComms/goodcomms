# GoodComms Deployment Scenarios (v0.9.99)

GoodComms supports three deployment modes. Choose the one that fits your infrastructure.

## Configuration via .env File

All modes support a `.env` file instead of listing environment variables inline. A fully commented template is provided as `server.env.example` in the release package.

**Docker Compose** — use `env_file:` in place of `environment:`:
```yaml
services:
  goodcomms-server:
    env_file: .env       # reads key=value pairs from .env in the same directory
    ports:
      - "4077:4077/udp"
      - "4078:4078/udp"
```

**systemd** — use `EnvironmentFile=` in the service unit (see Mode D).

**Bare binary (Linux shell)**:
```bash
set -a; source .env; set +a
./gc-server
```

**Bare binary (Windows PowerShell)**:
```powershell
Get-Content .env | Where-Object { $_ -match '^\s*[^#]' } | ForEach-Object {
    $k, $v = $_ -split '=', 2
    [System.Environment]::SetEnvironmentVariable($k.Trim(), $v.Trim())
}
.\gc-server.exe
```

---

---

## Mode A: Reverse Proxy (Recommended)

**Best for:** Production deployments with a domain, using Caddy, Nginx, or Traefik.

**Architecture:**
- TCP 4076: Internal plaintext for proxy to forward (not exposed publicly)
- UDP 4077: Voice (exposed directly for low latency)
- UDP 4078: Video (exposed directly for low latency)

**Docker Compose:**
```yaml
services:
  goodcomms-server:
    image: goodcomms/gc-server:0.9.99
    container_name: gc-server
    restart: unless-stopped
    networks:
      - proxy-network
    ports:
      - "4077:4077/udp"  # Voice
      - "4078:4078/udp"  # Video
      # Note: TCP 4076 is NOT exposed; the proxy reaches it via proxy-network
    environment:
      - IP_ADDR=0.0.0.0
      - PORT=4076          # Internal TCP port for proxy to forward to
      - VOICE_PORT=4077
      - VIDEO_PORT=4078
      - NO_TLS=true        # Proxy handles TLS — critical
      - TRUST_PROXY=true   # Rate limiting keys on X-Forwarded-For — required behind a proxy (v0.9.99+)
      - DATABASE_PATH=/app/data/goodcomms.db
      - STORAGE_DIR=/app/uploads
      - DRIVE_DIR=/app/drive
      - RETENTION_DAYS=0
      # Optional: GIF integration
      # - GIPHY_API_KEY=your_key_here
      # - KLIPY_API_KEY=your_key_here
      # Optional: Admin bootstrap (first run only)
      # - ADMIN_USER=admin
      # - ADMIN_PASS=changeme
    volumes:
      - ./data:/app/data
      - ./uploads:/app/uploads
      - ./drive:/app/drive

networks:
  proxy-network:
    external: true
```

**Caddyfile Example:**
```caddyfile
chat.yourdomain.com {
    reverse_proxy gc-server:4076
}
```

---

## Mode B: Standalone Self-Signed

**Best for:** LAN parties, home servers, or quick testing.

**Architecture:**
- TCP 443: HTTPS (auto-generated self-signed certs)
- TCP 80: HTTP redirect → HTTPS
- UDP 4077: Voice
- UDP 4078: Video

**Docker Compose:**
```yaml
services:
  goodcomms-server:
    image: goodcomms/gc-server:0.9.99
    container_name: gc-server
    restart: unless-stopped
    ports:
      - "443:443"          # HTTPS
      - "80:80"            # HTTP redirect
      - "4077:4077/udp"    # Voice
      - "4078:4078/udp"    # Video
    environment:
      - IP_ADDR=0.0.0.0
      - PORT=443
      - HTTP_PORT=80
      - VOICE_PORT=4077
      - VIDEO_PORT=4078
      # NO_TLS is false by default — self-signed certs auto-generated
      - DATABASE_PATH=/app/data/goodcomms.db
      - STORAGE_DIR=/app/uploads
      - DRIVE_DIR=/app/drive
    volumes:
      - ./data:/app/data   # Also stores auto-generated certs
      - ./uploads:/app/uploads
      - ./drive:/app/drive
```

**Client Behavior:**
- If connecting via IP (e.g., `https://192.168.1.50`): Client **automatically accepts** self-signed certs
- If connecting via domain (e.g., `https://chat.yourdomain.com`): Client **requires valid TLS cert**

---

## Mode C: Standalone Manual TLS

**Best for:** Production with your own certificates (LetsEncrypt, Certbot, etc.).

**Architecture:**
- TCP 443: HTTPS (your certs)
- TCP 80: HTTP → HTTPS redirect
- UDP 4077: Voice
- UDP 4078: Video

**Docker Compose:**
```yaml
services:
  goodcomms-server:
    image: goodcomms/gc-server:0.9.99
    container_name: gc-server
    restart: unless-stopped
    ports:
      - "443:443"          # HTTPS
      - "80:80"            # HTTP redirect
      - "4077:4077/udp"    # Voice
      - "4078:4078/udp"    # Video
    environment:
      - IP_ADDR=0.0.0.0
      - PORT=443
      - HTTP_PORT=80
      - VOICE_PORT=4077
      - VIDEO_PORT=4078
      - TLS_CERT_PATH=/app/certs/fullchain.pem
      - TLS_KEY_PATH=/app/certs/privkey.pem
      - DATABASE_PATH=/app/data/goodcomms.db
      - STORAGE_DIR=/app/uploads
      - DRIVE_DIR=/app/drive
    volumes:
      - ./data:/app/data
      - ./uploads:/app/uploads
      - ./drive:/app/drive
      - ./certs:/app/certs:ro  # Mount your cert files read-only
```

---

## Mode D: Bare Binary (Linux, systemd)

**Best for:** Servers without Docker, or anyone who prefers to manage services directly.

> This mode has not been formally tested beyond local `cargo run` scenarios. If you encounter issues, please report them at [github.com/goodcomms/goodcomms/issues](https://github.com/goodcomms/goodcomms/issues).

**Architecture:** Same as whichever mode fits your setup (A, B, or C) — just without Docker. The binary accepts all the same environment variables and CLI flags.

### Setup

1. Download `gc-server` (Linux) or `gc-server.exe` (Windows) from the release page.
2. Make it executable and move it to a permanent location:
```bash
chmod +x gc-server
sudo mv gc-server /opt/goodcomms/gc-server
```
3. Create the data directories:
```bash
sudo mkdir -p /opt/goodcomms/{data,uploads,drive}
```
4. Create a dedicated system user (recommended):
```bash
sudo useradd --system --no-create-home --shell /bin/false goodcomms
sudo chown -R goodcomms:goodcomms /opt/goodcomms
```

### systemd Service File

Create `/etc/systemd/system/goodcomms.service`.

**Option A — using a `.env` file (recommended, cleaner):**

Copy `server.env.example` to `/opt/goodcomms/.env`, edit it, then reference it with `EnvironmentFile=`:

```ini
[Unit]
Description=GoodComms Server
After=network.target

[Service]
Type=simple
User=goodcomms
WorkingDirectory=/opt/goodcomms
EnvironmentFile=/opt/goodcomms/.env
ExecStart=/opt/goodcomms/gc-server
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**Option B — inline environment variables (no .env file needed):**

```ini
[Unit]
Description=GoodComms Server
After=network.target

[Service]
Type=simple
User=goodcomms
WorkingDirectory=/opt/goodcomms

# Adjust to match your deployment mode
# Behind a reverse proxy (recommended):
Environment="IP_ADDR=0.0.0.0"
Environment="PORT=4076"
Environment="NO_TLS=true"
# Standalone self-signed (no proxy):
# Environment="PORT=443"

Environment="VOICE_PORT=4077"
Environment="VIDEO_PORT=4078"
Environment="DATABASE_PATH=/opt/goodcomms/data/goodcomms.db"
Environment="STORAGE_DIR=/opt/goodcomms/uploads"
Environment="DRIVE_DIR=/opt/goodcomms/drive"
Environment="LOG_DIR=/opt/goodcomms/logs"

# First run only — remove after bootstrapping owner account:
# Environment="ADMIN_USER=your_username"
# Environment="ADMIN_PASS=your_secure_password"

ExecStart=/opt/goodcomms/gc-server
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Starting the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable goodcomms
sudo systemctl start goodcomms
sudo systemctl status goodcomms
```

### Bootstrapping the Owner Account

Uncomment `ADMIN_USER` and `ADMIN_PASS` in the service file, start the server, log in with the client, then:

```bash
sudo systemctl stop goodcomms
# Edit /etc/systemd/system/goodcomms.service — remove ADMIN_USER and ADMIN_PASS
sudo systemctl daemon-reload
sudo systemctl start goodcomms
```

### Viewing Logs

```bash
sudo journalctl -u goodcomms -f
```

### Updating

Stop the service, replace the binary, restart:

```bash
sudo systemctl stop goodcomms
sudo mv gc-server-new /opt/goodcomms/gc-server
sudo chmod +x /opt/goodcomms/gc-server
sudo systemctl start goodcomms
```

### Windows

> The Windows server binary uses the same codebase as the Linux binary and should work correctly, but has not been formally tested as a deployed production server. Use Docker (Linux binary) if you want a well-tested path. Reports welcome at [github.com/goodcomms/goodcomms/issues](https://github.com/goodcomms/goodcomms/issues).

**Running directly (PowerShell — quick testing or LAN use):**

```powershell
$env:IP_ADDR="0.0.0.0"
$env:PORT="4076"
$env:NO_TLS="true"
$env:VOICE_PORT="4077"
$env:VIDEO_PORT="4078"
$env:DATABASE_PATH="C:\goodcomms\data\goodcomms.db"
$env:STORAGE_DIR="C:\goodcomms\uploads"
$env:DRIVE_DIR="C:\goodcomms\drive"
# First run only:
# $env:ADMIN_USER="your_username"
# $env:ADMIN_PASS="your_secure_password"
.\gc-server.exe
```

**Running as a Windows Service (NSSM):**

`gc-server.exe` is not a native Windows service and does not handle service control messages. [NSSM](https://nssm.cc) wraps any executable as a proper Windows service with automatic restart, logging, and start-on-boot.

1. Download NSSM and place `nssm.exe` somewhere on your PATH (e.g. `C:\tools\nssm.exe`).
2. Install the service:
```powershell
nssm install GoodComms "C:\goodcomms\gc-server.exe"
```
3. Set environment variables (run once per variable):
```powershell
nssm set GoodComms AppEnvironmentExtra `
    "IP_ADDR=0.0.0.0" `
    "PORT=4076" `
    "NO_TLS=true" `
    "VOICE_PORT=4077" `
    "VIDEO_PORT=4078" `
    "DATABASE_PATH=C:\goodcomms\data\goodcomms.db" `
    "STORAGE_DIR=C:\goodcomms\uploads" `
    "DRIVE_DIR=C:\goodcomms\drive"
```
4. Set the working directory:
```powershell
nssm set GoodComms AppDirectory "C:\goodcomms"
```
5. Start the service:
```powershell
nssm start GoodComms
```

To bootstrap the owner account, add `ADMIN_USER` and `ADMIN_PASS` to the `AppEnvironmentExtra` line, start the service, log in with the client, then remove them and restart.

**Viewing logs:**
```powershell
# NSSM captures stdout/stderr — configure log paths in NSSM or check:
Get-Content "C:\goodcomms\logs\*.log" -Tail 50
```

**Updating:**
```powershell
nssm stop GoodComms
# Replace gc-server.exe with the new binary
nssm start GoodComms
```

---

## Key Ports Summary

| Port | Protocol | Purpose |
|------|----------|---------|
| 443/4076 | TCP | WebSocket/HTTP signaling |
| 80 | TCP | HTTP → HTTPS redirect |
| 4077 | UDP | Voice (Opus) |
| 4078 | UDP | Video (H.264) |

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | 443 | Main TCP port |
| `NO_TLS` | false | Set to `true` when behind proxy |
| `TRUST_PROXY` | false | Rate limiting/bans key on the first `X-Forwarded-For` hop. Set to `true` behind a reverse proxy (required for Mode A, v0.9.99+); leave off otherwise — the header is forgeable without a proxy |
| `VOICE_PORT` | 4077 | UDP voice port |
| `VIDEO_PORT` | 4078 | UDP video port |
| `DATABASE_PATH` | `data/goodcomms.db` | SQLite database |
| `STORAGE_DIR` | `uploads` | Chat file uploads |
| `DRIVE_DIR` | `drive` | Server drive files |
| `RETENTION_DAYS` | 0 | Auto-delete old messages (0=disabled) |
| `ADMIN_USER` | - | Bootstrap admin username (first run only) |
| `ADMIN_PASS` | - | Bootstrap admin password (first run only) |
| `GIPHY_API_KEY` | - | GIF search integration |
| `KLIPY_API_KEY` | - | GIF search integration (recommended) |

---

**Last Updated**: March 17, 2026
