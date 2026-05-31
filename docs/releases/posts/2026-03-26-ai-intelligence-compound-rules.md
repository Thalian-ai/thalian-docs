---
date: 2026-03-26
slug: ai-intelligence-compound-rules
---

# AI brief personalization, cross-platform perspectives, behavioral anomalies, and more

Major intelligence upgrade: personalized AI briefs, cross-platform perspective views, behavioral anomaly detection, 7 compound rules, shadow admin detection, benchmark pricing, and UI refinements.

<!-- more -->

## "Since your last session" AI brief

The dashboard AI brief now opens with what changed since you last logged in. "Since yesterday, 2 new findings surfaced: the most urgent is..." Makes every login feel personalized instead of seeing the same static posture summary.

## Cross-platform perspective view on findings

Expanding a cross-platform finding now shows a side-by-side "What each tool sees" breakdown. Each connected platform's view is shown alongside what Thalian sees by joining them: making it immediately obvious why the finding is invisible in any single tool.

## Live cost counter on integration discovery

When connecting a new integration that reveals license waste findings, the discovery stream animates the cost counting up as findings are scored. Turns abstract waste into a visceral dollar amount.

## Drift velocity projection

Posture sparklines on the Reports page now show a dashed forward projection computed via linear regression on recent data points. Each metric shows where it's heading (e.g., "->92%") alongside the historical delta, giving early warning before thresholds are breached.

## AI chat contextual actions

When the AI mentions a high-severity finding about a specific person or device, it now notes available remediation actions: "You can suspend her in Okta directly from here if needed." The AI only offers when the finding is critical/high, the entity is specific, and the platform supports the action.

## Behavioral anomaly detection

New analysis rules detect unusual login patterns, off-hours access spikes, and sudden app access changes by comparing against per-user behavioral baselines. Fires when current behavior exceeds 2x the historical norm.

## 7 new cross-platform compound rules

New findings that require 3+ connected data sources to detect:

- Terminated employee with dual exfiltration paths (Slack + GitHub)
- Admin on compromised device with active EDR threat
- Dormant account with active OAuth abuse
- Coordinated multi-platform admin actions within 30 minutes
- And 3 more compound scenarios

## Shadow admin detection

New rule identifies users who are standard users in the identity provider but hold admin roles in 3+ connected SaaS apps. These "effective admins" bypass IDP-level MFA and session controls.

## Benchmark SaaS pricing

License waste findings now estimate cost impact using industry-standard per-user pricing for 40+ common SaaS apps. The dashboard cost banner works on day 1 without requiring contract uploads.

## Integration discovery reveal

After connecting your 2nd or later integration, a dedicated interstitial shows the cross-platform findings that only Thalian can see with a pipeline visualization.

## Improvements

- **Teams alerts**: Security finding cards sent to Microsoft Teams now include an "Approve in Thalian" button for approvable actions
- **Findings page**: Value badges consolidated, sort buttons merged, Type filter removed
- **Entity detail panel tightened**: Data Sources hidden when attack chain visible, AI dossier hidden when attack chain narrative exists
- **Remediation cards collapsed by default**: Approval cards now show just the action label + approve/deny buttons
- **Hourly auto-sync**: Connected integrations now sync every hour (previously every 6 hours)

## Fixes

- **Google OAuth app deduplication**: OAuth apps now deduplicated by normalized name
- **Scoring consistency**: All scoring imports cleaned up to use a single source of truth

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-26)
