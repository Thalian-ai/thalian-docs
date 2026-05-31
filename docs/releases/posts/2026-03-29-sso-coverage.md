---
date: 2026-03-29
slug: sso-coverage
---

# SSO coverage per identity, login source attribution, and more

Per-identity SSO vs direct-auth app breakdown, login authority scoring, 7 new SSO coverage findings, identity type filtering, and sync freshness before AI analysis.

<!-- more -->

## SSO coverage per identity

The Identities page now shows how many of each user's apps are SSO-managed vs direct-auth. Hover over the app count to see the breakdown (e.g., "3 SSO-managed, 2 direct auth"). When an IDP is connected and a user has direct-auth apps, the count highlights in purple to flag the gap.

## Login source attribution

The Last Login column now prefers IDP login data over SaaS timestamps. Hover over any login timestamp to see which platform it came from (e.g., "Last seen in Okta"). If all apps are SSO-covered, the tooltip says so; if some bypass SSO, it tells you how many.

## 7 new SSO coverage findings

Five identity-level rules detect SSO gaps:

- **Admin with direct-auth apps** (critical)
- **Executive with direct-auth apps** (critical)
- **Offboarded user with direct-auth apps** that SSO deprovisioning can't reach (critical)
- **Majority of apps bypassing SSO** (high)
- **New hire provisioned outside the IDP** workflow (medium)

Two behavioral anomaly rules detect spikes in direct-auth app grants and off-hours activity on non-SSO apps.

## Identity type filtering

The Identities page now has an All / Users / Service & Shared toggle. Service accounts (shared inboxes, bots, automation accounts) are identified using a 40+ prefix list and separated from human users in the view.

## Sync engine: stale-before-AI

AI Brief and AI Chat now trigger a background sync for any integration that hasn't synced recently before generating content, ensuring responses reflect current posture. Concurrent requests share the same in-flight sync to avoid redundant API calls.

## Improvements

- **MFA accuracy** — MFA findings now only fire when an IDP confirms MFA is disabled. SaaS-only workspaces no longer see false-positive "No MFA" findings
- **MFA column clarity** — When no IDP is connected, the MFA column shows a dash with a hover tooltip
- **AI chat SSO context** — The AI assistant now includes per-admin SSO coverage and workspace-level SSO stats
- **Disconnect cleanup** — Removing an integration now cancels all pending approval actions tied to that integration's entities
- **Audit timestamp precision** — Remediation actions and settings audit entries now show absolute timestamps
- **AI chat deep links** — AI responses now include direct links to relevant admin consoles
- **Status page: immediate incident creation** — Incidents created on the first degraded check rather than waiting for recovery

## Fixes

- **Signup workspace naming** — Personal email signups now get "Personal Workspace" instead of the email domain

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-29)
