# Connect Rippling

Step-by-step guide to connecting Rippling to Thalian for HR-driven access intelligence and terminated employee detection.

---

## What Thalian detects

Rippling gives Thalian authoritative employment data — who is active, who has been terminated, their department, manager, and start date. Thalian cross-references this HR data against your connected identity providers, SaaS apps, devices, and security tools to surface access that should have been revoked.

**Base HR rules (6):**

| Finding | Severity |
|---|---|
| Terminated employee still active in IDP | Critical |
| Offboarding delay — access not revoked within policy window | High |
| Ghost employee — IDP account with no HR record | Medium |
| Department drift — IDP department doesn't match HR record | Low |
| Manager mismatch — IDP manager doesn't match HR record | Low |
| New hire with no IDP account after grace period | Medium |

**Compound rules — require 3+ connected platforms (7):**

| Finding | Severity |
|---|---|
| Terminated employee retains admin entitlements in SaaS apps | Critical |
| Terminated employee has active managed device | High |
| Terminated employee on device with active threat | Critical |
| New hire has no managed device after onboarding window | Medium |
| Terminated employee has active mailbox forwarding rule | High |
| Terminated employee has active AI tool access | High |
| Terminated employee retains ITSM admin role | High |

---

## Prerequisites

- **Rippling account** with admin access

---

## Option 1 — Connect via OAuth (recommended)

1. Go to **Integrations** → **Browse**
2. Find **Rippling** and click **Connect**
3. Click **Authorize with Rippling** — you'll be redirected to Rippling
4. Sign in and approve read-only access to employee data
5. You'll be redirected back to Thalian — the integration is now connected

Thalian requests the following OAuth scopes:

| Scope | Justification |
|---|---|
| `employee:read` | Reads employee profiles, employment status, department, manager, and termination dates |
| `company:read` | Reads company structure for organizational context |

---

## Option 2 — Connect via API key

If your organization restricts OAuth, you can connect using an API key instead.

1. Sign in to your Rippling admin account
2. Go to **Settings** → **Developer Settings** → **API Keys**
3. Click **Create API Key**
4. Give the key a name (e.g., `Thalian`)
5. Set permissions to **Read Only** for:
   - Employees
   - Employment details (status, department, manager, start/termination dates)
6. Click **Create** and copy the API key — it is shown only once

### Connect in Thalian

1. Go to **Integrations** → **Browse**
2. Find **Rippling** and click **Connect**
3. Select **API Key** as the connection method
4. Paste your **API Key**
5. Click **Connect** — Thalian validates the credentials and begins the first sync

---

## What Thalian syncs

- **All employees** — name, email, employment status (active, terminated, on leave), department, manager email, start date, and termination date
- **Terminated employees** — used to identify accounts that remain active in connected IDPs, SaaS apps, or device managers after offboarding

Thalian does not sync payroll data, compensation, benefits, or any sensitive HR fields beyond employment status and identity attributes.

---

## How HR data powers cross-platform findings

Rippling data alone doesn't generate findings — it becomes the authoritative termination signal that Thalian uses to query across every other connected platform. For example:

- If a Rippling termination record exists for `jane@company.com`, Thalian checks whether her Okta account is still active, whether her laptop is still checking in to Jamf, whether she still has GitHub access, and whether her Salesforce account is still enabled.
- The more platforms connected alongside Rippling, the more compound findings Thalian can surface.

---

## Remediation

Rippling is a **read-only integration** — it is the HR source of truth, not a remediation target. All remediation actions for HR-triggered findings execute against the IDP or SaaS platform where the account remains active:

- Suspend the user in Okta, Entra ID, or Google Workspace
- Revoke device enrollment in Jamf, Intune, or Iru
- Revoke GitHub org membership or Salesforce access directly

---

## Troubleshooting

- **Invalid API key:** Ensure the key was copied without extra whitespace and has not been rotated or revoked in Rippling
- **Terminated employees not detected:** Confirm the API key has read access to employment status and termination date fields
- **Compound findings not appearing:** Compound rules require at least 3 connected platforms. Connect an IDP (Okta, Entra, Google Workspace, JumpCloud, or OneLogin) and at least one additional platform (device manager, ITSM, cloud IAM) alongside Rippling

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
