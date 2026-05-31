---
date: 2026-03-31
slug: entra-conditional-access
---

# Entra ID Conditional Access detection, Okta System Log correlation, and more

Three major detection upgrades: Entra ID Conditional Access policy analysis, Okta System Log correlation rules, and expanded GCP IAM privilege analysis.

<!-- more -->

## Entra ID: Conditional Access policy detection

Thalian now fetches and analyzes your Entra ID Conditional Access policies automatically after each sync (requires re-authorizing the Microsoft connection to grant `Policy.Read.All`). Three new detection rules fire when CA policies are available:

- **MFA policy in report-only mode** — logs violations but never blocks
- **Disabled MFA policy** — potential regression if previously enforced
- **Admin accounts excluded from all MFA-requiring CA policies** — critical gap

The AI assistant also gains a Conditional Access context block and can answer questions about which policies are enforced vs report-only, whether MFA is actually blocking sign-ins, and which admins aren't covered. Existing Entra connections without the new scope continue working — CA rules stay silent until re-auth.

## Okta System Log correlation

Three new Okta-specific detection rules now use System Log data to surface credential and authentication risks that event-by-event inspection misses:

- **Failed MFA spike** — 5+ failed MFA challenges per user in the sync window (potential credential stuffing)
- **MFA factor disabled** — user or admin disabled an MFA factor (elevated severity for admin accounts)
- **User-reported compromise** — user clicked "This wasn't me" in Okta (highest-confidence indicator of active account takeover)

Okta System Log has been synced since launch; these rules make that data actionable without any new connection steps.

## GCP IAM privilege analysis

GCP IAM now detects 4 new privilege and configuration risks beyond IDP gap detection:

- Owner role sprawl per project
- Service accounts with admin-level roles
- Users with admin access across 3+ projects
- Systemic over-provisioning (>50% of users at Editor or higher)

These rules fire even when Google Workspace is the IDP.

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-31)
