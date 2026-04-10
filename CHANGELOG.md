# Release Notes

All notable changes for each release.

## [0.9.98] - 2026-03-22 (updated 2026-04-09)

### Post-Release Fixes (2026-04-09)
- **Windows encoder selection improved**: Fixed an issue where the video encoder might not be selected correctly on systems with multiple GPUs (e.g., NVIDIA discrete + AMD iGPU). The encoder selection now properly identifies and uses the correct vendor. Additionally, a software encoder fallback is now available if hardware encoding cannot be initialized, preventing silent video failures.

### New Features
- **Channel editing**: Server administrators can now edit a channel's name, description, and text/voice flags directly from the Admin panel. Changes apply immediately for all connected users.
- **Channel reordering**: Channels can now be reordered using ▲▼ buttons in the Admin panel. The new order is saved server-side and reflected in all clients' sidebars instantly.

### Bug Fixes
- **Drive folder sizes now display correctly**: Folders in Server Drive now show the total size of all files they contain. Previously all folders displayed 0.0 MB.
- **Drive file sizes now show correct units**: Small files (under 1 MB) now display in bytes or kilobytes instead of "0.0 MB".
- **Window position restored off-screen after monitor change**: The app window no longer opens invisibly when launched after a monitor is disconnected or the display configuration changes. The saved position is now validated before restore.
- **Server (Mode B/C): process exited immediately after starting**: When running in direct-TLS or self-signed certificate mode the server would start, bind to its ports, then exit with code 0 right away. Fixed — the server now stays running until a shutdown signal is received.
- **Reconnecting while viewing a DM moved you to the first channel**: After a network reconnect, if you were in a direct message conversation the client would navigate away to the first channel. Direct message context is now preserved across reconnects.
- **Voice and video fail silently on non-default ports**: If your server runs voice and video on ports other than 4077/4078 (e.g. a second server or a custom deployment), connecting clients could chat normally but voice and video would not work. The server now tells each client the correct ports to use at login.

### UX Improvements
- **Permission matrix legend**: A legend now appears below the channel permissions editor — `● Explicit  ◐ Inherited  ○ No Access` — so the indicator states are no longer left to interpretation.
- **Push to Talk hint**: A short explanation now appears below the PTT controls in Audio Settings describing the default mic-gate behaviour and when PTT is the right fix.

---

## [0.9.97] - 2026-03-16 (updated 2026-03-19)

### Post-Release Fixes (2026-03-19)
- **Direct message conversations now persist between sessions**: Opening multiple DM conversations, closing the client, and reopening would cause all DMs to disappear from the sidebar. They now correctly reappear on next login.
- **Sending a direct message is now instant**: Previously there was a ~1 second delay before your own sent message appeared. It now appears immediately.

### Bug Fixes
- **"Loading older messages" bar no longer gets stuck**: Scrolling to the very top of a channel could cause the loading bar to appear and remain permanently — only clearing via Jump to Present or a channel switch. This is now fixed; the bar clears as soon as the history request completes.
- **Pinned message navigation improved**: Clicking a pinned message now brings it into view near the top of the chat and leaves the chat in a scrollable state. Previously the jump could land many messages away or lock the scroll to the very beginning of history.

### Performance
- **Smooth 60 FPS Streaming**: Improved video pacing for high-refresh-rate desktops. Streaming from a 144Hz monitor now feels significantly smoother for all viewers.
- **Optimized Deafened State**: Resource usage is now minimized when you are deafened, as audio packets are dropped immediately instead of being processed in the background.

### Security
- **Video Relay Protection**: Hardened the media relay system to prevent unauthorized stream disruptions or hijacking.
- **Self-service password change**: Users can now change their own password from Settings. Requires current password verification. All other active sessions are automatically logged out for security.
- **Enhanced Input Validation**: Strengthened server-side checks for usernames and file uploads to ensure platform stability.
- **Security Audit Completed**: A comprehensive internal security review has been completed, verifying the integrity of our self-hosted architecture.

### Bug Fixes
- **Voice audio gap when a peer rejoins**: Fixed an issue where an idle user could not hear a peer for up to ~1 minute after the peer restarted their app and rejoined a voice channel. Audio now resumes immediately.
- **Voice Reliability**: Fixed an issue where audio could occasionally cut out when a user left and rejoined a voice channel in rapid succession.
- **General Stability**: Minor fixes to the audio and video subsystems for better long-term session stability.

### Deployment
- Clarified deployment options for reverse proxy setups, self-signed certificates, and manual TLS configurations.

---

## [0.9.96] - 2026-03-12

### Performance
- **Image memory usage**: Better memory management for images and GIFs. Client memory stays bounded even on image-heavy channels.
- **Avatar cache**: Avatars now persist through cache clears and reconnects.

### Bug Fixes
- **Multi-device login**: Online status no longer flickers when switching devices.
- **Voice status**: Fixed race condition where voice channel presence could show incorrect status.
- **Console warnings**: Eliminated repeated layout warning messages.

### UI Improvements
- **Permission matrix**: Admins no longer see their own role as editable in the channel permissions matrix.

### Security
- **Password reset**: Admin password resets now properly invalidate all active sessions.
- **Registration**: Separate login/register modes with password confirmation.

### Known Issues
- Linux: Second screen share session may require restarting the app (KDE Plasma only).

---

## [0.9.95] - 2026-03-10

### Channel Permissions
- **Private channels**: Redesigned permission system. Private channels no longer inherit global role permissions - access must be explicitly granted.
- **Permission matrix**: New interface for setting per-channel permissions.

### Memory & Cache
- **Image cache**: New memory-bounded LRU cache with 150MB budget for images.
- **GIF handling**: Separate 60MB budget for GIFs with optimized decoding.

### Bug Fixes
- **Private channel broadcasts**: Fixed information leak where private channel details could be visible to unauthorized users.
- **Device picker**: Fixed overlay rendering issues in Settings audio device selectors.

---

## [0.9.94] - 2026-03-10

### UI Updates
- Renamed "People" tab to "Members" for clarity.
- Added clickable website link in Settings > About.
- Compacted Settings layout.

### Admin
- Added video stream count to server statistics.

---

**GoodComms** - Engineered for Privacy. Built with Rust.
