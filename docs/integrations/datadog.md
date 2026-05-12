# Connect Datadog

Step-by-step guide to connecting Datadog to Thalian for observability access intelligence and identity gap detection.

---

## What Thalian detects

Thalian syncs your Datadog users and role assignments, cross-references them against your corporate identity provider (Okta, Entra ID, Google Workspace, JumpCloud, or OneLogin), and surfaces access risks:

- **Admin not in IDP** — a Datadog admin has no matching account in your identity provider, meaning their access is outside your offboarding workflow (critical)
- **Admin without MFA** — a Datadog admin does not have multi-factor authentication enabled (high — only fires when no primary IDP is connected, since SAML/SSO handles authentication in that case)
- **User not in IDP** — an active Datadog user has no matching corporate identity (high)
- **Offboarded employee still active** — a user suspended or deprovisioned in your IDP still has an active Datadog account (critical)

---

## Prerequisites

- **Datadog account** — Admin access to your Datadog organization
- **API Key** — from **Organization Settings → API Keys**
- **Application Key** — from **Organization Settings → Application Keys**; the Application Key must have the `user_access_read` scope (or belong to an admin user)

---

## Create an API Key and Application Key

1. Log in to Datadog and go to **Organization Settings** (gear icon, bottom left).
2. Under **Access**, select **API Keys**.
3. Click **+ New Key**, enter a name like `Thalian`, and copy the API key.
4. In the same **Access** section, select **Application Keys**.
5. Click **+ New Key**, enter a name like `Thalian`, and copy the Application Key.

!!! warning "Application Key scope"
    The Application Key inherits the permissions of the user who creates it. To list all users and roles, the key must belong to an admin user or have the `user_access_read` permission explicitly granted.

---

## Connect in Thalian

1. Go to **Integrations** → **Browse** and find **Datadog**.
2. Click **Connect**.
3. Enter your **API Key** and **Application Key**.
4. If your Datadog organization is on a non-US site (EU, US3, US5, AP1), enter the site domain in the **Site** field (e.g., `datadoghq.eu`). Leave blank for the default US site (`datadoghq.com`).
5. Click **Save and Sync**.

---

## What is synced

- **Users** — all Datadog users (active, disabled, pending) with name, email, and status
- **Role assignments** — used to classify users as admin (Datadog Admin Role or any custom role with "admin" in the name) vs. standard user
- **MFA status** — per-user MFA enrollment flag (stored as metadata for the admin-no-MFA rule)

Thalian does **not** sync Datadog monitors, dashboards, logs, metrics, or application data.

---

## Notes

- **SAML/SSO:** If your organization uses SAML single sign-on for Datadog (via Okta, Entra ID, etc.), the admin-without-MFA rule is automatically suppressed — authentication and MFA are handled by your IDP, not Datadog's native MFA. Offboarding gap detection still applies.
- **Site configuration:** Supported Datadog sites: `datadoghq.com` (US), `datadoghq.eu` (EU), `us3.datadoghq.com`, `us5.datadoghq.com`, `ap1.datadoghq.com`, `ddog-gov.com`.

---

## Remediation actions

From a Datadog finding in Thalian, you can trigger the following actions directly against the user without leaving the **Findings** page. Actions use the API key and Application key configured during connect and call the Datadog Management API v2 at `PATCH /api/v2/users/{userId}`.

| Action | What it does |
|---|---|
| **Suspend user** (`suspend_user`) | Sets the user's `disabled` attribute to `true`, blocking sign-in to your Datadog organization while preserving the account record for audit |
| **Unsuspend user** (`unsuspend_user`) | Restores access by setting `disabled` back to `false` |

The Application key used for sync must have the `user_access_manage` permission for these actions to succeed. A key with only `user_access_read` can run sync but will fail at remediation with a 403.

!!! warning "Not exposed by the Datadog API"
    MFA enforcement and session revocation are not currently available through the Datadog Management API. To force re-authentication, suspend and unsuspend the user, which terminates active sessions as a side effect.

---

*For more information on how Thalian detects access gaps, see [Findings and Remediation](../findings-and-remediation.md).*
