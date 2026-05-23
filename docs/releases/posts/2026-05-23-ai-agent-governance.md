---
date: 2026-05-23
slug: ai-agent-governance-problem
---

# The AI agent governance problem nobody has tools for yet

Your IT stack was designed around humans. Your AI agents are not humans. Here is what that gap actually looks like in production, and the control framework that names it.

<!-- more -->

A few weeks ago, an engineer at a 140-person SaaS company authorized Claude with `gmail.readonly` scope using their personal Google account. They were prototyping a project that summarized their inbox. The OAuth grant lived inside their corporate Google Workspace via their personal Gmail recovery flow. They never used the corporate SSO. The grant was active for 23 days before anyone noticed. By then, the agent had read 4,200 emails. The Okta admin console showed nothing. The Intune device was compliant. The CrowdStrike sensor was healthy.

None of those tools were lying. They were just looking at the wrong thing.

This is the AI agent governance problem. It is not a hypothetical, it is not a 2027 problem, and it is not solved by your existing IAM tools.

---

## Why this is a new problem

Three things changed in the last 12 months that combined to create this gap.

The first is the **explosion of non-human identities** (NHIs). Service accounts, bots, API integrations, and now AI agents. Most enterprises now have more NHIs than humans. A typical mid-market customer of ours has between 2x and 5x more service accounts than employees. None of those NHIs go through the same lifecycle controls humans do, because the controls were designed for humans.

The second is **AI agents as a specific class of NHI**. Tools like LangChain, CrewAI, Devin, Gumloop, n8n, and Claude Computer Use create persistent identities that act on behalf of a human but are not the human. They have credentials. They have OAuth grants. They authenticate against your IdP and your SaaS apps. They run 24 hours a day. Most of them were created in the last 18 months.

The third is **board-level pressure**. Every IT and security leader has been asked some version of *"what's our AI risk?"* in a board meeting since the start of 2026. The leader knows the answer is "we are not really sure yet" and that the honest answer is "we cannot see most of it." Pressure plus a real visibility gap plus emerging frameworks (NIST CSF 2.0, ISO 42001) equals a category that did not exist 18 months ago.

---

## The four shapes of the problem

We see this surface four ways in real production environments. Each is a real finding that Thalian's detection engine generates, with the rule_id named so you can verify against the source.

### Identity sprawl without lifecycle ownership

AI agents are often created by individual engineers experimenting with new tooling. The agent gets a service account, an OAuth grant, or both. No one assigns an owner. No one documents what the agent does. The agent runs. The engineer leaves the company. The agent keeps running.

A real finding from one of our workspaces last week:

*"4 unclassified machine identities matching known AI framework patterns (LangChain, n8n) are active but have no documented owner. Created by 4 different employees, 2 of whom have left in the last 90 days."*

The rule is `possible_ai_agent_unclassified`. It runs against the Okta AI Agents API endpoint. It triages NHIs that look like agents (based on naming patterns from the major frameworks) but have not been formally classified yet, so an admin can confirm or dismiss.

The classification matters because once an identity is tagged as an `ai_agent`, the detection engine treats it differently. AI agents are excluded from MFA enforcement rules (they cannot prompt). They are excluded from off-hours anomaly rules (they run 24/7 by design). They are excluded from the human-workforce identity quota count on your plan. But they are subject to their own rules that humans are not.

If you are looking at your Okta or Entra population today and the bot count is hard to distinguish from the human count, you have this problem.

### OAuth blast radius that no human would ever get

Humans typically request the minimum scope they need. AI agents typically request the broadest scope they can. The frameworks usually ship with `read all your data` baked into the example configuration, and engineers tend to ship the example configuration.

We see this constantly:

*"6 admins in Salesforce have granted 11 or more unsanctioned OAuth apps with write-level scopes. None of these apps appear in your sanctioned-app list. Combined blast radius covers 18 different SaaS platforms."*

The rule is `admin_excessive_oauth`. It runs against Google Workspace OAuth grants, scoped by admin status from the IdP. The detection is tiered: 1 to 2 grants is medium, 3 to 5 is high, 6 or more is critical. We emit one finding per admin, not one finding per workspace, so the remediation flow lets you revoke per-app per-admin without leaving the panel.

This rule fires in nearly every workspace we connect, including ours. The OAuth scope review is the single most actionable AI governance task most teams can do in their first week.

### Detection rules tuned for humans, not for agents

This one is subtle. The anomaly detection rules that catch suspicious human behavior are wrong when applied to agents.

A human logging in at 3 AM is suspicious. An agent that runs 24/7 logging in at 3 AM is just doing its job.

A human triggering 50 OAuth scopes in one day is suspicious. An agent legitimately fanning out across your tools to do work might trigger 200.

A human with no recent login activity is dormant. An agent with no recent login activity might be paused or retired or just slow.

If your anomaly tool treats agents as humans, you will either drown in false positives or set the thresholds so high that real human anomalies get missed too. Thalian's behavioral rules use a helper called `isAiAgent()` that returns true via two paths: platform-tag (the identity has `user_type_detail='ai_agent'` AND `identity_type='service_account'`, both required as defense-in-depth) or admin-set override (an admin classified the identity manually). Agents in either bucket are excluded from MFA, SSO enforcement, off-hours, login frequency, location diversity, failed-auth, and app-spike rules. The rules still fire for humans. They are silent on agents.

But agents are not silent on every rule. They are subject to their own. Which leads to the next shape.

### Compound risk: agents exceeding the human workforce

A workspace with 50 humans and 10 AI agents is fine. A workspace with 50 humans and 200 AI agents is a different posture problem. The ratio matters.

A real finding:

*"AI agent count is 32, exceeding 20 percent of the 140-employee human workforce. Growth velocity over the past 60 days: 18 new agents. No offboarding lifecycle policy in place."*

The rule is `drift::ai_agent_growth`. It fires when active AI agents exceed 20 percent of the human workforce in the company. The category is `compound_risk` because it is a steady-state ratio signal, not a 30-day delta. The output is informational, but the underlying question is real: at what point does your AI agent fleet become a population you need to govern as a population, not as individual identities?

This is also where the offboarding gap becomes a critical risk:

*"3 AI agents created by employees who are now terminated. Their grants are still active. Their tokens are still valid."*

The rule is `hr::terminated_ai_tool_access`. It joins your HR system against your IdP and your OAuth grant inventory. When the rule fires, you have evidence of a control gap that auditors will care about.

---

## What a solution actually looks like

There are four pieces. None of them work without the other three.

### Classify NHIs at the platform layer

The first move is to know which identities are AI agents and which are not. The Okta AI Agents API endpoint (`/api/v1/iam/agents`) does this natively for the platform. We sync it in the same `diffAndInsert` call as humans and tag the rows as `identity_type='service_account'` plus `user_type_detail='ai_agent'`. Stored fields include declared OAuth scopes, client ID, grant type, and owner attribution.

For platforms that do not have an AI Agents endpoint, the manual classification path matters. In Thalian, an admin can flip an identity to `ai_agent` from the dropdown in the entity detail panel. The override is gated by the `initiate_remediation` permission. The classification event is written to the audit log as `identity_classified` so an auditor can trace who classified what and when.

Without classification, every downstream rule treats agents as humans and the false-positive rate is bad. With classification, the rules below work correctly.

### Suppress human-centric rules on classified agents

This is what the `isAiAgent()` helper does. It excludes agents from rules that are conceptually wrong to apply to them: MFA enforcement, SSO enforcement, off-hours activity, login frequency anomaly, location diversity anomaly, app access pattern anomaly, failed-auth spike, app count spike, never-logged-in stale-account detection.

These rules still fire for humans. The agents are silently excluded so the rules stay accurate.

### Apply agent-specific rules

A separate ruleset focused on what governs an agent fleet:

- **`possible_ai_agent_unclassified`** — unclassified NHIs matching known framework naming patterns
- **`drift::ai_agent_growth`** — ratio of agents to humans crossing 20 percent
- **`hr::terminated_ai_tool_access`** — agents whose creator was terminated
- **`admin_excessive_oauth`** — admin identities with excessive unsanctioned grants (per the OAuth blast radius story above, particularly acute when the admin is an AI agent)
- **`ai_tool_data_access`** + **`ai_tool_write_access`** + **`ai_tool_widespread`** — detection of unsanctioned AI tools with OAuth grants to corporate data, sourced from a known-AI-tools catalog of 25 AI tools and 12 AI Automation platforms (LangChain, Claude, Cursor, Gumloop, Zapier, n8n, Workato, Devin, and others)

These rules fire only against classified or pattern-detected agents. They give you the agent posture without polluting your human posture.

### Map to a control framework so auditors can verify

This is where NIST CSF 2.0 stands alone. The 2.0 release explicitly extended the PR.AA category from "users" to "users, services, and hardware." That is the canonical control reference for NHI governance, and it is the only major framework that names it cleanly.

The relevant controls are:

- **PR.AA-01** — Identities and credentials for authorized users, services, and hardware are managed
- **PR.AA-03** — Users, services, and hardware are authenticated commensurate with the risk of the transaction
- **PR.AA-05** — Access permissions, entitlements, and authorizations are defined and managed
- **ID.AM-01** — Inventories of hardware managed by the organization
- **ID.AM-05** — Resources are prioritized based on classification, criticality, and value
- **DE.CM-03** — Personnel activity and technology usage are monitored

ISO 42001:2023 is the new AI management system standard and adds a different lens: **A.6.2.6** (AI system operation and monitoring) and **A.6.2.8** (AI system event logs, which Thalian's immutable audit log primitive maps to directly). SOC 2 CC6.1 (logical access controls) and CC6.7 (lifecycle on termination) cover the boring-but-critical pieces.

Map your AI agent posture to those controls in your evidence binder. When the auditor or the customer security questionnaire asks how you govern AI in production, the answer is a control mapping plus a populated detection pass-fail status. Not a paragraph of policy text.

---

## What to do this week

If you are reading this and recognizing the gap, three actions in roughly that order.

**One.** Inventory your AI agents. If you are on Okta, the AI Agents endpoint is a single API call. If you are on Entra ID or Google Workspace, look at your OAuth app inventory and filter to clients with names matching common framework patterns. We maintain a working list in our detection engine that you can use as a reference.

**Two.** Audit the OAuth scope grants of every identity classified as an admin. The single highest-impact finding you can act on this month is the admin OAuth blast radius problem. Most workspaces have admins with broader OAuth grants than they need.

**Three.** Map your current findings to NIST CSF 2.0 PR.AA-01, PR.AA-03, and PR.AA-05. If you do not have findings against any of those controls, you do not have detection on the NHI population. Start there.

Thalian does all three of these out of the box. You can see it on a synthetic mid-market stack at [demo.thalian.ai](https://demo.thalian.ai) without signing up. The Findings page surfaces the same rules described above against seeded data. If you want to run them against your own stack, the [free trial](https://app.thalian.ai/signup) connects to Okta, Entra ID, Google Workspace, and 38 other platforms in under two minutes per integration. The AI agent rules fire on the second sync.

The AI agents are already in your environment. The question is whether your detection layer can see them. Until very recently the honest answer was no.

---

*For the full ISO 42001 + NIST CSF 2.0 control mapping in Thalian, see [Compliance](../../compliance.md). For the data Thalian sends to its AI provider and the retention semantics, see [AI Transparency](../../ai-transparency.md). For the chain-of-custody primitive on the audit log, see [Audit Log Chain of Custody](../../audit-log-chain-of-custody.md).*
