---
date: 2026-06-06
slug: cross-platform-agent-governance
---

# Cross-platform AI agent governance: declared vs. observed, joined across platforms

Thalian already told you which AI agents exist in your environment. Now it governs them: what each agent is authorized to do, what it actually does, and the drift between the two, correlated across every platform the agent touches.

<!-- more -->

## The gap this closes

Detecting that an AI agent exists is the easy part. The question that matters for governance is the one an inventory does not answer: is this agent operating inside its mandate? A `crewai-finance-bot` that was approved for read-only ledger reconciliation should not be initiating transfers. An agent approved on Okta should not quietly show up acting on GitHub next week. And when an agent's behavior changes, you need to know whether someone changed its authorization three days earlier or whether it drifted on its own.

This release adds the six layers that answer those questions.

## Declared authorization

Every confirmed AI agent can carry a **governance record**, set from its detail panel under **Inventory -> Non-human**:

- **Declared purpose** -- a plain-language statement of what the agent is for.
- **Authorized scope categories** -- from a normalized vocabulary (`read:data`, `write:identity`, `financial`, `admin:org`, and so on) that maps platform-specific OAuth scopes, IAM actions, and audit event types into one comparable set.
- **Autonomy level** -- `human in loop` (proposes, a person approves each action), `human on loop` (acts, a person can override), or `fully autonomous`.
- **Business owner**, **review cadence**, and a hard **authorization expiry** that is separate from the soft review reminder.

The record is the artifact a governance body asks for. An agent without one now surfaces a finding instead of sitting silently in your inventory.

## Observed behavior

Each analysis run builds a weekly behavioral baseline per agent from its activity: which scope categories it actually invoked, how many distinct accounts it touched, which platforms it was active on, what share of its activity came from datacenter IP ranges, and the PII categories its scopes imply. That observed picture is what Thalian compares against the declared record.

## Drift detection

When authorized and observed diverge, Thalian flags it:

- **Scope exceeded** -- the agent used a scope category outside its authorized set. Critical when the extra scope is financial, identity-write, credential-write, or org-admin.
- **Behavioral drift** -- a 3x-or-more jump in activity, reach, or API volume against the agent's own four-week baseline.
- **New platform expansion** -- the agent appeared on a system it was never active on, with a higher-priority variant when the new activity originates from datacenter IPs (the signature of programmatic reach rather than a reviewed integration).
- **Authorization expired** and **access review overdue** -- the agent is still operating past its expiry or its scheduled review.
- **Agent acting on another agent** -- one AI agent acting on a second agent's identity or endpoint, an unreviewed trust chain between non-human identities.
- **Agent showing human signals** -- a confirmed agent with failed password attempts, browser sessions, or geo-diverse logins. That means either a stolen agent credential or a human operating under a service account to dodge per-user controls.

## Risk tiering and least privilege

Every agent gets an auto-computed **risk tier** from its scopes, autonomy level, and platform reach, shown as a badge on its detail panel. A high-risk agent on an infrequent review cadence raises a finding on its own. And once an agent has a few weeks of history, a **least-privilege advisory** points out scope categories it is authorized for but has never actually used, so you can trim the grant.

## One agent, many platforms

The same logical agent often appears as several platform identities: a Slack bot, an Okta agent, and a GitHub App that are really one thing. A **Platform legs** section on the agent's detail panel links them, with heuristic suggestions (matched on shared OAuth client IDs, name similarity, and scope overlap) that an admin confirms or dismisses.

## Why did it change?

When you open the causality view on a drift finding, Thalian now lays the agent's weekly trajectory next to the recent governance and configuration changes in the same window, so the root-cause analysis can point to the scope grant or integration that preceded the drift, instead of guessing.

## Compliance

Two AI-specific framework tabs join SOC 2, ISO 27001, NIST CSF 2.0, and ISO 42001 on the **Compliance** page: **NIST AI RMF (GOVERN)** and the **EU AI Act**. The agent governance findings map to GOVERN functions 1.1, 2.2, 2.4, and 5.1 and to EU AI Act articles 9, 10, 13, and 14, and both score over time on the Compliance Trend chart.

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/v2026.06)
