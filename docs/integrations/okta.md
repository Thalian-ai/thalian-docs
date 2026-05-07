# Connect Okta

Step-by-step guide to connecting Okta to Thalian for identity and access intelligence.

---

## Prerequisites

- **Okta admin account** with permission to create applications
- **Okta domain** — your Okta org URL (e.g., `yourcompany.okta.com`)

## Create an API Services app in Okta

Thalian connects using Okta's OAuth 2.0 client credentials flow. You'll need to create a service app in Okta that grants read-only API access.

1. Sign in to your Okta admin console
2. Go to **Applications** → **Applications**
3. Click **Create App Integration**
4. Select **API Services** and click **Next**
5. Give the app a name (e.g., `Thalian`) and click **Save**

## Grant the required OAuth scopes

After creating the app:

1. Go to the **Okta API Scopes** tab on your new app
2. Grant the following scopes:

| Scope | Purpose |
|-------|---------|
| `okta.users.read` | Sync users, status, and MFA enrollment |
| `okta.apps.read` | Sync assigned applications |
| `okta.groups.read` | Sync group memberships |
| `okta.logs.read` | Read system log events |
| `okta.iam.agents.read` | Sync AI agent principals (required for NHI governance) |

3. Click **Grant** for each scope

!!! note "okta.iam.agents.read scope"
    The `okta.iam.agents.read` scope is required to pull AI agent identities from Okta's AI Agents feature. Without this scope, Thalian will sync users, apps, groups, and logs normally — but AI agent identities will not appear in the Identities page and NHI governance rules will not fire. You can add this scope to an existing connection without reconnecting: open the Okta API Scopes tab on your Thalian service app and click **Grant** next to `okta.iam.agents.read`.

## Copy your credentials

1. Go to the **General** tab of your app
2. Copy the **Client ID** from the Client Credentials section
3. Click **Generate new client secret** and copy the value — it is only shown once

## Connect in Thalian

1. Go to **Integrations** → **Browse**
2. Find **Okta** and click **Connect**
3. Enter your **Okta domain** (e.g., `yourcompany.okta.com`)
4. Paste your **Client ID** and **Client Secret**
5. Click **Connect** — Thalian validates the credentials and begins the first sync

## What Thalian syncs

- **Users** — full directory including status, last login, and profile attributes
- **Groups** — group memberships and assignments
- **MFA status** — enrolled factors per user
- **Apps** — assigned applications and provisioning status
- **System log events** — authentication events, admin actions, and policy changes

## Okta security configuration analysis

Beyond user and access data, Thalian fetches and analyzes Okta's org-level security configuration after each sync:

- **ThreatInsight** — whether IP-based threat intelligence is enabled and its enforcement mode
- **MFA enrollment policies** — which authenticator types are required vs. optional, and which users are excluded
- **Password policies** — minimum length, complexity, and history requirements per group
- **API token hygiene** — active long-lived API tokens that should be rotated or scoped down
- **Session settings** — session lifetime, persistent cookie settings, and idle timeout configuration
- **Network zones** — defined trusted zones vs. unrecognized origin checks

This configuration data powers 14 Okta-specific detection rules — for example, firing when ThreatInsight is in audit-only mode (logging threats but not blocking them), when an MFA policy excludes high-privilege groups, or when admin accounts have long-lived API tokens. The AI assistant also uses this context to answer questions about your Okta security posture.

No additional OAuth scopes are required — all configuration data is accessible with the four scopes granted during initial setup.

## Okta AI Agents and NHI governance

Thalian syncs AI agent principals from Okta's AI Agents feature — the identities Okta creates when you connect agent-based AI services (LangChain apps, CrewAI agents, Gumloop automations, n8n AI nodes, and others) to your Okta org. These are distinct from human users and traditional service accounts: they have declared OAuth scopes, no password, and a client credentials grant flow.

### What Thalian does with AI agents

- **Classified as a first-class identity tier**: AI agents appear in the Identities page with a purple **AI Agent** badge and a dedicated **AI Agents** tab. They are excluded from MFA, SSO enforcement, off-hours, and behavioral anomaly rules — agents run 24/7 by design and those rules produce false positives when applied to non-human principals.
- **Excluded from identity quota**: AI agents do not count against your plan's identity limit.
- **Visible metadata**: The identity detail panel shows the agent's declared OAuth scopes (the API permissions it was granted), client ID, grant type, and the human who created it (if available from Okta).
- **Orphan detection**: If no creating human is recorded, the panel shows a warning. This surfaces agents that were provisioned without accountability — no named owner means no offboarding process when the agent is decommissioned.

### AI agent findings

Two findings govern the NHI population:

- **Possible AI agent unclassified** (medium): A service account matches AI framework naming patterns but hasn't been formally classified as an AI agent. The finding prompts an admin to classify it via the Account type dropdown in identity detail — classifying resolves the finding and applies NHI-specific governance.
- **AI agent count growing** (medium, compound risk): Active AI agents represent more than 20% of the human workforce. Unreviewed agent proliferation creates persistent access that outlives the humans who provisioned it, with no offboarding process. The finding lists all active agents and links to the access review flow.

Both findings map to SOC 2 CC6.1/CC6.2, ISO 27001 A.5.15/A.5.18, and NIST CSF 2.0 PR.AA-01.

### Classification from inside Thalian

If Okta doesn't classify a principal as an AI agent at the API level (for example, a service account created outside Okta's AI Agents flow that is actually running an AI workload), admins can classify it manually. Open the identity detail panel, find the **Account type** dropdown, and select **AI Agent**. This applies the same NHI-specific governance rules as a platform-tagged agent.

## Troubleshooting

- **Invalid credentials:** Ensure the Client ID and Secret were copied correctly and the app has not been deactivated
- **Missing data:** Confirm all five OAuth scopes have been granted on the app's Okta API Scopes tab — including `okta.iam.agents.read` if you expect AI agent identities
- **No AI agents appearing:** Verify `okta.iam.agents.read` is granted. If the scope is missing, Thalian logs a warning on the sync result but continues syncing users and apps normally.
- **Rate limiting:** Thalian respects Okta's rate limits automatically. If syncs are slow, this may indicate heavy API usage on your Okta org

---

*For a full list of supported platforms, see [Integrations Guide](../integrations-guide.md).*
