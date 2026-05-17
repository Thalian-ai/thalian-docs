# Connect Box

Step-by-step guide to connecting Box to Thalian for identity gap detection and external sharing monitoring.

---

## What Thalian detects

Thalian cross-references Box enterprise users against your corporate identity provider and monitors admin event logs for data exposure activity — 6 detection rules:

| Finding | Severity |
|---|---|
| Offboarded employee retaining Box file access — a Box user whose IDP account is deprovisioned but whose Box account remains active | Critical |
| Box admin not in IDP — a Box admin or co-admin has no matching IDP account | Critical |
| Box user not in IDP — a standard Box enterprise user has no matching IDP account | High |
| External sharing activity — files or folders have been shared externally in the sync window | Medium |
| Box Shield anomaly alert not investigated — one or more Box Shield anomaly alerts from the last 30 days have not been reviewed (requires Box Shield) | High |
| Departing employee bulk-downloading from Box — an offboarded user downloaded files in the 7 days following account suspension or deprovisioning (requires IDP connected) | Critical |

---

## Prerequisites

- **Box Business or Enterprise account**
- **Box admin** permissions to authorize the OAuth connection
- At least one IDP (Okta, Entra ID, Google Workspace, JumpCloud, or OneLogin) connected for IDP gap detection

---

## Connect via OAuth

1. Go to **Integrations** → **Browse**
2. Find **Box** and click **Connect**
3. Click **Authorize with Box** — you'll be redirected to the Box authorization page
4. Sign in with your Box admin account
5. Review the requested permissions and click **Grant access to Box**
6. You'll be redirected back to Thalian — the integration is now connected and the first sync begins

### Requested scopes

| Scope | Justification |
|---|---|
| `manage_managed_users` | Reads all enterprise users — names, emails, roles, and login timestamps |
| `manage_enterprise_properties` | Reads enterprise admin event logs for external sharing activity |

No write permissions are requested. Thalian does not modify Box users, files, or folder permissions.

---

## What Thalian syncs

- **All enterprise users** — name, email, role (admin, co-admin, user), status, and last login timestamp
- **Admin event logs** — external sharing events (`SHARE`, `SHARE_EXPIRATION_UPDATED`) from the Box admin event stream
- **IDP gap detection** — each Box user is cross-referenced against your connected IDP by email address

Thalian does not read file contents, folder structures, or any document data. Only user identity and sharing activity metadata is collected.

---

## Remediation

Box is a **read-only integration** — Thalian surfaces findings but does not modify Box user accounts or file permissions. Remediation for Box findings targets the IDP or the Box Admin Console:

- **Offboarded employee / user not in IDP** — deactivate the user in Box Admin Console → **Users**, or provision them in your IDP
- **Admin not in IDP** — provision the admin in your IDP, or demote their Box role to a standard user
- **External sharing activity** — review the shared items in Box Admin Console → **Reports** and revoke external access as needed

---

## Troubleshooting

- **No users found:** The authorizing account must have admin-level access to read the full enterprise user list
- **External sharing events missing:** Admin event log access requires a Box Business Plus or Enterprise plan
- **Box Shield rules not firing:** Shield anomaly alert syncing requires a Box Enterprise Plus plan with Box Shield enabled. Confirm Shield is active in your Box admin settings
- **IDP gaps not detected:** Confirm at least one IDP is connected and synced before Box findings will fire
- **OAuth error during connect:** Ensure pop-ups are not blocked and the admin account has sufficient permissions

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
