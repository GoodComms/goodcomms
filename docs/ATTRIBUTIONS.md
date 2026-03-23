# Third-Party Attributions

GoodComms is built on a foundation of high-quality open-source libraries. We are grateful to every maintainer and contributor listed here.

---

## Core Frameworks

### [Iced](https://github.com/iced-rs/iced)
- **License:** MIT
- **Role:** Cross-platform GUI framework powering the GoodComms client.

### [Tokio](https://github.com/tokio-rs/tokio)
- **License:** MIT
- **Role:** Async runtime for all network I/O across client and server.

### [Axum](https://github.com/tokio-rs/axum)
- **License:** MIT
- **Role:** HTTP and WebSocket server framework.

### [SQLx](https://github.com/launchbadge/sqlx)
- **License:** MIT / Apache 2.0
- **Role:** Async, compile-time-checked SQL for all database persistence (SQLite).

### [Serde](https://github.com/serde-rs/serde) + [serde_json](https://github.com/serde-rs/json)
- **License:** MIT / Apache 2.0
- **Role:** Serialization and deserialization of the WebSocket protocol and configuration.

---

## Audio

### [Opus](https://opus-codec.org/) via [audiopus](https://github.com/nickelc/audiopus)
- **License:** BSD-3-Clause (Opus codec) / MIT (audiopus bindings)
- **Role:** Low-latency, high-fidelity audio codec for voice channels and stream audio.

### [cpal](https://github.com/RustAudio/cpal)
- **License:** Apache 2.0
- **Role:** Cross-platform audio I/O — capture and playback for voice channels.

### [nnnoiseless](https://github.com/jneem/nnnoiseless)
- **License:** MIT
- **Role:** RNNoise-based real-time noise suppression applied to microphone input.

---

## Video

### [OpenH264](https://github.com/cisco/openh264) via [openh264](https://crates.io/crates/openh264)
- **License:** BSD-2-Clause
- **Role:** Software H.264 encoder/decoder used for video streaming on Linux.

### [windows](https://github.com/microsoft/windows-rs) crate
- **License:** MIT / Apache 2.0
- **Role:** Windows platform bindings for Direct3D 11, Media Foundation (MFT H.264 hardware encode/decode), Windows Graphics Capture (WGC), DXGI swap chains, and WASAPI audio loopback.

### [nokhwa](https://github.com/l1npengtul/nokhwa)
- **License:** Apache 2.0
- **Role:** Cross-platform camera capture (stub, pending full implementation).

---

## Network & Security

### [tokio-tungstenite](https://github.com/snapview/tokio-tungstenite)
- **License:** MIT
- **Role:** Async WebSocket implementation for both client and server connections.

### [rustls](https://github.com/rustls/rustls)
- **License:** MIT / Apache 2.0 / ISC
- **Role:** Memory-safe TLS implementation — no OpenSSL dependency.

### [reqwest](https://github.com/seanmonstar/reqwest)
- **License:** MIT / Apache 2.0
- **Role:** HTTP client used for link preview fetching and webhook delivery.

### [jsonwebtoken](https://github.com/Keats/jsonwebtoken)
- **License:** MIT
- **Role:** JWT generation and validation for session authentication.

### [argon2](https://github.com/RustCrypto/password-hashes/tree/master/argon2)
- **License:** MIT / Apache 2.0
- **Role:** Password hashing for user credentials and webhook tokens.

### [rcgen](https://github.com/rustls/rcgen)
- **License:** MIT / Apache 2.0
- **Role:** Self-signed TLS certificate generation for standalone deployments.

### [governor](https://github.com/antifuchs/governor)
- **License:** MIT
- **Role:** Rate limiting on HTTP endpoints and webhook ingestion.

---

## Platform & System

### [keyring](https://github.com/hwchen/keyring-rs)
- **License:** MIT / Apache 2.0
- **Role:** OS-native credential storage (Windows Credential Manager, Linux Secret Service, macOS Keychain) for saved server passwords.

### [clap](https://github.com/clap-rs/clap)
- **License:** MIT / Apache 2.0
- **Role:** Command-line argument and environment variable parsing for the server binary.

### [sysinfo](https://github.com/GuillaumeGomez/sysinfo)
- **License:** MIT
- **Role:** System metrics (CPU, RAM, disk) for the server admin dashboard.

### [dashmap](https://github.com/xacrimon/dashmap)
- **License:** MIT
- **Role:** Concurrent hashmap for managing active user sessions and voice channel state.

### [rfd](https://github.com/PolyMeilex/rfd)
- **License:** MIT
- **Role:** Native file picker dialogs (screen capture selection, file upload).

### [arboard](https://github.com/1Password/arboard)
- **License:** MIT / Apache 2.0
- **Role:** Clipboard access.

### [winit](https://github.com/rust-windowing/winit)
- **License:** Apache 2.0
- **Role:** Window creation and event loop management.

---

## Linux Platform

### [ashpd](https://github.com/bilelmoussaoui/ashpd)
- **License:** MIT
- **Role:** xdg-desktop-portal client for PipeWire screen capture negotiation (Linux screen sharing).

### [pipewire](https://gitlab.freedesktop.org/pipewire/pipewire) via [pipewire-rs](https://gitlab.freedesktop.org/pipewire/pipewire-rs)
- **License:** MIT
- **Role:** PipeWire media graph access for screen capture and audio on Linux.

### [x11](https://github.com/erlepereira/x11-rs)
- **License:** MIT
- **Role:** Xlib bindings for native X11 window creation and XShm screen capture fallback on Linux.

---

## Text & Parsing

### [pulldown-cmark](https://github.com/raphlinus/pulldown-cmark)
- **License:** MIT
- **Role:** CommonMark Markdown parser for rendering formatted chat messages.

### [regex](https://github.com/rust-lang/regex)
- **License:** MIT / Apache 2.0
- **Role:** URL detection, link preview extraction, and rich content parsing.

### [url](https://github.com/servo/rust-url)
- **License:** MIT / Apache 2.0
- **Role:** URL parsing and validation (including SSRF protection for link previews).

### [unicode-segmentation](https://github.com/unicode-rs/unicode-segmentation)
- **License:** MIT / Apache 2.0
- **Role:** Correct Unicode grapheme cluster boundaries for text rendering.

---

## Assets

### [Noto Sans](https://fonts.google.com/noto)
- **License:** SIL Open Font License (OFL) 1.1
- **Role:** Primary UI typeface and Unicode symbol coverage.

---

## Utility Libraries

| Library | License | Role |
| :--- | :--- | :--- |
| `chrono` | MIT / Apache 2.0 | Date/time handling |
| `tracing` + `tracing-subscriber` | MIT | Structured logging |
| `anyhow` / `thiserror` | MIT / Apache 2.0 | Error handling |
| `uuid` | MIT / Apache 2.0 | Unique identifiers (sessions, webhooks) |
| `rand` | MIT / Apache 2.0 | Cryptographic randomness |
| `bincode` | MIT | Binary encoding for UDP protocol frames |
| `byteorder` | MIT / Apache 2.0 | Byte-order-aware binary parsing |
| `lru` | MIT | LRU cache for rate limiting and session state |
| `parking_lot` | MIT / Apache 2.0 | Fast synchronization primitives |
| `mime_guess` | MIT | MIME type detection for file uploads |
| `dirs` | MIT / Apache 2.0 | Platform-appropriate data directory resolution |
| `open` | MIT | Opens URLs in the system browser |
| `tower` / `tower-http` | MIT | HTTP middleware (CORS, tracing, static files) |

---

*This document serves as formal acknowledgement of the third-party open-source components integrated into GoodComms.*
