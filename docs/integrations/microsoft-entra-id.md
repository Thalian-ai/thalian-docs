# Connect Microsoft Entra ID

Step-by-step guide to connecting Microsoft Entra ID (formerly Azure AD) to Thalian for identity and access intelligence.

---

## Prerequisites

- **Microsoft 365 tenant** with an Entra ID directory
- **Global Reader** or **Global Administrator** role to authorize the OAuth consent

## Connect via OAuth

1. Go to **Integrations** → **Browse**
2. Find **Microsoft Entra ID** and click **Connect**
3. Click **Authorize with Microsoft**
4. Sign in with your Microsoft admin account
5. Review the requested permissions — Thalian requests read-only scopes for directory data and sign-in logs
6. Click **Accept** to grant consent
7. You'll be redirected back to Thalian — the integration is now connected

### Scope Warnings

If your tenant policies restrict certain consent scopes, Thalian detects this and shows which features are degraded. You can reconnect at any time to grant additional permissions.

## Requested Permissions

A single Microsoft OAuth consent covers Entra ID and all other Microsoft integrations (Intune, Outlook, SharePoint, Teams). Thalian requests the following scopes:

| Scope | Used by | Justification |
|---|---|---|
| `User.Read.All` | Entra ID | Enumerates all tenant users — names, departments, MFA status, account enabled/disabled — to build the identity inventory and risk scores |
| `Directory.Read.All` | Entra ID | Reads directory role assignments to classify admin vs. standard accounts and detect privilege escalation |
| `AuditLog.Read.All` | Entra ID | Ingests sign-in logs and directory audit events to detect risky sign-ins, impossible travel, MFA bypass, and privilege changes |
| `Application.Read.All` | Entra ID | Discovers enterprise app registrations and their role assignments to identify overprivileged or risky third-party OAuth apps |
| `Policy.Read.All` | Entra ID | Reads Conditional Access policies to detect report-only, disabled, or MFA-gap policies. CA rules stay silent until this scope is granted |
| `DeviceManagementManagedDevices.Read.All` | Intune | Pulls Intune-managed device inventory — OS version, compliance state, encryption status — for endpoint posture checks |
| `Mail.Read` | Outlook | Detects suspicious mailbox forwarding rules (a common exfiltration vector). Does not read email body/content |
| `MailboxSettings.Read` | Outlook | Reserved for future mailbox configuration analysis |
| `Sites.Read.All` | SharePoint | Reads SharePoint site metadata and external sharing settings to flag overshared or publicly accessible sites |
| `ChannelMessage.Send` | Teams | Reserved for future Teams alert delivery |
| `Team.ReadBasic.All` | Teams | Reserved for future Teams workspace enumeration |
| `offline_access` | All | Standard OAuth — allows token refresh without re-prompting the admin |
| `openid` | All | Standard OIDC — required to receive an id_token for tenant ID extraction |
| `profile` | All | Standard OIDC — returns admin's display name during initial connect |
| `email` | All | Standard OIDC — returns admin's email address during initial connect |

## Alternative: API Credentials

If your organization restricts OAuth consent flows, you can connect using application credentials instead:

1. Register an application in **Entra ID** → **App registrations**
2. Grant the application the `Directory.Read.All` and `AuditLog.Read.All` permissions (application type)
3. Create a **client secret**
4. In Thalian, select the **API** connection method
5. Enter your **Tenant ID**, **Client ID**, and **Client Secret**
6. Click **Save**

## What Thalian Syncs

- **Users** — full directory including status, last sign-in, and license assignments
- **Groups** — group memberships, dynamic groups, and role assignments
- **Sign-in logs** — successful and failed sign-ins with location and device details
- **Enterprise apps** — registered and consented applications
- **Conditional access** — policies and their current state (requires `Policy.Read.All`)
- **Identity Protection** — risky users list with risk level and last risk event details
- **PIM role assignments** — Privileged Identity Management permanent role assignments (vs. time-limited eligible assignments)
- **Admin authentication methods** — which authentication methods each admin account has registered, used to detect weak or absent MFA factors
- **Guest invitation policy** — org-level settings for who can invite external guests and whether guest accounts require MFA

!!! note "No reconnection required for existing connections"
    All Phase 2 data (Identity Protection, PIM, admin auth methods, guest policy) is accessible with the scopes granted during initial OAuth setup. Existing Entra connections do not need to be reconnected.

## AI tool detection

After syncing Entra, Thalian inspects every enterprise application's delegated OAuth grants and matches the app name against a known AI tool catalog (Claude, ChatGPT, Cursor, Gumloop, Microsoft Copilot, Perplexity, n8n, Zapier, and 30+ others). AI tools surface in the AI Governance category with their granted Microsoft Graph scopes attached.

### Findings that fire on Entra-sourced AI tools

- **AI tool has access to corporate data** (high, critical at five or more tools): an unsanctioned AI tool holds read access to Mail, Files, Sites, Calendars, or Contacts via Microsoft Graph delegated scopes.
- **AI tool with write access to email or files** (critical): an AI tool holds `Files.ReadWrite`, `Sites.ReadWrite`, `Mail.ReadWrite`, or `Mail.Send` scopes.
- **Unsanctioned AI tool in widespread use** (medium, tiers up with adoption): an AI tool is granted to ten or more users without being sanctioned.
- **Terminated employee AI tool still has data access** (high): an offboarded employee retains active AI tool grants. Requires an HR integration (Rippling, BambooHR, or Workday) connected for the cross-platform join.

### Scope coverage

Findings key on whichever delegated scopes the Entra OAuth grant includes. Detection covers Google OAuth scope vocabulary (`drive`, `gmail`, `mail`, `calendar`, `contacts`, `docs`, `sheets`, `slides`) and Microsoft Graph delegated scope vocabulary (`Mail.*`, `Files.*`, `Sites.*`, `Calendars.*`, `Contacts.*`). App-only application permissions granted via `appRoleAssignments` are not yet evaluated.

### No re-authorization required

The existing `Directory.Read.All` scope (already requested during initial connect) covers reading `oauth2PermissionGrants`. Existing Entra connections begin surfacing AI tool findings on the next sync without any admin action.

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
