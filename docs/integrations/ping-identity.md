# Connect PingOne

Step-by-step guide to connecting PingOne to Thalian for identity and access intelligence.

---

## Prerequisites

- **PingOne admin account** with environment admin permissions
- **Environment ID** — found in **Settings** → **Environment** in the PingOne admin console
- **Region** — your PingOne region (`us`, `eu`, `ca`, or `au`)

## Create a Worker Application in PingOne

Thalian connects using PingOne's OAuth 2.0 client credentials flow. You'll need to create a Worker application in your environment.

1. Sign in to the PingOne admin console
2. Navigate to **Applications** → **Applications**
3. Click **+** to add a new application
4. Select **Worker** as the application type and click **Next**
5. Give the application a name (e.g., `Thalian`) and click **Save**

## Grant the required roles

After creating the Worker application, assign read-only roles so Thalian can access your directory data:

1. Go to the **Roles** tab on your new application
2. Click **Grant Roles**
3. Assign the following role:

| Role | Purpose |
|------|---------|
| `Environment Admin` (read scope) | Read users, role assignments, applications, and MFA settings |

!!! warning "Least privilege"
    If your PingOne plan supports it, use a more restrictive role (e.g., **Identity Data Read Only**) rather than full Environment Admin. Thalian only needs read access.

## Copy your credentials

1. Go to the **Configuration** tab of your Worker application
2. Copy the **Client ID**
3. Click **Generate Secret** and copy the **Client Secret** — it is only shown once

## Connect in Thalian

1. Go to **Integrations** → **Browse**
2. Find **PingOne** and click **Connect**
3. Enter your **Environment ID** (e.g., `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
4. Paste your **Client ID** and **Client Secret**
5. Enter your **Region** (`us`, `eu`, `ca`, or `au`)
6. Click **Connect** — Thalian validates the credentials and begins the first sync

## What Thalian syncs

- **Users** — full directory including status, last login, username, title, and population (org grouping)
- **Admin assignments** — role assignments used to classify users as `admin` vs. `user` identity type
- **SSO applications** — enabled applications with protocol type (SAML, OIDC) and SSO status
- **Org MFA settings** — global MFA enablement state used for posture analysis

**Note:** Per-user MFA device status requires individual API calls per user (N+1 pattern) and is currently deferred. MFA fields in Thalian will show as unknown for PingOne users; the `ping_identity::admin_no_mfa` rule uses org-level MFA policy data instead.

## Detection rules enabled

Connecting PingOne enables the following platform-specific detection rules, plus all 38 cross-platform IDP rules (offboarding gaps, stale users, MFA coverage, admin sprawl, and more).

| Rule | Severity | What it detects |
|------|----------|-----------------|
| `ping_identity::admin_no_mfa` | High | Admins in PingOne where org-level MFA is not enforced |
| `ping_identity::user_not_in_idp` | High | PingOne users with no matching identity in your primary IDP |
| `ping_identity::offboarded_user_active` | High | Users suspended or deprovisioned in your primary IDP who remain active in PingOne |

## Remediation actions

From a PingOne finding in Thalian, you can trigger the following actions directly against the user without leaving the **Findings** page. Actions call the PingOne Workforce API at `PATCH /v1/environments/{envId}/users/{userId}` with an `enabled` flag, and run under the same Worker application credentials used for sync.

| Action | What it does |
|---|---|
| **Suspend user** (`suspend_user`) | Sets the user to `enabled: false`, blocking sign-in across every PingOne-protected application |
| **Unsuspend user** (`unsuspend_user`) | Reverses a prior suspension by setting `enabled: true` |

The Worker application's granted role must include write access to the users resource for these actions to succeed. The read-only roles described above are sufficient for sync but not for remediation.

!!! warning "Not yet supported through Thalian"
    `force_password_change`, `revoke_sessions`, `force_mfa_enroll`, and `remove_admin_role` are not currently available for PingOne. Perform these actions from the PingOne admin console.

## Troubleshooting

- **Auth failed:** Confirm the Client ID and Client Secret were copied from the **Configuration** tab and that the application is enabled
- **No users synced:** Verify the Worker application has been granted a role with read access to the environment — unauthenticated or under-permissioned apps return empty results
- **Wrong region:** If the connection times out, double-check the region. PingOne API base URLs differ by region (`auth.pingone.com` for US, `auth.pingone.eu` for EU, etc.)
- **Missing admin data:** Role assignment reads require the Worker app to have access to environment-level role APIs — confirm the granted role covers `roleAssignments`

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
