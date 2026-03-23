# GoodComms Webhook & Slash Command System

GoodComms supports two ways to integrate external services: **Webhooks** for HTTP-triggered messages, and **Slash Commands** for user-initiated commands. Both can trigger external HTTP requests or execute server-side actions.

---

## Webhooks

Webhooks allow external applications to push messages directly into channels using secure, token-authenticated HTTP requests. Ideal for monitoring tools, CI/CD pipelines, or custom bots.

### Quick Start

1. Go to the **Admin** tab → **Channels** → find your target channel → **Webhooks**
2. Enter a name and click **Create**
3. **Copy the generated URL immediately** — the token cannot be retrieved again

### Sending a Message

**Endpoint:** `POST https://<your-server>/api/webhooks/<your-token>`

**Payload:**
```json
{
  "content": "### 🛡️ System Alert\n**Status**: Operational\n**Trace**: [View Logs](https://example.com)",
  "username": "Optional Override Name"
}
```

### Security
* **Token**: High-entropy UUIDs
* **Storage**: Argon2 hashed — raw tokens never touch the database
* **Scope**: Bound to the channel they were created in

### Rate Limiting
Subject to global and per-token rate limits to prevent abuse.

---

## Slash Commands

Slash commands let users type `/command` in chat to trigger responses, webhooks, or server-side actions.

### Creating Commands

1. Go to the **Admin** tab
2. Click **Slash Commands** in the sidebar
3. Click **+ New Command**
4. Fill in the fields and click **Create**:
   - **Name**: The command word (without `/`) — e.g., `help`
   - **Description**: Shown in the autocomplete dropdown when users type `/`
   - **Type**: `Text`, `Webhook`, or `Action`
   - **Response / URL**: What happens when triggered (depends on type — see below)
   - **Required Permission** *(optional)*: Restrict who can run this command

### Command Types

#### Text
Posts a fixed text response into the channel when the command is used.

| Field | Value |
|-------|-------|
| Name | Command word, e.g. `rules` |
| Type | `Text` |
| Response | The message to post |

**Example:** A `/rules` command with response `Please read #rules before posting.` — any user typing `/rules` will see that message appear in chat.

---

#### Webhook
Triggers an HTTP POST to an external URL when the command is used. Use `{}` in the request body to insert any arguments the user typed after the command, and `{username}` to include the triggering user's name.

| Field | Value |
|-------|-------|
| Name | Command word, e.g. `deploy` |
| Type | `Webhook` |
| Webhook URL | The external endpoint to call |
| Response | JSON body to send — use `{}` for user-supplied arguments |

**Example:** A `/deploy` command with Webhook URL pointing to your CI service and body `{"ref": "main", "triggered_by": "{username}"}` — typing `/deploy` fires a POST to your endpoint with that payload.

**Example with arguments:** A `/ticket` command with body `{"title": "{}"}` — typing `/ticket Login page broken` POSTs `{"title": "Login page broken"}` to your endpoint.

---

#### Action
Executes a built-in server-side moderation action.

| Field | Value |
|-------|-------|
| Name | Command word, e.g. `kick` |
| Type | `Action` |
| Response | The action name (see table below) |
| Required Permission | Permission required to run this command |

**Built-in Actions:**

| Action | Usage | Notes |
|--------|-------|-------|
| `kick` | `/kick <username> [reason]` | Requires `admin_access` |
| `ban` | `/ban <username> [reason]` | Requires `admin_access` |
| `timeout` | `/timeout <username> [minutes]` | Requires `admin_access` |
| `untimeout` | `/untimeout <username>` | Requires `admin_access` |
| `slowmode` | `/slowmode <channel_id> <seconds>` | Requires `admin_access` |

### Managing Permissions

* **Creating/Deleting commands**: Requires `ManageWebhooks` permission or admin
* **Executing commands**: Optional `required_permission` field per command

Available permission strings:
- `admin_access`
- `manage_users`
- `manage_channels`
- `manage_messages`
- `manage_webhooks`

---

## Quick Reference

| Feature | Trigger | Use Case |
|---------|---------|----------|
| Webhook | External HTTP POST | CI/CD alerts, monitoring |
| Text Command | `/text` in chat | Info commands, responses |
| Webhook Command | `/webhook` in chat | Trigger external services |
| Action Command | `/action` in chat | Moderation, server control |

---

**Last Updated**: March 21, 2026 — v0.9.98
