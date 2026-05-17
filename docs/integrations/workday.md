# Connect Workday

Step-by-step guide to connecting Workday to Thalian for HR-driven access intelligence and terminated employee detection.

---

## What Thalian detects

Workday gives Thalian authoritative employment data — who is active, who has been terminated, their department, manager, and start date. Thalian cross-references this HR data against your connected identity providers, SaaS apps, devices, and security tools to surface access that should have been revoked.

Workday shares the same HR detection rules as Rippling and BambooHR:

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

- **Workday tenant** with REST API access enabled
- **Integration System User (ISU)** in Workday with read access to Worker data, or an API client configured for OAuth

---

## Connect via API token

Thalian connects to Workday using a REST API token scoped to your tenant.

1. Sign in to your Workday tenant as an admin
2. Create an **Integration System User (ISU)** with read-only access to:
   - Worker data (name, email, employment status, department, manager, start/termination dates)
3. Generate a **REST API token** for the ISU, or note the ISU username and password
4. Note your **Tenant URL** (e.g., `https://wd5-impl-services1.workday.com`) and **Tenant Name** (e.g., `your_company`)

### Connect in Thalian

1. Go to **Integrations** → **Browse**
2. Find **Workday** and click **Connect**
3. Enter your **Tenant URL**, **Tenant Name**, and **API Token** (or ISU credentials)
4. Click **Connect** — Thalian validates the credentials and begins the first sync

---

## What Thalian syncs

- **All workers** — name, email, employment status (Active, Terminated, Leave of Absence), department, title, manager email, hire date, and termination date
- **Terminated workers** — used as the authoritative signal to identify accounts that remain active in connected IDPs, SaaS apps, or device managers after offboarding

Thalian does not sync payroll data, compensation, benefits, time-off balances, or any sensitive HR fields beyond employment status and identity attributes.

---

## How HR data powers cross-platform findings

Workday data alone doesn't generate findings — it becomes the authoritative termination signal that Thalian uses to query across every other connected platform. For example:

- If a Workday termination record exists for `jane@company.com`, Thalian checks whether her Okta account is still active, whether her laptop is still checking in to Jamf, whether she still has GitHub access, and whether her Salesforce account is still enabled.
- The more platforms connected alongside Workday, the more compound findings Thalian can surface.

---

## Remediation

Workday is a **read-only integration** — it is the HR source of truth, not a remediation target. All remediation actions for HR-triggered findings execute against the IDP or SaaS platform where the account remains active:

- Suspend the user in Okta, Entra ID, or Google Workspace
- Revoke device enrollment in Jamf, Intune, or Iru
- Revoke GitHub org membership or Salesforce access directly

---

## Troubleshooting

- **Connection failed:** Ensure the Tenant URL includes the full hostname (e.g., `https://wd5-impl-services1.workday.com`) and the Tenant Name matches your Workday configuration
- **No workers synced:** Confirm the ISU or API token has read access to the `/workers` endpoint
- **Terminated employees not detected:** Verify that terminated worker records are accessible via the API and include termination date
- **Compound findings not appearing:** Compound rules require at least 3 connected platforms. Connect an IDP and at least one additional platform alongside Workday

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
