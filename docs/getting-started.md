# Getting Started with Thalian

Welcome to Thalian — an AI-powered IT intelligence platform that surfaces insights your IT team would otherwise miss by analyzing data across your identity providers, endpoint managers, security tools, and SaaS applications.

---

## Creating Your Workspace

1. **Sign up** at [app.thalian.ai/signup](https://app.thalian.ai/signup) using your work email or Google account
2. **Name your workspace** — this is your organization's Thalian environment. All data, integrations, and team members are scoped to this workspace
3. **Start your free trial** — Pro features are available for 30 days. After that, you can continue on the Free plan or upgrade

## Onboarding: Connect Your First Integration

After creating your workspace, the onboarding flow walks you through connecting your primary identity provider (IDP):

### Step 1: Choose Your IDP

Select the identity provider your organization uses:

- **Okta** — API token authentication
- **Microsoft Entra ID** (Azure AD) — OAuth or app registration
- **Google Workspace** — OAuth with admin consent
- **JumpCloud** — API key authentication
- **OneLogin** — Client ID and secret
- **PingOne** — OAuth 2.0 client credentials (Worker app)

### Step 2: Enter Credentials

- **OAuth platforms** (Google Workspace, Entra ID): Click "Connect" to authorize via your provider's consent screen. Thalian requests only read-only scopes needed for user directory, app inventory, and device management
- **API key platforms** (Okta, JumpCloud, OneLogin): Enter your API credentials. These are encrypted at rest — the plaintext is never stored

### Step 3: First Sync

Once connected, Thalian immediately syncs your data:

- **Identities:** User accounts, roles, MFA status, last login dates
- **Applications:** SaaS apps, SSO configuration, OAuth grants
- **Devices:** Managed endpoints, compliance status, encryption state (if MDM is connected)

The first sync typically takes 30-60 seconds depending on the size of your directory.

### Step 4: Review Results

After sync completes, you'll see a summary of what Thalian discovered:

- Number of identities synced
- Number of applications detected
- Initial findings (security risks, access gaps, compliance issues)

Click "Go to Dashboard" to start exploring.

## Understanding the Dashboard

The dashboard (`/home`) is your workspace command center. Here's what you'll see:

![The Thalian dashboard showing stat cards, MFA coverage, cross-platform signals, and integration health](./assets/screenshots/dashboard/dashboard-overview.png)

### Stat Cards (top row)

Five summary metrics updated after each analysis:

| Card | What It Shows |
|---|---|
| **Open Findings** | Total active findings across all categories |
| **Identities** | Total unique identities across all connected platforms |
| **Shadow IT** | Number of unvetted/unsanctioned applications detected |
| **Stale Accounts** | Users who haven't logged into any connected platform in 30+ days |
| **Security Posture** | Normalized composite score (0–100) based on severity-weighted findings. Uses a linear formula: raw score divided by a workspace-scaled ceiling, capped at 100 |

Each card shows a trend arrow comparing to the previous analysis run.

### MFA Coverage Donut

Visual breakdown of MFA enrollment across your identity base. Shows enabled vs. disabled counts, with a special warning if any admin accounts lack MFA.

### Findings by Severity

Bar breakdown of open findings by severity level (critical, high, medium, low). Click any severity to jump to the filtered Findings page.

### Integration Health

Status of your connected integrations: last sync time, error indicators, and sync health. Capped at 4 visible entries with a "View all" link.

### Recent Activity

Live feed of the latest 8 events across your workspace: syncs, findings resolved, team changes, remediation actions.

### AI Brief

An AI-generated summary of your workspace's security posture and notable changes since the last analysis. Ask follow-up questions to dig deeper into any area.

## What's Next?

- **[Connect more integrations](./integrations-guide.md)** — The more platforms you connect, the more cross-platform insights Thalian can surface
- **[Review your findings](./findings-and-remediation.md)** — Start with critical and high severity findings
- **[Set up your team](./settings-and-admin.md)** — Invite team members and assign appropriate roles
- **[Explore reports](./reports-and-audit.md)** — Track your security posture over time

---

*For information on connecting additional platforms, see [Integrations Guide](./integrations-guide.md).*
