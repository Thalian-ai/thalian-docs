---
date: 2026-03-25
slug: drift-signals-auto-remediation
---

# Cross-platform drift signals, automatic remediation after sync, and more

Three new drift signal rules requiring multiple data sources, automatic remediation firing after every sync, AI risk summaries on identity detail, and OAuth scope risk labels.

<!-- more -->

## Three new cross-platform drift signal rules

- **SSO coverage declining** — Fires when the proportion of apps authenticating via SSO is trending downward. Includes breach date projection and top non-SSO apps. Requires an IDP and app discovery data.
- **Termination-to-access-removal lag growing** — Measures average time between HR termination and IDP suspension across recent offboardings. Fires when lag exceeds 2 days across 3+ events, and flags whether the lag is worsening. Surfaces terminated employees who still have enrolled devices. Requires HR and an IDP; MDM data incorporated when available.
- **Ghost identity growth** — Detects users being added to SaaS tools without a corresponding IDP account. Fires when 3+ such users appear in 60 days, or when the rate is accelerating. Requires an IDP and at least 2 connected SaaS platforms.

## Automatic remediation after every sync

Workspaces with auto-remediation enabled now have eligible findings actioned immediately after each scheduled sync. Safe actions (create ticket, notify user, sanction app) execute automatically; higher-risk actions (suspend user, revoke OAuth token, block app) are queued for admin approval with email notification. All agentic actions appear in the Remediation page under the "Automatic" filter tab.

## AI Risk Summary on identity detail

Opening any identity with open findings now shows a "Risk Summary" block powered by Claude — a concise narrative covering their risk score, MFA status, app access breadth, device compliance, and blast radius.

## OAuth scope risk labels on unsanctioned apps

The Applications page Unsanctioned tab now shows inline scope risk chips (e.g. "Writes email", "Reads calendar files") sourced from server-side OAuth scope interpretation.

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-25)
