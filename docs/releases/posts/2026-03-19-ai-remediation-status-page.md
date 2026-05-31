---
date: 2026-03-19
slug: ai-remediation-status-page
---

# AI-reasoned remediation, public status page, behavioral baselines, and more

AI remediation with reasoning, public status page, Layer 3 behavioral baselines, notification filtering, and major performance improvements.

<!-- more -->

## AI-reasoned remediation

For entities with multiple overlapping findings, ask Claude to analyze all open risks in context and propose a sequenced action plan with reasoning. Available from the Remediation page and from Identities via a one-click deep-link. Approval cards now show Claude's assessment — what the situation is, what risk it poses, and why the recommended actions are the right response.

## Public status page

Real-time platform health at [status.thalian.ai](https://status.thalian.ai) with per-service 90-day uptime bars, incident history, affected services chips, and email subscription for downtime alerts.

## Layer 3 behavioral baselines

Per-entity anomaly detection that builds individual baselines for users, devices, and apps and flags deviations from established patterns.

## Findings notification filtering

Alert notifications now fire only on new findings; recurring open findings no longer generate repeat alerts.

## Improvements

- **Findings page** — Each tab now includes a short descriptor explaining what the tab tracks
- **Causality Insights** — Redesigned compact card layout; remediation suggestions are now entity-specific
- **Performance** — Initial app bundle reduced by 69%; database queries parallelized for faster page load
- **Auto-sync reliability** — Integrations now sync in parallel, preventing worker timeouts on workspaces with many connected platforms
- Auth guards and workspace scoping added to 7 previously unguarded API endpoints

## Changes

- Status page moved from an in-app route to a standalone Cloudflare Worker, accessible without authentication

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-19)
