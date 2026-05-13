---
date: 2026-05-12
slug: compliance-frameworks
---

# Compliance frameworks — NIST CSF 2.0 and ISO 42001 mapped, Trend chart plots all four

The Compliance page now covers four frameworks side-by-side: SOC 2, ISO 27001, NIST CSF 2.0, and ISO 42001. The Compliance Trend chart plots all four framework scores over time, so security teams can see at a glance which posture is moving and which framework is dragging.

<!-- more -->

## What shipped

Two new frameworks join SOC 2 and ISO 27001 as tabs on the Compliance page:

- **NIST CSF 2.0** — Cybersecurity Framework 2.0. Six controls: **PR.AA-01** (privileged access and non-human identity lifecycle — the canonical NHI control), **PR.AA-03** (authentication strength), **PR.AA-05** (access reviews covering users, services, and hardware), **ID.AM-01** (asset inventory), **ID.AM-05** (resource criticality), and **DE.CM-03** (activity monitoring). CSF 2.0 explicitly extended PR.AA to cover "users, services, and hardware" — making it the canonical framework for AI agent and NHI governance.
- **ISO 42001** — the first international AI management system standard (ISO/IEC 42001:2023). Seven Annex A controls: **A.4.2** (AI resource documentation), **A.6.2.2** (AI system requirements and specification — the sanctioning gate for AI tools entering the org), **A.6.2.6** (AI system operation and monitoring), **A.6.2.8** (AI system event logs — backed by the immutable audit log), **A.7.3** (data acquired by AI), **A.9.2** (responsible use processes including offboarding lifecycle), and **A.10.3** (third-party AI suppliers).

## Why ISO 42001 matters

ISO 42001 is the cleanest framework to point to when a customer or regulator asks how AI is governed in production. It covers the operational surface where shadow AI, ungoverned AI tool grants, and AI agent sprawl create real exposure — exactly where Thalian's detection rules already operate. Run the Compliance page filtered to ISO 42001 before an AI risk assessment and the failing controls name the specific tools, agents, or data flows driving the gap. That list becomes the input for your AI inventory and risk register.

Coverage today maps to Thalian's existing AI governance detections:

- **A.4.2 + A.6.2.2** — Unsanctioned AI tool adoption and unclassified machine identities matching known agent framework patterns
- **A.6.2.6** — Workforce-ratio drift in the AI agent population, and AI tools granted write access to corporate data
- **A.7.3** — OAuth scopes granting AI tools read or write access to corporate data
- **A.9.2** — Terminated employees with active AI tool grants, and widespread unsanctioned AI adoption
- **A.10.3** — Third-party AI vendors operating outside any supplier management process

## Compliance Trend chart plots all four frameworks

The Compliance Trend chart on the Compliance page now plots SOC 2, ISO 27001, NIST CSF 2.0, and ISO 42001 scores side-by-side over time. Each framework writes its own score column to `drift_snapshots` on every analysis run, so the trend resolves automatically as new snapshots accumulate. Hovering any point shows all four framework scores for that date, making it easy to spot which framework is moving and when.

If you only see two lines today, that is expected — NIST CSF 2.0 and ISO 42001 scores start being recorded from this release forward. Historical gaps render as line breaks until enough post-deploy snapshots fill in.

## Availability

The Compliance page is available on **Pro and Enterprise** plans. All four framework tabs are visible to anyone with the **Security Analyst**, **Admin**, **Super Admin**, or **Auditor** role.

[View full Compliance documentation](../../compliance.md) · [View on GitHub](https://github.com/thalian-ai/thalian/releases/tag/v2026.05)
