# Connect Microsoft Intune

Step-by-step guide to connecting Microsoft Intune to Thalian for endpoint management intelligence.

---

## Prerequisites

- **Microsoft 365 tenant** with Intune licenses assigned
- **Intune Administrator** or **Global Reader** role to authorize the OAuth consent

## Connect via OAuth

1. Go to **Integrations** → **Browse**
2. Find **Microsoft Intune** and click **Connect**
3. Click **Authorize with Microsoft**
4. Sign in with your Microsoft admin account
5. Review the requested permissions — Thalian requests read-only scopes for device management data
6. Click **Accept** to grant consent
7. You'll be redirected back to Thalian — the integration is now connected

## Requested Permissions

Intune shares the Microsoft OAuth consent with Entra ID. Two scopes are specific to Intune:

- `DeviceManagementManagedDevices.Read.All` — pulls managed device inventory for endpoint posture checks
- `DeviceManagementApps.Read.All` — pulls per-device installed application inventory for AI tool detection (see below)

For the full list of Microsoft scopes, see [Connect Microsoft Entra ID](./microsoft-entra-id.md).

!!! note "Existing Microsoft connections need a one-time reconnect"
    `DeviceManagementApps.Read.All` was added to the Microsoft OAuth scope set in May 2026 alongside the device-side AI agent detection capability. If you connected Microsoft before this scope was added, you will see a one-time **Software inventory** scope warning on the Integrations page. Click **Reconnect** to grant the scope and dismiss the warning. The reconnect is audit-logged as `integration_scope_extended` for SOC 2 CC8.1 attribution. Your other Microsoft integrations (Entra, Outlook, Teams, SharePoint) continue working at their existing scope until you reconnect.

## Alternative: API Credentials

If your organization restricts OAuth consent flows:

1. Register an application in **Entra ID** → **App registrations**
2. Grant the application `DeviceManagementManagedDevices.Read.All` AND `DeviceManagementApps.Read.All` permissions (application type)
3. Create a **client secret**
4. In Thalian, select the **API** connection method
5. Enter your **Tenant ID**, **Client ID**, and **Client Secret**
6. Click **Save**

## What Thalian Syncs

- **Devices** — managed devices including OS, model, and enrollment status
- **Compliance status** — per-device compliance state and policy violations
- **Configurations** — device configuration profiles and their assignment status
- **Installed applications (AI tool detection)** — per-device sideloaded application inventory pulled via `/v1.0/deviceManagement/managedDevices/{id}/detectedApps`. Each app is classified against a closed catalog of 30+ AI tools (Cursor, Claude desktop, ChatGPT desktop, Ollama, LM Studio, and others). Classified installs surface the **AI tool installed on managed device** finding and feed three cross-platform compound rules that join the device install with the identity's OAuth grants and IDP classification. Requires `DeviceManagementApps.Read.All` (see Requested Permissions above). Per-device fetches run in parallel batches of 10 with graceful per-host failure. Without the scope, Thalian gracefully skips the app sync and surfaces a **Software inventory** scope warning on the Integrations page.

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
