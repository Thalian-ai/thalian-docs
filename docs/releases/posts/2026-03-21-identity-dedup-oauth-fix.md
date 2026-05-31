---
date: 2026-03-21
slug: identity-dedup-oauth-fix
---

# Cross-platform identity context, one finding per person, and OAuth scope accuracy

Major identity intelligence upgrade: findings now include cross-platform context, deduplicate to one finding per person, and OAuth scope interpretation is now precise.

<!-- more -->

## Cross-platform identity context in findings

Findings that affect a user active across multiple identity providers now say so explicitly. For example, "Tori Ferrante is active across Google Workspace and Okta but is logging in from a device not enrolled in your MDM." The finding detail panel also shows teal platform badges on the affected entity chip.

## One finding per person, not per platform record

The intelligence engine now understands that the same email address across multiple integrations represents the same human being. Findings like "Active users without managed devices" now generate a single finding per person with full cross-platform context, rather than one finding per integration record. The underlying identity data model now has three distinct tiers: authoritative IDP only, raw all-platform records, and a new canonical per-person layer.

## Fixes

- **OAuth scope false positives eliminated** — The basic OIDC `email` scope was incorrectly being matched as a substring of "mail," flagging apps like Figma and Slack as having inbox access. A centralized scope interpretation engine now tests each OAuth scope individually with precise patterns. Only explicit grants like `gmail.readonly` or `https://mail.google.com/` imply inbox access.
- **Shadow IT findings no longer duplicated** — Apps already covered by the more specific sensitive-scope finding are now excluded from the general unvetted apps finding.
- **Impact analysis scores now match Security Posture** — The "Current" and "After action" scores in the simulate panel now use the same sigmoid-normalized formula as the dashboard.

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-21)
