# Release Notes

All notable changes for each release.

---

## [0.9.99] - 2026-07-04 — The Video Streaming & Reliability Update

### Screen Sharing
- **Perfectly smooth motion from any monitor**: The encoder now samples frames on an exact fixed-rate clock instead of following your monitor's refresh timing — streaming from a 144Hz or 120Hz display no longer produces periodic micro-stutter for viewers.
- **60 FPS streaming option**: Settings → Video Streaming → Stream Frame Rate (30 default / 60). Smoother motion at the same bitrate.
- **New Go Live picker**: Clicking Go Live opens a single window with live preview snapshots of every screen and window, plus the Include Audio toggle and capture device — click a preview and the stream starts instantly.
- **Encoder fallback**: If your GPU encoder fails — at startup or mid-stream — the stream automatically switches to software encoding with a notification instead of dying silently. A new Video Encoder setting (Automatic / Hardware Only / Force Software) covers flaky GPU drivers.
- **Stream settings apply live**: Frame rate, encoder mode, and audio auto-leveling changes now take effect on a running stream — no restart needed.
- **Auto-leveled stream audio**: Quiet system audio (e.g. low speaker volume at night) is automatically brought up to a consistent level for viewers. Settings → "Auto-level stream audio" (default on).
- **Resizing a shared window now follows through to viewers** instead of the stream staying at the original size forever.
- **Viewer improvements**: per-stream mute button, dark-themed viewer toolbar, and automatic self-healing if a stream freezes after a network blip.
- **Fullscreen game capture guard**: Exclusive-fullscreen games that can't be captured now stop cleanly with guidance (use Borderless Windowed) instead of a silent dead stream.

### Server Administration
- **Server settings now actually work**: "Allow Public Registration" is enforced, Max Upload Size is enforced (live, 0 = unlimited), and Max Download Speed applies without a restart. Invalid values are rejected with a clear message when saving.
- **Automatic database backups**: Set an interval, how many backups to keep (oldest rotated out), and where they go. A new Maintenance page in the admin panel adds manual Backup, Restart Server, Clear All Messages, and Emergency Stop — all behind confirmation dialogs. Backups use a crash-safe snapshot method.
- **Destructive actions ask first**: Deleting a user, channel, or role now shows a confirmation dialog. Deleting a user also disconnects them immediately.
- **Settings fields save explicitly**: Text settings now have a Save button per field instead of applying every keystroke.

### Reliability & Fixes
- **Stream audio works after restarting a stream**: Restarting a screen share could leave viewers with video but no audio (for up to 20 seconds, or the entire stream if the previous one was short). Viewers now resync to the new stream's audio immediately.
- **Switching servers is clean**: Connecting to a different server fully resets the previous session (no more leftover channels or state), and the Servers tab shows your current connection with a proper Disconnect button.
- **Fixed a crash affecting new installs**: A single corrupted image on a server could crash every freshly-installed client on login. Images are now validated on both the server (at upload) and the client (at display).
- **Rate limiter fix for reverse-proxy deployments**: Behind a reverse proxy, a burst of legitimate reconnects could trip the brute-force limiter and ban the proxy itself, taking the server offline for everyone. Only failed logins count now, and proxy deployments should set `TRUST_PROXY=true` (see server.env.example).
- **Upload errors are human-readable**, and error popups can actually be dismissed.
- **Failed drive transfers clear themselves and show why**: a failed upload/download used to leave its progress entry stuck as "active" forever with no feedback.
- **Unauthorized admin actions now return a clear error** instead of nothing happening.
- **Drive folders open on double-click.**

---

## [0.9.98a] - 2026-06-06

### Improvements
- **Smoother screen-share video**: The encoder now uses constant-bitrate rate control, so it spends its full bitrate budget steadily — fast on-screen motion stays sharp instead of getting starved and blocky.
- **Display no longer sleeps while streaming or watching**: Your monitor and the app stay awake during an active screen share or while watching one, preventing the freeze that happened when the display went to sleep — especially on laptops on battery.

### Bug Fixes / UX
- **Clean stop when a shared window closes**: Closing the window or screen you're sharing now stops the share cleanly with a clear notification instead of erroring.
- **Installer no longer pre-checks the desktop shortcut**: "Create desktop shortcut" is now unchecked by default during installation.
- **Version label placement**: The client version now appears directly under its label.

---

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
