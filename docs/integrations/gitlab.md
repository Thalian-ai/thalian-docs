# Connect GitLab

Step-by-step guide to connecting a GitLab group to Thalian for developer access intelligence and identity gap detection.

---

## What Thalian detects

Thalian syncs your GitLab group members and projects, cross-references them against your corporate identity provider (Okta, Entra ID, Google Workspace, JumpCloud, or OneLogin), and surfaces developer access risks:

- **Admin or maintainer without MFA** — a group owner or maintainer has MFA disabled on their account (high)
- **Admin or maintainer not in IDP** — a maintainer or owner has no matching account in your IDP, meaning their access is outside your offboarding workflow (critical)
- **Developer not in IDP** — a developer or reporter has no matching corporate identity (high)
- **External member with elevated access** — an external user holds Developer or higher access, which allows pushing code and managing pipelines (high)
- **Deploy key with write access** — a deploy key with push permissions can commit code directly to repositories without requiring a merge request or review (medium)
- **Group access token with no expiry** — a group-level access token has no expiration date set, creating a persistent access credential if leaked (medium)
- **Offboarded employee still active** — a user suspended or deprovisioned in your IDP still has active GitLab group membership (high)
- **Stale group member** — a member has had no commit, comment, or merge request activity in 90 or more days (medium)

---

## Prerequisites

- **GitLab group** — Thalian connects at the group level; project-level connections are not supported
- **Owner-level access** — you must be a group Owner to create a Group Access Token
- **GitLab plan** — audit event history requires GitLab Premium or Ultimate; all other detection works on Free

---

## Create a Group Access Token

Thalian uses a Group Access Token (PAT) for authentication. This does not require OAuth — no redirect or app authorization is needed.

1. In GitLab, navigate to your group
2. Go to **Settings** → **Access Tokens**
3. Click **Add new token**
4. Set the token name (e.g. `thalian-sync`)
5. Leave the expiration date blank or set a long expiry (Thalian will alert you if the token expires)
6. Select **Owner** as the role — this is required to read member emails, MFA status, and group tokens
7. Select these scopes:
   - `read_api` — reads group members, projects, deploy keys, and group tokens
   - `read_user` — reads per-user MFA status (required for `gitlab::admin_no_mfa` detection)
8. Click **Create group access token**
9. **Copy the token value** — it is shown only once

---

## Find your Group ID

1. Navigate to your group in GitLab
2. The Group ID is displayed below the group name on the group overview page
3. You can also find it at **Settings** → **General** → **Advanced** → **Group ID**

---

## Connect in Thalian

1. Go to **Integrations** → **Browse**
2. Find **GitLab** and click **Connect**
3. Enter:
   - **Group Access Token** — the `glpat-xxx` value you copied
   - **GitLab URL** — `gitlab.com` for GitLab.com, or your self-hosted domain (e.g. `gitlab.yourcompany.com`)
   - **Group ID** — the numeric ID from the step above
4. Click **Connect** — the first sync begins immediately

---

## What Thalian syncs

- **Group members** — all members including inherited members from subgroups, with role, email, MFA status, and last activity date
- **Projects** — all non-archived group projects and subgroup projects, tracked as applications
- **Deploy keys** — per-project deploy keys including whether they have write (push) access
- **Group access tokens** — active group-level tokens with scope and expiry information
- **Audit events** — last 30 days of group-level access changes (add/remove member, role changes, visibility changes). Requires GitLab **Premium or Ultimate**. Thalian skips this gracefully on Free plans.

Member emails are used for IDP matching. Thalian requires a connected IDP to perform cross-reference detection.

---

## Required scopes

| Scope | Purpose |
|---|---|
| `read_api` | Reads group members, projects, deploy keys, and group tokens |
| `read_user` | Reads per-user MFA status. Without this scope, `gitlab::admin_no_mfa` will not generate findings |

---

## Self-hosted GitLab

Thalian supports self-hosted GitLab instances. Enter your instance URL in the **GitLab URL** field (e.g. `gitlab.yourcompany.com`). Thalian only uses the hostname — credentials in the URL are stripped automatically.

Ensure your Thalian integration can reach your self-hosted instance over HTTPS. HTTP-only instances are not supported.

---

## Remediation actions

From a GitLab finding in Thalian, you can trigger the following action directly against the group member without leaving the **Findings** page. Actions use the Group Access Token you configured during connect.

| Action | What it does |
|---|---|
| **Remove from group** (`remove_from_group`) | Calls `DELETE /api/v4/groups/{id}/members/{user_id}` to revoke the user's membership in the connected group. The user's GitLab account remains active; only their access to this group is removed |

A Group Access Token cannot suspend or delete the user's GitLab account itself. Account-level lifecycle actions require GitLab Admin API access, which is only available on self-hosted instances and is not exposed to GATs. To fully offboard a user, suspend them in your primary IDP and let SAML/SCIM propagate, or perform the account suspension from the GitLab admin area on a self-hosted instance.

---

## Troubleshooting

- **MFA findings not appearing:** The Group Access Token must include `read_user` scope. Re-create the token with both `read_api` and `read_user` scopes and reconnect
- **Audit events not syncing:** Audit events require GitLab Premium or Ultimate. Thalian detects this and skips the audit pull gracefully — other rules still run
- **Missing members:** Confirm the token role is **Owner**. Lower roles cannot read member emails or group token metadata
- **Self-hosted connection fails:** Verify the GitLab URL is reachable over HTTPS and the domain is correct

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
