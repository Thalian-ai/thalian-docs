# Connect BambooHR

Step-by-step guide to connecting BambooHR to Thalian for HR-driven access intelligence and terminated employee detection.

---

## What Thalian detects

BambooHR gives Thalian authoritative employment data — who is active, who has been terminated, their department, manager, and start date. Thalian cross-references this HR data against your connected identity providers, SaaS apps, devices, and security tools to surface access that should have been revoked.

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

- **BambooHR account** with admin access
- Your BambooHR **company subdomain** — the prefix in your BambooHR URL (e.g., `yourcompany` from `yourcompany.bamboohr.com`)

---

## Create an API key in BambooHR

Thalian connects using a BambooHR API key.

1. Sign in to your BambooHR admin account
2. Click your profile icon (top right) → **API Keys**
3. Click **Add New Key**
4. Give the key a name (e.g., `Thalian`) and click **Generate Key**
5. Copy the API key — it is shown only once

BambooHR API keys inherit the permissions of the generating user. Use an admin account to ensure Thalian can read all employee records including employment status and termination data.

---

## Connect in Thalian

1. Go to **Integrations** → **Browse**
2. Find **BambooHR** and click **Connect**
3. Enter your **company subdomain** (e.g., `yourcompany` — not the full URL)
4. Paste your **API Key**
5. Click **Connect** — Thalian validates the credentials and begins the first sync

---

## What Thalian syncs

- **All employees** — name, email, employment status (active, terminated, on leave), department, manager email, start date, and termination date
- **Terminated employees** — used to identify accounts that remain active in connected IDPs, SaaS apps, or device managers after offboarding

Thalian does not sync payroll data, compensation, benefits, time-off records, or any sensitive HR fields beyond employment status and identity attributes.

---

## How HR data powers cross-platform findings

BambooHR data alone doesn't generate findings — it becomes the authoritative termination signal that Thalian uses to query across every other connected platform. For example:

- If a BambooHR termination record exists for `jane@company.com`, Thalian checks whether her Okta account is still active, whether her laptop is still checking in to Jamf, whether she still has GitHub access, and whether her Salesforce account is still enabled.
- The more platforms connected alongside BambooHR, the more compound findings Thalian can surface.

---

## Remediation

BambooHR is a **read-only integration** — it is the HR source of truth, not a remediation target. All remediation actions for HR-triggered findings execute against the IDP or SaaS platform where the account remains active:

- Suspend the user in Okta, Entra ID, or Google Workspace
- Revoke device enrollment in Jamf, Intune, or Iru
- Revoke GitHub org membership or Salesforce access directly

---

## Troubleshooting

- **Invalid credentials:** Ensure the subdomain is entered without `https://` or `.bamboohr.com` — only the prefix (e.g., `yourcompany`)
- **Terminated employees not detected:** BambooHR API keys inherit the generating user's permissions. Confirm the user is an admin with access to employee status and termination fields
- **Compound findings not appearing:** Compound rules require at least 3 connected platforms. Connect an IDP (Okta, Entra, Google Workspace, JumpCloud, or OneLogin) and at least one additional platform (device manager, ITSM, cloud IAM) alongside BambooHR

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
