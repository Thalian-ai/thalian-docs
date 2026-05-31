---
date: 2026-03-29
slug: salesforce-integration
---

# Salesforce integration, Slack alert actions, Okta OAuth 2.0 upgrade

New Salesforce CRM integration, dismiss and snooze actions on Slack alerts, and Okta auth upgrade to OAuth 2.0 client credentials.

<!-- more -->

## Salesforce integration

Connect Salesforce to detect CRM access gaps between your Salesforce org and your corporate identity provider. Thalian syncs all active and inactive Salesforce users and cross-references against Okta, Entra ID, Google Workspace, JumpCloud, and OneLogin. Four new findings:

- **Salesforce admin not in IDP** (critical)
- **Salesforce user not in IDP** (high)
- **Stale Salesforce user** with suspended/deprovisioned IDP account but active CRM access (high)
- **Connected app authorized by unknown user** outside the IDP (medium)

Salesforce is read-only — Thalian uses `api refresh_token offline_access id` OAuth scopes only. No write permissions requested.

## Slack alerts: dismiss and snooze from Slack

Security alerts sent to Slack now include Dismiss and Snooze 7d buttons directly on every alert card. Both actions update the original Slack message in place with a confirmation line. All actions are signature-verified and written to the immutable audit log.

## Okta: upgraded to OAuth 2.0 client credentials

Okta sync and all 7 Okta remediation functions now authenticate using OAuth client credentials instead of a static SSWS API token. If you have Okta connected, you'll need to reconnect with your Okta Client ID and Client Secret. The new flow is more secure and handles token rotation automatically.

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-29)
