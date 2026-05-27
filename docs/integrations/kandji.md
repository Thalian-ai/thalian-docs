# Connect Iru

Step-by-step guide to connecting Iru to Thalian for Apple device management intelligence.

---

## Prerequisites

- **Iru account** with API access enabled
- **Iru admin permissions** to generate API tokens

## Create an API Token in Iru

1. Sign in to the Iru web app
2. Go to **Settings** → **Access** → **API Token**
3. Click **Add Token**
4. Give the token a name (e.g., `Thalian Read-Only`)
5. Set permissions to read-only access for devices and blueprints
6. Click **Save** and copy the token value

## Recommended Permissions

When creating the API token, grant read-only access to the following:

- **Device list** — required to sync the full Apple device inventory
- **Device details** — required to read hardware, OS, and blueprint data
- **Blueprints** — required to read blueprint assignments and library item status

No write permissions are needed.

## Connect in Thalian

1. Go to **Integrations** → **Browse**
2. Find **Iru** and click **Connect**
3. Enter your **Iru subdomain** (e.g., `yourcompany.api.kandji.io`)
4. Paste your **API token**
5. Click **Save** — Thalian validates the credentials and begins the first sync

## What Thalian Syncs

- **Apple devices** — Mac, iPhone, iPad, and Apple TV inventory with hardware and OS details
- **Blueprints** — blueprint assignments and library item status
- **Compliance** — per-device compliance state based on blueprint parameters
- **Installed apps (AI tool detection)** — per-device installed application inventory pulled via `/api/v1/devices/{device_id}/apps`. Each app is classified against a closed catalog of 30+ AI tools (Cursor, Claude desktop, ChatGPT desktop, Ollama, LM Studio, and others). Classified installs surface the **AI tool installed on managed device** finding and feed three cross-platform compound rules that join the device install with the identity's OAuth grants and IDP classification. Per-device fetches run in parallel batches of 10 with graceful per-host failure (logged but not blocking the sync). If your Iru API token does not include **Device library apps** read permission, Thalian skips the app sync and surfaces a **Software inventory** scope warning on the Integrations page; regenerate the token with the permission to enable AI tool detection.

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
