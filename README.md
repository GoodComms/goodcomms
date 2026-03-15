# GoodComms v0.9.97

High-performance, self-hosted communication platform built in Rust.

GoodComms is a lightweight, secure alternative to centralized chat platforms. It is engineered for speed and stability with zero cloud dependencies and zero telemetry. The server acts as a pure packet relay, ensuring your data remains private and under your control.

## Key Features

- Built for Speed: Native Windows and Linux applications that respect system resources. No web-browser bloat.
- High-Quality Video: Smooth, low-latency screen sharing designed for high-resolution displays.
- Privacy First: 100% Private. No telemetry, no tracking, and no cloud dependencies (optional GIF search is controlled by the server owner).
- Crystal Clear Voice: Robust audio engine with push-to-talk and noise suppression support.
- Modern Chat Experience: Full Markdown support, code blocks with language labels, and interactive slash commands.
- Sovereign Data: Built-in Server Drive for private file storage and sharing directly from your own server.

## Getting Started: Client

The GoodComms client is available for Windows and Linux.

### Windows: Installer vs. Portable
- Windows Installer (.exe setup): The standard way to install the app. It handles file placement and creates desktop shortcuts. Note: Updates are manual; to update, simply run the installer for the new version.
- Portable Mode (.exe): For users who want to run GoodComms without installation.
  - To enable Portable Mode, create a folder named "data" in the same directory as the executable.
  - All configuration, logs, and cache files will be stored inside this folder.

### Linux
The Linux client is provided as a portable binary. It requires PipeWire and xdg-desktop-portal for screen capture. Ensure these are installed and running on your distribution.

### Joining a Server
1. Launch the client and click the plus (+) icon in the left sidebar.
2. Enter the server address (domain or IP).
3. If it is your first time on that server, follow the prompts to register a new account. Your credentials are specific to that server only.

## Getting Started: Server

The GoodComms server is a lightweight relay designed to run on minimal hardware (1 vCPU / 1GB RAM).

### Quick Start with Docker (Recommended)
The fastest way to get a server running is using Docker Compose:

```yaml
services:
  goodcomms-server:
    image: goodcomms/gc-server:latest
    restart: unless-stopped
    ports:
      - "443:443"
      - "80:80"
      - "4077-4078:4077-4078/udp"
    environment:
      - ADMIN_USER=your_username
      - ADMIN_PASS=your_secure_password
    volumes:
      - ./data:/app/data
      - ./uploads:/app/uploads
      - ./drive:/app/drive
```

### Administrative Setup (Owner Flow)
1. Set your `ADMIN_USER` and `ADMIN_PASS` environment variables (or CLI arguments) before starting the server for the first time.
2. Once the server is running, connect using the client and log in with these credentials to claim "Owner" status.
3. **Security Note**: After the first successful login, it is recommended to remove the `ADMIN_USER` and `ADMIN_PASS` variables from your configuration/docker-compose file to prevent accidental credential exposure.
4. Open the "Admin" panel in the client to manage channels and roles.

## Message Formatting

GoodComms supports Markdown-style text formatting:
- Bold: **text**
- Italic: *text*
- Strikethrough: ~~text~~
- Inline code: `code`
- Code Blocks: Wrap code in triple backticks on their own lines.

Keyboard shortcuts:
- Enter: Send message.
- Shift+Enter: New line.
- Ctrl+V: Paste text or images.

## Slash Commands
Type / in the chat input to see available commands:
- /me <text>: Action message (italicized).
- /shrug: Append ¯\_(ツ)_/¯.
- /tableflip: (╯°□°）╯︵ ┻━┻.

## Configuration & Security

### Server Arguments
- -i, --ip: Bind IP address.
- -p, --port: Main TCP port (default: 443).
- -v, --voice-port: UDP Voice port (default: 4077).
- --video-port: UDP Video port (default: 4078).

### Security Features
- Secure-by-Default: No default passwords; admin must be set on first run.
- Authentication: JWT-based sessions with immediate revocation on logout.
- Encryption: Passwords hashed with Argon2; TLS used for all communications.
- Hardening: Rate limiting, SSRF protection, and parameterized SQL queries.

## Documentation
For more detailed information, see the files in the docs/ directory:
- GETTING_STARTED.md: A more in-depth guide for new users and admins.
- DEPLOYMENT_SCENARIOS.md: Advanced server setups (Caddy, Nginx, etc.).
- GIF_SETUP.md: Enabling GIF search in chat.

## Known Limitations
- Linux: System audio loopback is not supported; software-only video encoding.
- Camera: Camera capture is not yet implemented.
- Drive: Folder sizes display as 0.0 MB.

GoodComms v0.9.97 - Engineered for Privacy. Built with Rust.

## Technical Specifications
- Native Windows GPU Video Pipeline: Uses D3D11, MFT hardware H.264, and Windows Graphics Capture.
- GPU-Direct Video Display: Frames presented via Win32 HWND and DXGI swap chain.
- Simulcast Video SFU: Server-routed multi-quality video streams.
- FEC Parity Recovery: XOR-based forward error correction for packet loss recovery.
- Binary Protocol V5: Optimized header system for low-latency media relay.
