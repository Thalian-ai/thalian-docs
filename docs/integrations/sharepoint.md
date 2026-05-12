# Connect SharePoint

Step-by-step guide to connecting SharePoint to Thalian for collaboration security intelligence.

---

## Prerequisites

- **Microsoft 365 tenant** with SharePoint Online
- **SharePoint Administrator** or **Global Reader** role to authorize the OAuth consent

## Connect via OAuth

1. Go to **Integrations** → **Browse**
2. Find **SharePoint** and click **Connect**
3. Click **Authorize with Microsoft**
4. Sign in with your Microsoft admin account
5. Review the requested permissions — Thalian requests read-only scopes for SharePoint site and sharing data
6. Click **Accept** to grant consent
7. You'll be redirected back to Thalian — the integration is now connected

## Requested Permissions

SharePoint shares the Microsoft OAuth consent with Entra ID. The scope specific to SharePoint is `Sites.Read.All`, which reads site metadata and external sharing settings to flag overshared or publicly accessible sites. For the full list of Microsoft scopes, see [Connect Microsoft Entra ID](./microsoft-entra-id.md).

## What Thalian Syncs

- **Sites** — SharePoint site inventory including site type, owner, and storage usage
- **External sharing** — sites and documents shared with external users or anyone links
- **Document permissions** — permission levels and sharing configurations across sites

## Remediation actions

SharePoint findings support **Revoke sessions** (`revoke_sessions`) only, which delegates to Microsoft Graph to invalidate the user's active refresh tokens and force re-authentication at next sign-in.

Account-wide lifecycle actions (suspend, force password change, MFA enroll, admin role removal) should be performed through the [Microsoft Entra ID integration](./microsoft-entra-id.md) rather than from a SharePoint finding. Disabling the user at the SharePoint scope would disable the entire Microsoft 365 identity, which is rarely the intent when responding to an oversharing or external-link finding.

## Troubleshooting

- **Missing sites:** Ensure the authorizing account has at least read access to the SharePoint admin center
- **Scope warnings:** If your tenant restricts SharePoint API consent, reconnect and grant the additional permissions

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
