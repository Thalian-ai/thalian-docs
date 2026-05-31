---
date: 2026-03-30
slug: workday-compliance-export
---

# Workday HR integration, compliance evidence export, and more

New Workday HR integration, compliance audit evidence export for Enterprise, a redesigned integration library, and a batch of display and gating fixes.

<!-- more -->

## Workday HR integration

Connect Workday to cross-reference employee lifecycle data against your identity providers and SaaS access. Thalian syncs active and terminated workers from Workday and detects terminated employees who still have active IDP accounts, SaaS entitlements, or managed devices. Joins with Okta, Entra ID, Google Workspace, JumpCloud, OneLogin, Intune, Jamf, and all connected SaaS platforms. Read-only — uses Workday's REST API with basic auth credentials. Adds to the existing HR intelligence layer alongside Rippling and BambooHR.

## Compliance: evidence export (Enterprise)

The Compliance page now includes PDF and Excel evidence pack export for Enterprise workspaces. Pro users see a locked "Export evidence" button with an upgrade prompt. The export includes control status, mapped rules, open findings, and a timestamp — formatted for handing directly to an auditor.

## Compliance: audit log tab

A dedicated Audit Log tab is now available on the Compliance page, showing a filterable, searchable feed of all user and system actions. Pro workspaces see 30-day history with CSV export; Enterprise sees 1-year retention.

## Integration library redesign

The integration browser has a new card layout and filter bar. Cards now show category, connection status, and sync stats. Filter by category (Identity, Device, HR, Cloud, etc.) to find what you need faster.

## Clickable stat pills on integrations

Stat pills on connected integration cards (e.g. "42 identities", "7 findings") now navigate directly to the relevant filtered view — Identities, Applications, Devices, or Findings — scoped to that integration.

## Improvements

- **Plan tier copy** — Billing and upgrade pages now accurately reflect what each plan includes, with detailed Compliance and Security & Governance categories for Pro and Enterprise respectively
- **Compliance tab gating** — All three tabs on the Compliance page now correctly require Pro or Enterprise (previously bypassable via direct URL)

## Fixes

- **Display labels** — Raw internal identifiers no longer appear in the UI. Action types, remediation statuses, compliance statuses, and audit event types are now shown as readable labels everywhere
- **Audit log retention** — Now correctly shows 90-day history for Pro workspaces
- **Pricing constants** — Billing page pricing sourced from a single constant
- **Compliance deep links** — "View findings" links inside expanded compliance controls now navigate correctly
- **AI chat MFA/login accuracy** — The AI assistant no longer reports MFA or login status for platforms that don't expose that data

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-30)
