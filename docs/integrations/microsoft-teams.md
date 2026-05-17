# Connect Microsoft Teams

Step-by-step guide to connecting Microsoft Teams to Thalian for finding alert delivery.

---

## Prerequisites

- **Microsoft Teams** workspace where you want to receive alerts
- **Teams admin** or sufficient permissions to manage connectors/webhooks in a channel

## Connect via OAuth

1. Go to **Integrations** → **Browse**
2. Find **Microsoft Teams** and click **Connect**
3. Click **Authorize with Microsoft**
4. Sign in with your Microsoft account
5. Review the requested permissions
6. Click **Accept** to grant consent
7. You'll be redirected back to Thalian — the integration is now connected

## Requested Permissions

Teams shares the Microsoft OAuth consent with Entra ID. The scopes specific to Teams are:

| Scope | Justification |
|---|---|
| `ChannelMessage.Send` | Sends finding alerts and remediation notifications to Teams channels |
| `Team.ReadBasic.All` | Lists available teams for alert channel configuration |

For the full list of Microsoft scopes, see [Connect Microsoft Entra ID](./microsoft-entra-id.md).

## Alternative: Webhook

If your organization restricts OAuth for Teams:

1. In Microsoft Teams, go to the channel where you want to receive alerts
2. Click the **channel name** → **Connectors** (or **Manage channel** → **Connectors**)
3. Find **Incoming Webhook** and click **Configure**
4. Give the webhook a name (e.g., `Thalian Alerts`) and click **Create**
5. Copy the **webhook URL**
6. In Thalian, select the **Webhook** connection method
7. Paste the webhook URL and click **Save**

## Configure Alert Delivery

After connecting Teams:

1. Open the Microsoft Teams integration card in **Integrations**
2. Toggle **Alerts** on
3. Set the **severity threshold** — only findings at or above this level are sent

Thalian sends adaptive card notifications for each new finding that meets your threshold.

## What Thalian Uses Teams For

Teams is used exclusively for **alert delivery**. Thalian does not read or sync messages, files, or user data from Teams. When a new finding is generated that meets your severity threshold, Thalian posts an adaptive card to your configured channel.

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
