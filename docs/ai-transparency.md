# AI Transparency

**Thalian, LLC**
**Effective Date:** April 2026
**Last Reviewed:** May 2026

---

## 1. Overview

Thalian uses AI to analyze cross-platform IT data, generate security findings, plan remediation actions, and answer natural-language questions about your environment. This page describes exactly what AI models we use, what data enters and exits the AI pipeline, and the controls in place to keep you in charge.

## 2. Models and Providers

Thalian does not operate its own AI models. We integrate with **Anthropic** via the Claude API.

| Function | Model | Purpose |
|---|---|---|
| **AI chat and analysis** | Claude Sonnet (Free / Pro) · Claude Opus 4.7 (Enterprise) | Natural-language Q&A, executive briefs, entity dossiers, root-cause analysis |
| **Remediation planning** | Claude Sonnet | Generating step-by-step remediation action plans from findings |
| **Contract extraction** | Claude Sonnet | Parsing uploaded license/contract documents into structured fields |

We use Anthropic's commercial API tier. Under Anthropic's API terms, data submitted through the API is **not used to train models**.

We may integrate additional or alternative AI providers in the future and will update this page accordingly.

## 3. What Data Enters AI Prompts

When the AI engine runs, a system prompt is assembled from your workspace data. The following categories are included:

| Data Category | Examples | Why It Is Needed |
|---|---|---|
| **Identities** | Email, display name, role, MFA status, department, last login | To reason about user risk and cross-platform access patterns |
| **Applications** | App name, SSO status, category, sanction status | To assess shadow IT, license waste, and coverage gaps |
| **Devices** | Device name, OS, compliance status, encryption state | To evaluate endpoint posture alongside identity risk |
| **Entitlements** | Who has access to which app, with what role, last used when | To detect over-provisioning, ghost access, and offboarding gaps |
| **Findings** | Open security findings (title, severity, affected entity) | To prioritize remediation and explain risk context |
| **Audit events** | Recent sign-in events, admin actions, forwarding rules | To detect behavioral anomalies and support root-cause analysis |
| **Integration metadata** | Connected platform names, sync status | To scope answers to your actual environment |

### What Is Excluded

- **Integration credentials** (API tokens, OAuth secrets) are encrypted with AES-256-GCM and **never** included in AI prompts
- **Other workspaces' data** is never included — prompts are scoped to a single workspace
- **Raw database identifiers** (internal IDs, Supabase project references) are not exposed to the AI model
- **Payment data** (Stripe customer IDs, billing details) is never sent to AI providers

## 4. What the AI Produces

The AI generates the following types of output, all scoped to your workspace:

- **Security findings** — plain-language sentences describing a risk, its severity, and a recommended action
- **Executive briefs** — posture summaries highlighting what changed since your last session
- **Entity dossiers** — profiles of specific users combining data from all connected platforms
- **Remediation plans** — step-by-step action plans with up to 8 actions per plan
- **Root-cause analysis** — grouping of related findings by shared underlying cause
- **Contract extraction** — structured fields (vendor, cost, renewal date, etc.) from uploaded documents
- **Chat responses** — answers to natural-language questions about your environment

AI outputs are stored within your workspace (in the `ai_messages` table) and are subject to your plan's data retention period.

## 5. Zero-Training Guarantee

Your data is **not** used to train, fine-tune, or improve any AI model. This guarantee has three layers:

1. **Anthropic's API terms** — Data submitted via the Claude API is not used for model training under Anthropic's standard commercial API agreement
2. **Prompt scoping** — Each prompt contains only your workspace data; no data is pooled across customers
3. **No feedback loops** — Thalian does not submit AI outputs, user corrections, or conversation ratings back to Anthropic for training purposes

## 6. Data Retention at the AI Provider

Anthropic's API processes prompts in real time for AI inference. Under Anthropic's commercial API terms, Anthropic may retain API inputs and outputs for up to **30 days** for trust and safety review purposes. This data is **not** used to train, fine-tune, or improve any AI model — Anthropic's API terms explicitly prohibit using API-submitted data for model training.

Thalian uses the standard commercial API tier and has not enrolled in Anthropic's Zero Data Retention (ZDR) program. If you require a ZDR commitment — where Anthropic retains no inputs or outputs for any period — please contact us at privacy@thalian.ai to discuss Enterprise plan options.

AI conversation history is retained **within your Thalian workspace** according to your plan tier:

| Plan | AI Conversation Retention |
|---|---|
| **Free** | 7 days |
| **Pro** | 1 year |
| **Enterprise** | Unlimited |

You can disable chat persistence entirely in **Settings > General** or from the AI chat panel. When disabled, conversations are not saved to your workspace and are discarded at the end of each session.

## 7. Human Oversight Controls

Thalian treats AI as a decision-support tool. You make the final call.

### Remediation Tiers

Not all AI-recommended actions execute automatically. Actions are tiered by risk:

| Tier | Examples | Behavior |
|---|---|---|
| **Auto-execute** | Create ITSM ticket, send notification, sanction app, trigger sync | Runs immediately — low risk, easily reversible |
| **Requires approval** | Suspend user, revoke OAuth token, block app, contain host, force password change | Queued for human review — a workspace admin must explicitly approve before execution |
| **Never automated** | Deactivate user, retire device, remote lock | Recommended only — Thalian will not execute these actions |

### Cryptographic Confirmation

High-impact actions initiated through AI chat require an **HMAC-signed confirmation token** with a 5-minute expiry. The token is presented to the user before execution and creates an immutable audit log entry.

### Audit Trail

Every AI-initiated action — whether auto-executed or approved — is logged in the immutable audit log with a SHA-256 content hash. These entries cannot be modified or deleted.

## 8. Accuracy and Limitations

AI-generated findings and recommendations are based on the data available in your workspace at the time of analysis. Important limitations:

- **Completeness depends on integrations** — Findings can only surface risks from platforms you have connected. Gaps in integration coverage may result in blind spots.
- **AI may produce incorrect or incomplete analysis** — The AI engine is a reasoning tool, not an oracle. Always verify critical findings before acting on them.
- **Behavioral baselines need history** — Anomaly detection rules require at least two weeks of sync data to establish per-user baselines. Early findings may have lower confidence.
- **No real-time monitoring** — Thalian analyzes data at sync intervals (hourly for connected integrations), not in real time. Time-sensitive threats may not surface immediately.
- **Contract extraction is best-effort** — Uploaded documents are parsed by AI; extracted fields should be verified against the original document.

## 9. Your Controls

| Control | How to Use |
|---|---|
| **Disconnect an integration** | Stops data from that platform from entering future AI prompts |
| **Delete AI conversation history** | Available via **Settings** or automatically enforced by your plan's retention period |
| **Require approval for write actions** | Enabled by default for high-impact actions; configurable per workspace |
| **Restrict AI access by role** | Viewer and Auditor roles can see findings but cannot initiate AI-driven remediation |
| **Export your data** | Full workspace export (JSON) available from **Settings** — includes AI conversation history |
| **Delete your account** | Contact support@thalian.ai — all data including AI history deleted within 30 days |

## 10. Regulatory Alignment

This page is provided for transparency. For formal data processing commitments:

- **GDPR:** See the [Data Processing Agreement](./data-processing-agreement.md) for Controller/Processor roles, sub-processor list, and international transfer safeguards
- **CCPA:** See the [Privacy Policy](./privacy-policy.md) for California-specific rights
- **SOC 2 / ISO 27001:** See [Compliance](./compliance.md) for how AI-related findings map to control frameworks

## 11. Changes

We will update this page when we change AI providers, models, or the categories of data included in AI prompts. Material changes will be communicated via email to workspace administrators.

## 12. Contact

For questions about AI data handling, contact privacy@thalian.ai.

---

*For our complete privacy practices, see [Privacy Policy](./privacy-policy.md). For technical security details, see [Information Security Policy](./information-security-policy.md).*
