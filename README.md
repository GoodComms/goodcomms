# GoodComms v0.9.97

High-performance, self-hosted communication platform built in Rust.

GoodComms is a lightweight, secure alternative to centralized chat platforms. It is engineered for speed and stability with zero cloud dependencies. The server acts as a pure packet relay, ensuring your data remains private and under your control.

## Key Features

- Built for Speed: Native Windows and Linux applications that respect system resources. No web-browser bloat.
- High-Quality Video: Smooth, low-latency screen sharing designed for high-resolution displays.
- Privacy First: Self-host your own Private Fortress. Your messages, files, and voice data stay on your hardware.
- Crystal Clear Voice: Robust audio engine with push-to-talk and noise suppression support.
- Modern Chat Experience: Full Markdown support, code blocks with language labels, and interactive slash commands.
- Sovereign Data: Built-in Server Drive for private file storage and sharing directly from your own server.

## Message Formatting

GoodComms supports Markdown-style text formatting in chat messages.

Keyboard shortcuts in the chat input:
- Enter: send message
- Shift+Enter: new line (for multi-line messages and code blocks)
- Ctrl+V: paste (multi-line text pastes correctly)

### Inline Formatting

| Format | Syntax | Example |
|---|---|---|
| Bold | **text** | **text** |
| Italic | *text* | *text* |
| Strikethrough | ~~text~~ | ~~text~~ |
| Inline code | `code` | `code` |

### Code Blocks

Wrap code in triple backticks on their own lines. Optionally add a language name after the opening fence to display a label above the block.

Example:
```rust
fn hello() {
    println!("Hello!");
}
```

## Slash Commands

GoodComms supports slash commands in chat. Type / to see available commands.

### Built-in Commands
- /me <text>: Display an action message (italicized)
- /shrug: Append ¯\_(ツ)_/¯
- /tableflip: (╯°□°）╯︵ ┻━┻
- /unflip: ┬─┬ノ( º _ ºノ)

### Custom Commands (Admin)
Admins can create custom slash commands via Admin -> Slash Commands for text responses, webhooks, or server-side actions.

## Roles and Permissions

### Permission Matrix
In Admin -> Channels -> Permissions, permissions are shown with status indicators:
- Filled: Explicitly set for this channel
- Half: Inherited from global role permissions
- Empty: Not set (denied)

### Channel-Specific Permissions
- view_channel: See channel in list
- send_message: Post messages
- join_voice: Join voice channels
- speak_voice: Talk in voice
- manage_messages: Pin/delete messages
- manage_users: Kick/ban users
- manage_webhooks: Create webhooks and slash commands

## Getting Started

### Client (gc-client)

#### Windows
The Windows client uses native APIs for optimal performance. No external dependencies are required for standard operation.

#### Portable Data Mode
1. Create a folder named "data" in the same directory as the executable.
2. All configuration, logs, and cache files will be stored inside this folder.

#### Linux Runtime Dependencies
The Linux client requires PipeWire and xdg-desktop-portal for screen capture. Ensure your distribution has these components installed and running.

### Server (gc-server)

The server is a lightweight relay designed to run on minimal hardware.

#### Execution
- Standalone: ./gc-server --ip 0.0.0.0 --port 443
- Behind Proxy: ./gc-server --ip 127.0.0.1 --port 4076 --no-tls

## Configuration

### Server Arguments
- -i, --ip: Bind IP address
- -p, --port: Main TCP port for WebSocket/HTTP (default: 443)
- --no-tls: Disable internal encryption (use when behind a reverse proxy)
- -v, --voice-port: UDP Voice port (default: 4077)
- --video-port: UDP Video port (default: 4078)
- -d, --storage-dir: Directory for chat media
- -a, --admin: Admin username (set on first run)
- -w, --password: Admin password (set on first run)

All arguments can also be set via environment variables.

## Security

GoodComms uses a secure-by-default philosophy.
- Bootstrapping: You must set an admin username and password on the first run.
- Authentication: JWT-based sessions with immediate revocation on logout.
- Encryption: Passwords hashed with Argon2. Secure TLS used for all communications.
- Hardening: Rate limiting, SSRF protection, and parameterized SQL queries are used throughout the system.

## Known Limitations

### Linux
- System audio loopback is not supported for screen sharing.
- Sharing individual windows may result in a black screen; use full display share.
- Software-based video encoding/decoding leads to higher CPU usage than Windows.

### All Platforms
- Camera capture is not yet implemented.
- Folder sizes in the Drive display as 0.0 MB.

GoodComms v0.9.97 - Engineered for Privacy. Built with Rust.

## Technical Specifications

- Native Windows GPU Video Pipeline: Uses D3D11, MFT hardware H.264, and Windows Graphics Capture.
- GPU-Direct Video Display: Frames presented via Win32 HWND and DXGI swap chain.
- Simulcast Video SFU: Server-routed multi-quality video streams.
- FEC Parity Recovery: XOR-based forward error correction for packet loss recovery.
- Binary Protocol V5: Optimized header system for low-latency media relay.
