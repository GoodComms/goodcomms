# GIF Integration Setup

GoodComms supports **Klipy** and **Giphy** for GIF search. To maintain privacy and sovereignty, GoodComms does not ship with a built-in API key. Each server administrator must provide their own.

---

## Recommended: Klipy

Klipy is the recommended provider. It offers a selection similar to Tenor and is straightforward to set up.

1. Go to the [Klipy Developers Dashboard](https://klipy.co/developers) and create an account.
2. Create an application to generate your API key.
3. Add `KLIPY_API_KEY=your_key` to your server environment.

## Optional: Giphy

Giphy is a solid alternative, though its free "Beta" keys have stricter rate limits.

1. Go to the [GIPHY Developers Dashboard](https://developers.giphy.com/dashboard/) and create an account.
2. Click **Create an App**, select **API**, and fill in the basic details.
3. Copy the generated Beta Key.
4. Add `GIPHY_API_KEY=your_key` to your server environment.

---

## Configuring the Server

You can provide either or both keys. If both are set, GoodComms will use Klipy and fall back to Giphy.

**Docker Compose** — add to your `docker-compose.yml` environment:

```yaml
services:
  goodcomms-server:
    environment:
      - KLIPY_API_KEY=your_klipy_key_here
      - GIPHY_API_KEY=your_giphy_key_here   # optional
```

**Bare binary — Linux:**
```bash
KLIPY_API_KEY="your_key" ./gc-server
```

**Bare binary — Windows (PowerShell):**
```powershell
$env:KLIPY_API_KEY="your_key"; .\gc-server.exe
```

---

## Verification

1. Restart the server with the new environment variables.
2. In the client, click the smiley face icon in the chat input bar.
3. Open the **GIFs** tab and run a search — results should appear.
