# GoodComms v0.9.97 🚀

**High-performance, self-hosted communication platform built in Rust.**

GoodComms is a lightweight, secure alternative to centralized chat platforms, engineered for speed and stability. Zero cloud dependency — the server is a pure packet relay that does no media processing.

## ✨ Key Features

- **Native Windows GPU Video Pipeline**: 100% native screen sharing using D3D11, MFT hardware H.264 encode/decode, and Windows Graphics Capture. Zero FFmpeg dependency on Windows.
- **GPU-Direct Video Display**: Decoded frames presented via native Win32 HWND + DXGI swap chain — zero CPU readback. Each watched stream opens in its own native window with a media controls toolbar.
- **Simulcast Video SFU**: Server-routed multi-quality video (Source, 1080p, 720p) with on-demand quality switching.
- **FEC Parity Recovery**: XOR-based forward error correction recovers single-fragment loss without retransmission.
- **Binary Protocol V5**: 20-byte binary header system for zero-latency media relay.
- **Iced 0.14 + tiny-skia UI**: Software-rasterised chat UI with no DXGI swap chain — completely decoupled from the video pipeline, no DWM interference.
- **Hybrid Secure Gateway**: Flexible networking supporting Standalone Auto-TLS and Proxy Mode.
- **Unified Port Architecture**: WebSocket and HTTP traffic unified on a single port (Default: 443).
- **Optimized Voice Engine**: Robust AudioSystem with Jitter Buffer, PLC, and Push-To-Talk.

---

## 💬 Message Formatting

GoodComms supports Markdown-style text formatting in chat messages.

**Keyboard shortcuts in the chat input:**
- `Enter` — send message
- `Shift+Enter` — new line (for multi-line messages and code blocks)
- `Ctrl+V` — paste (multi-line text pastes correctly)

### Inline Formatting

| Format | Syntax | Example |
|---|---|---|
| Bold | `**text**` | **text** |
| Italic | `*text*` or `_text_` | *text* |
| Strikethrough | `~~text~~` | ~~text~~ |
| Inline code | `` `code` `` | `code` |

### Code Blocks

Wrap code in triple backticks on their own lines. Optionally add a language name after the opening fence — it appears as a label above the block.

````
```rust
fn hello() {
    println!("Hello!");
}
```
````

````
```python
def greet():
    print("Hello!")
```
````

````
```
plain code block, no language label
```
````

Language labels are display-only (no syntax highlighting yet). Any word works: `rust`, `python`, `js`, `ts`, `go`, `c`, `cpp`, `sql`, `bash`, `json`, `html`, etc.

---

## ⌨️ Slash Commands

GoodComms supports slash commands in chat. Type `/` to see available commands.

### Built-in Commands
- `/me <text>` — Display an action message (italicized)
- `/shrug` — Append ¯\_(ツ)_/¯
- `/tableflip` — (╯°□°）╯︵ ┻━┻
- `/unflip` — ┬─┬ノ( º _ ºノ)

### Custom Commands (Admin)
Admins can create custom slash commands via **Admin → Slash Commands**:

| Type | Description |
|------|-------------|
| Text | Simple text response |
| Webhook | Trigger external HTTP endpoint |
| Action | Server-side actions (kick, ban, timeout, etc.) |

Example: Create a `/serverinfo` command that responds with server info.

---

## 🔐 Roles & Permissions

### Permission Matrix
In **Admin → Channels → Permissions**, each role shows permissions with status indicators:

| Symbol | Meaning |
|--------|---------|
| ● (filled) | Explicitly set for this channel |
| ◐ (half) | Inherited from global role permissions |
| ○ (empty) | Not set (denied) |

**How it works:**
- **Public channels** inherit permissions from the Default role
- **Private channels** inherit from the creator's highest role
- Admins (hierarchy ≥100) bypass all permission checks
- The ●/◐/○ indicators show whether a permission is explicitly set or inherited

### Channel-Specific Permissions
Available permissions for channels:
- `view_channel` — See channel in list
- `send_message` — Post messages
- `join_voice` — Join voice channels
- `speak_voice` — Talk in voice
- `manage_messages` — Pin/delete messages
- `manage_users` — Kick/ban users
- `manage_webhooks` — Create webhooks & slash commands

### Global Permissions
These apply server-wide, not per-channel:
- `manage_channels` — Create/delete channels
- `manage_permissions` — Edit roles
- `admin_access` — Full admin (bypasses all checks)
- `drive_read/write/manage` — Drive access

---

## 🚀 Getting Started

### 🖥️ Client (gc-client)

#### **Windows** (Primary Platform)
No external dependencies required. The Windows client uses 100% native Windows APIs (D3D11/MFT/WGC) for video.

```powershell
cargo build --release -p gc-client
```

#### **Portable Data Mode**
GoodComms supports a fully portable "USB Mode".
1. Create a folder named `data` in the same directory as the `gc-client` executable.
2. Run the client.
3. All configuration, logs, and cache files will be stored inside this `data` folder.

#### **Linux Runtime Dependencies**
The Linux client uses PipeWire (via xdg-desktop-portal) for screen capture and OpenH264 (statically bundled) for video encoding. Audio uses ALSA/PulseAudio via CPAL.

**Arch Linux**:
```bash
sudo pacman -S alsa-lib libpulse pipewire xdg-desktop-portal libx11 libxext libxcb
```

**Ubuntu/Debian/Mint**:
```bash
sudo apt update && sudo apt install -y libasound2 libpulse0 libpipewire-0.3-0 xdg-desktop-portal libx11-6 libxext6 libxcb1
```

**Fedora**:
```bash
sudo dnf install alsa-lib pulseaudio-libs pipewire-libs xdg-desktop-portal libX11 libXext libxcb
```

> **Note:** A running PipeWire session and an xdg-desktop-portal backend (e.g. `xdg-desktop-portal-gnome` or `xdg-desktop-portal-kde`) are required for screen sharing on Linux.

---

### 🌐 Server (gc-server)

The server is a pure relay/SFU — zero media processing. Designed to run on minimal hardware (1 vCPU / 1GB RAM VPS).

#### **Docker Deployment (Recommended)**
```bash
# Pull latest
docker pull goodcomms/gc-server:latest

# Or pin to a specific version
docker pull goodcomms/gc-server:0.9.97
```

**Quick start (Standalone / LAN):**
```yaml
services:
  goodcomms-server:
    image: goodcomms/gc-server:latest
    restart: unless-stopped
    ports:
      - "443:443"
      - "80:80"
      - "4077:4077/udp"
      - "4078:4078/udp"
    environment:
      - ADMIN_USER=your_username
      - ADMIN_PASS=your_secure_password
    volumes:
      - ./data:/app/data
      - ./uploads:/app/uploads
      - ./drive:/app/drive
```

The included `docker-compose.yml` has all three deployment modes (Standalone, Reverse Proxy, Manual TLS). See **[docs/DEPLOYMENT_SCENARIOS.md](docs/DEPLOYMENT_SCENARIOS.md)** for detailed setup guides including Caddy/Nginx reverse proxy configuration.

#### **Execution**
```bash
# Standalone (Fortress Mode)
./gc-server --ip 0.0.0.0 --port 443

# Behind Proxy (Integrator Mode)
./gc-server --ip 127.0.0.1 --port 4076 --no-tls
```

---

## 🛠️ Configuration

### Server Arguments
- `-i, --ip`: Bind IP address (default: `127.0.0.1`)
- `-p, --port`: Main TCP port for WebSocket/HTTP (default: `443`)
- `--http-port`: HTTP redirect port (default: `80`)
- `--no-tls`: Disable internal encryption (use when behind Caddy/Nginx)
- `-v, --voice-port`: UDP Voice port (default: `4077`)
- `--video-port`: UDP Video port (default: `4078`)
- `-d, --storage-dir`: Directory for chat media (default: `uploads`)
- `--db-path`: Path to SQLite database
- `--log-dir`: Directory for rotating log files (default: `logs/`, set empty to disable)
- `--log-max-size-mb`: Max log file size in MB before rotation (default: `50`)
- `--log-max-files`: Number of rotated log files to keep (default: `5`)
- `-a, --admin`: Admin username (set on first run)
- `-w, --password`: Admin password (set on first run)

All arguments can also be set via environment variables.

### Custom Ports Example
You can use any ports you want. Just open/forward them on your router:

```bash
# Example: Custom ports (8443 for web, 5077 voice, 5078 video)
./gc-server -p 8443 -v 5077 --video-port 5078 --no-tls -a admin -w mypassword
```

Or via environment variables:
```bash
PORT=8443 VOICE_PORT=4077 VIDEO_PORT=4078 NO_TLS=true ADMIN_USER=admin ADMIN_PASS=pass ./gc-server
```

Then clients connect to `your-ip:8443`.

### Optional Integrations
- [GIF Setup Guide](docs/GIF_SETUP.md): Enable GIF search (Klipy/Giphy) in chat.

---

## 🛡️ Security & Bootstrapping

GoodComms uses a "Secure-by-Default" philosophy. **There is no default "admin/admin" login.**

1.  **Bootstrapping**: On the first run, provide `ADMIN_USER` and `ADMIN_PASS` via environment variables or CLI arguments. The server will not allow any logins until these are set — restart the server with credentials to enable access.
2.  **TLS Strategy**:
    - **Domains**: Client enforces strict certificate validation.
    - **Raw IPs**: Client allows self-signed certificates for easy LAN/Home hosting.
3.  **Authentication**:
    - JWT-based sessions with `token_version` revocation — logout immediately invalidates reconnection with old tokens.
    - Passwords stored with Argon2 hashing. Credentials never stored in plaintext (OS keyring used on client).
    - 20-hour server-side token refresh keeps long-running sessions (app left open for days) from experiencing stale HTTP tokens.
4.  **Server Hardening** (v0.9.45+):
    - No public unauthenticated routes for user content — all files, avatars, and drive items require a valid JWT.
    - Per-user message rate limiting (4 msg/s, burst 20) enforced server-side.
    - SSRF protection on server-side link preview fetches — private IP ranges blocked.
    - Webhook tokens sent via `Authorization: Bearer` header, not URL parameters.
    - CORS restricted — denies all cross-origin browser requests.
    - Parameterized SQL queries throughout — no format!() string interpolation in DB calls.

---

## ⚠️ Known Limitations

### Linux
| Area | Limitation |
|------|-----------|
| Screen Share — Audio | System audio loopback is **not supported**. Shared screens have no audio. This is a platform limitation of `xdg-desktop-portal`. |
| Screen Share — Window Capture | Sharing an **individual application window** often results in a black screen. Share your **full display** instead. |
| Screen Share — Second Session | On **KDE Plasma**, starting a second screen share in the same app session does not show a source picker. **Restart the app** before sharing again. |
| Video Performance | Encoding and decoding is **software-only** (OpenH264) — significantly higher CPU usage than Windows. |
| Video Viewer | The video viewer window has **no quality selector or volume controls**. Use the voice channel volume settings for audio. |

### All Platforms
| Area | Limitation |
|------|-----------|
| Camera | Camera capture is **not yet implemented**. |
| Screen Share | There is no explicit "Stop Sharing" button. **Close the screen share window** to stop. |
| Code Blocks | Syntax highlighting shows a **language label only** — no colour highlighting. |
| Drive | Folder sizes always display as **0.0 MB** (contents are accessible; size display is a known issue). |

---

**GoodComms v0.9.97** - Engineered for Privacy. Built with Rust.
