# Connect Fleet

Step-by-step guide to connecting Fleet to Thalian for cross-platform endpoint intelligence.

---

## Prerequisites

- **Fleet instance** running Fleet 4.x or later (self-hosted or Fleet Cloud)
- **Fleet admin** account or API access to generate an API token
- Your Fleet server URL (e.g., `https://fleet.yourcompany.com`) — must be reachable over HTTPS from the internet (see [Network requirements](#network-requirements) below)

---

## Network requirements

Thalian's sync worker makes outbound HTTPS calls to your Fleet server URL to pull host inventory and send remediation actions. This means **your Fleet instance must be reachable from the public internet** on port 443.

### Fleet Cloud

No action needed. Fleet Cloud (`<your-org>.fleetdm.io`) is already publicly reachable.

### Self-hosted Fleet on a VPS or server

Your Fleet instance needs a public hostname or IP with a valid TLS certificate. If Fleet is behind a firewall, allow inbound HTTPS traffic from Thalian's sync worker. If you use a private domain or self-signed certificate, the connection will fail — Fleet must be reachable via a publicly trusted HTTPS endpoint.

For Fleet deployment documentation, see [Fleet's self-hosting guide](https://fleetdm.com/docs/deploy/introduction-to-fleet).

### Local Fleet instance (development or testing)

Fleet running on `localhost` is not reachable from Thalian's sync worker. Use a tunneling tool to expose your local instance publicly:

```bash
# Using ngrok (https://ngrok.com)
ngrok http 8080
```

Copy the `https://` URL ngrok generates (e.g., `https://abc123.ngrok.io`) and use that as your Fleet server URL in Thalian. Note that free ngrok URLs change on restart — for a stable setup, use a paid ngrok plan with a fixed subdomain or deploy Fleet on a VPS.

---

## Generate an API token in Fleet

Thalian uses a Fleet API token for authentication. The token needs at least `observer` role at the global scope to read all hosts.

**Option A — Use an existing user's API token:**

1. Sign in to your Fleet admin console
2. Go to **Settings** → **My account**
3. Click **Get API token** to view or generate your token
4. Copy the token value

**Option B — Create a dedicated read-only user (recommended):**

Create a dedicated Fleet user for Thalian so you can revoke access independently:

```bash
fleetctl user create \
  --name "Thalian" \
  --email "thalian-sync@yourcompany.com" \
  --global-role observer
```

Then sign in as that user and copy the API token from **Settings** → **My account**.

---

## Connect in Thalian

1. Go to **Integrations** → **Browse**
2. Find **Fleet** and click **Connect**
3. Enter your **Fleet server URL** — include `https://` (e.g., `https://fleet.yourcompany.com`)
4. Paste your **API token**
5. Click **Save** — Thalian validates the connection and begins the first sync

---

## What Thalian syncs

Thalian syncs automatically every hour. You can also trigger a manual sync from **Integrations** at any time.

- **Host inventory** — all enrolled hosts across Windows, macOS, and Linux, with hardware details, OS version, and last check-in timestamp
- **Policy results** — osquery policy pass/fail status per host, used to compute compliance
- **Compliance status** — derived from Fleet policy results: a host with zero failing policies is `compliant`; one or more failures marks it `non-compliant`
- **Disk encryption** — whether FileVault, BitLocker, or Linux disk encryption is verified or enforced
- **MDM enrollment** — whether the host is enrolled manually or automatically
- **Device ownership** — Fleet's `primary_user` field is used to correlate devices to identities across your other integrations (e.g., matching a Fleet host to the same person's Okta account or GitHub access)
- **Installed software (AI tool detection)** — per-host installed application inventory pulled via Fleet's `/api/v1/fleet/hosts/{id}/software` endpoint. Each app is classified against a closed catalog of 30+ AI tools (Cursor, Claude desktop, ChatGPT desktop, Ollama, LM Studio, and others). Classified installs surface the **AI tool installed on managed device** finding and feed three cross-platform compound rules that join the device install with the identity's OAuth grants and IDP classification. Software inventory access is included in the existing API scope — no re-consent or configuration change required.

---

## Remediation actions

From a Fleet finding in Thalian, you can trigger the following actions directly against the device without leaving Thalian:

| Action | What it does |
|---|---|
| **Sync device** | Triggers Fleet to immediately refetch the latest host data (osquery check-in) |
| **Remote lock** | Locks the device — the user must enter a PIN or passcode to regain access |
| **Retire device** | Wipes the device and removes it from Fleet inventory |

!!! warning "Retire device is irreversible"
    The **Retire device** action sends a full wipe command to the endpoint and removes it from Fleet. This cannot be undone. Only use it for lost, stolen, or decommissioned devices.

---

## Troubleshooting

- **Connection refused or timeout:** Your Fleet server URL is not reachable from the internet. Review the [Network requirements](#network-requirements) section above.
- **Invalid token:** Ensure the token was copied without extra whitespace and has not been revoked. Verify the token works by running `fleetctl get hosts --config-path /dev/null` or making a direct API call.
- **No hosts found:** The token must have at least `observer` role at the global scope. A team-scoped observer will only see hosts in that team.
- **TLS error:** Fleet must be served with a publicly trusted TLS certificate. Self-signed certificates are not supported.

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
