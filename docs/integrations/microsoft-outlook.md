# Connect Microsoft Outlook

Step-by-step guide to connecting Microsoft Outlook to Thalian for email security intelligence.

---

## Prerequisites

- **Microsoft 365 tenant** with Exchange Online
- **Global Reader** or **Exchange Administrator** role to authorize the OAuth consent

## Connect via OAuth

1. Go to **Integrations** → **Browse**
2. Find **Microsoft Outlook** and click **Connect**
3. Click **Authorize with Microsoft**
4. Sign in with your Microsoft admin account
5. Review the requested permissions — Thalian requests read-only scopes for mailbox configuration data
6. Click **Accept** to grant consent
7. You'll be redirected back to Thalian — the integration is now connected

## Requested Permissions

Outlook shares the Microsoft OAuth consent with Entra ID. The scopes specific to Outlook are:

| Scope | Justification |
|---|---|
| `Mail.Read` | Detects suspicious mailbox forwarding rules (a common exfiltration vector). Does not read email body/content |
| `MailboxSettings.Read` | Reserved for future mailbox configuration analysis |

For the full list of Microsoft scopes, see [Connect Microsoft Entra ID](./microsoft-entra-id.md).

## What Thalian Syncs

- **Mailbox monitoring** — mailbox configurations and delegation settings
- **Forwarding rule detection** — inbox rules that forward mail externally, which can indicate compromised accounts or data exfiltration

## Remediation actions

Outlook findings support **Revoke sessions** (`revoke_sessions`) only, which delegates to Microsoft Graph to invalidate the user's active refresh tokens and force re-authentication at next sign-in.

Account-wide lifecycle actions (suspend, force password change, MFA enroll, admin role removal) should be performed through the [Microsoft Entra ID integration](./microsoft-entra-id.md) rather than from an Outlook finding. Disabling the user at the Outlook scope would disable the entire Microsoft 365 identity, which is rarely the intent when responding to a mailbox-specific finding such as a suspicious forwarding rule.

## Troubleshooting

- **Insufficient permissions:** The authorizing account must have Exchange admin or Global Reader permissions
- **Missing mailboxes:** Ensure the connected tenant includes the Exchange Online licenses for the mailboxes you expect to see

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
