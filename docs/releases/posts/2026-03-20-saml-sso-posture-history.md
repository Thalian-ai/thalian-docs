---
date: 2026-03-20
slug: saml-sso-posture-history
---

# SAML 2.0 SSO, Security Posture score history, AI context upgrades, and more

Enterprise SAML SSO, posture score sparklines with trend deltas, status page issue reports, in-app feedback, and major AI assistant context improvements.

<!-- more -->

## SAML 2.0 SSO for enterprise workspaces

Enterprise plan workspaces can now configure SAML 2.0 single sign-on directly from Settings. Admins provide their IdP metadata URL and email domain; Thalian registers the SAML connection and displays the ACS URL and Entity ID. Supports SP-initiated and IdP-initiated login. SSO users are automatically provisioned into the correct workspace on first sign-in.

## Security Posture score history and sparkline

The Security Posture stat on the dashboard now shows a live sparkline trend line (up to 30 data points) and a delta indicator comparing your current score to the previous analysis run. The AI assistant also has access to the full posture trend history.

## Status page issue report notifications

When a user submits a report via the "Report an Issue" form on the status page, two emails are now triggered: an internal notification to support and a confirmation email to the reporter.

## In-app feedback form

A "Give feedback" option is now available in the bottom-left user menu. Submit bug reports, feature requests, or general feedback without leaving the app.

## AI assistant context upgrades

- **Remediation history** — Full visibility into remediation actions taken in the last 30 days, including pending approvals and Claude's prior reasoning
- **Richer entity data** — Findings linked to named users/apps, device detail includes OS platform, applications include category, admin accounts include department
- **Finding descriptions** — Full description of each open finding, enabling plain-language explanations
- **Stale access analysis** — Identifies entitlements where the app hasn't been used in 90+ days
- **What-if simulation tool** — Answer "what would happen if I suspended this user?" by running live simulation

## Security Posture score redesign

The "Risk Score" stat has been replaced with a unified **Security Posture** score (0-100, higher is better). Uses sigmoid normalization so a single critical finding doesn't catastrophically tank the number. Displayed with a letter grade (A-F) and color-coded.

## Fixes

- **Slack alert deduplication** — Two compounding bugs causing alerts to re-fire on every sync are now fixed. Dedup keys use stable external identifiers, and email app sync only removes genuinely absent apps.
- **Integration sync failures on status page** — Failed syncs now correctly flip integration status to `error`, which the status page health check reads. Integrations that haven't synced in 10+ hours are flagged as degraded.

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-20)
