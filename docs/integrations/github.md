# Connect GitHub

Step-by-step guide to connecting a GitHub organization to Thalian for developer access intelligence and identity gap detection.

---

## What Thalian detects

Thalian cross-references GitHub organization members and outside collaborators against your corporate identity provider (Okta, Entra ID, Google Workspace, JumpCloud, or OneLogin) to surface access that exists outside your IDP lifecycle:

- **Outside collaborator with write access** — an external user has write or admin access to one or more repositories (high)
- **Org owner not in IDP** — a GitHub organization owner has no matching account in your IDP, meaning their access is invisible to your offboarding workflow (critical)
- **Outside collaborator count** — tracks the total number of external collaborators across your organization as a posture signal (medium)

---

## Prerequisites

- **GitHub organization** — Thalian connects at the org level, not the individual user level
- **Org owner account** — you must authorize Thalian as a GitHub org owner to grant the necessary OAuth scopes

---

## Connect via OAuth

1. Go to **Integrations** → **Browse**
2. Find **GitHub** and click **Connect**
3. Click **Authorize with GitHub**
4. Sign in to GitHub if prompted
5. Select the **organization** you want to connect — you must be an owner of the org to grant access
6. Review the requested permissions and click **Authorize**
7. You'll be redirected back to Thalian — the integration is now connected and the first sync begins

---

## Requested Permissions

| Scope | Justification |
|---|---|
| `read:org` | Reads org members, teams, and outside collaborators for identity sync |
| `write:org` | Enables remediation actions — removing outside collaborators or org members. Requested at initial connect but only used when an action is explicitly initiated or approved |
| `read:user` | Reads user profile data for IDP matching |
| `user:email` | Reads the authenticated user's email addresses for identity matching |

Thalian will only use write permissions when a remediation action is explicitly initiated or approved by your team.

---

## Connect via personal access token

If your organization restricts third-party OAuth apps, or you prefer to scope access through a dedicated machine account, you can connect GitHub with a classic personal access token instead of OAuth.

1. In GitHub, go to **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** and click **Generate new token**
2. Select the scopes in the table below
3. Generate the token and copy it (classic tokens start with `ghp_`)
4. In Thalian, go to **Integrations** → **Browse**, find **GitHub**, and click **Connect**
5. Switch the method toggle to **API Credentials**
6. Paste the token into **Personal Access Token**, and enter your organization login (the `org` slug in `github.com/<org>`) into **Organization**
7. Click **Connect** to save the credentials and start the first sync

The token must be created by an org owner, or a member with org admin rights, for org-level data and remediation to work.

### Token scopes (classic)

| Scope | Enables |
|---|---|
| `read:org` | Org members, owners, outside collaborators, and org security settings |
| `repo` | Private repositories, collaborator permissions, deploy keys, branch protection, Dependabot alerts, and secret scanning alerts |
| `admin:org_hook` | Org-level webhook inventory (optional) |
| `read:audit_log` | Org audit log events (GitHub Enterprise Cloud only; optional) |
| `admin:org` | Remediation actions such as removing an outside collaborator or org member. Only used when an action is explicitly initiated or approved |

- **Public-only orgs:** you can omit `repo`. Private repositories and the signals tied to them (deploy keys, branch protection, Dependabot, secret scanning) are skipped without it
- **Advanced Security signals:** secret scanning and Dependabot alerts require GitHub Advanced Security, or public repositories
- **Read-only connection:** omit `admin:org` if you do not want Thalian to perform any write actions. Sync and detection still work

### Fine-grained tokens

Fine-grained personal access tokens can be used for read-only sync. Grant the token access to the organization, then set Organization permissions of **Members** (read) and **Administration** (read), and Repository permissions of **Metadata** (read), **Administration** (read), **Dependabot alerts** (read), and **Secret scanning alerts** (read). For remediation actions and full org audit log access, use a classic token.

---

## What Thalian syncs

- **Organization members** — all current members with their role (member or owner), email address, and last activity
- **Outside collaborators** — external users with repository access, including which repos and their permission level (read, write, admin)
- **IDP gap detection** — each member and collaborator is cross-referenced against your connected IDP to identify accounts outside the corporate directory

GitHub usernames are matched to IDP accounts by email address. For members whose GitHub email is private or doesn't match their corporate email, Thalian attempts a username match as a fallback.

---

## Troubleshooting

- **Organization not listed during OAuth:** Ensure you are an owner of the org. Members cannot grant org-level OAuth access
- **Missing outside collaborators:** Confirm the `read:org` scope was granted — this scope is required to list collaborators
- **IDP gaps not detected:** Ensure at least one IDP (Okta, Entra ID, Google Workspace, JumpCloud, or OneLogin) is connected and synced. Thalian requires IDP data to perform the cross-reference
- **Remediation actions unavailable:** Re-authorize with `write:org` scope (OAuth), or reconnect with a token that includes `admin:org`
- **Connection fails with the token method:** Confirm the token is not expired, includes `read:org`, and that the **Organization** field matches your org login exactly (the slug from `github.com/<org>`, not the display name)
- **Private repos or security alerts missing with a token:** The token needs the `repo` scope. Secret scanning and Dependabot alerts also require GitHub Advanced Security

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
