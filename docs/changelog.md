# Changelog

Notable changes, new features, and fixes for the Thalian platform.

---

## May 2026

### New Features

- **Link emails across platforms**: When the same person uses different email addresses on different platforms (for example, a corporate `tori@thalian.ai` on Google Workspace and a personal `toriferrante@gmail.com` as the Anthropic billing admin), neither platform reports the connection, so they previously appeared as two separate identities with two separate finding sets. The identity detail panel now has a **Linked emails** section where an admin can mark one or more additional emails as belonging to the same human. The cross-platform dedup layer then merges the identities into a single canonical entity, so cross-platform findings like "this person is admin in Google Workspace and admin on Anthropic billing" surface as a single connected risk instead of two unrelated ones. The action requires the `initiate_remediation` permission, is audit-logged as `identity_aliased` for SOC 2 CC8.1 evidence, and rejects any alias already claimed by another identity in the workspace. Platform-reported aliases (Google secondary emails, Okta `secondEmail`) continue to apply automatically; admin overrides take precedence on conflict. The AI chat context also receives a **Linked identities** block so asking about either email returns the merged view.

- **AI Agent and non-human identity (NHI) governance**: Okta AI Agents are now synced as a first-class identity tier — separate from humans and traditional service accounts. AI agents appear in the Identities page with a dedicated tab and count chip, are excluded from MFA, SSO enforcement, off-hours anomaly, and behavioral baseline rules (agents run 24/7 by design), and are excluded from plan-limit identity counts. The identity detail panel shows a purple **AI Agent** badge plus an **Agent info** section with declared OAuth scopes, client ID, grant type, and owner attribution — with a warning when no human owner is recorded. Admins can also classify identities as AI agents via the Account type dropdown in identity detail, which applies the same NHI-specific governance rules. Two new findings surface NHI risks: **Possible AI agent unclassified** fires when a service account matches AI framework naming patterns (LangChain, CrewAI, Gumloop, n8n, and others) but hasn't been formally classified, prompting an admin to confirm or dismiss; and **AI agent count growing** fires when active agents exceed 20% of the human workforce — a compound-risk signal for uncontrolled NHI proliferation with no offboarding lifecycle. Both findings map to SOC 2 CC6.1/CC6.2 and ISO 27001 A.5.15/A.5.18. Okta requires the `okta.iam.agents.read` scope to pull the AI Agents API; see the [Okta integration guide](./integrations/okta.md) for scope details.

- **NIST CSF 2.0 compliance framework**: Thalian's detection rules are now mapped to NIST Cybersecurity Framework 2.0, including six controls surfaced on the Compliance page: **PR.AA-01** (privileged access and non-human identity lifecycle governance — the canonical NHI control), **PR.AA-03** (authentication strength across platforms), **PR.AA-05** (access reviews covering users, services, and hardware), **ID.AM-01** (asset inventory completeness), **ID.AM-05** (resource prioritization by criticality), and **DE.CM-03** (personnel and technology activity monitoring). NIST CSF 2.0 explicitly extended PR.AA to cover "users, services, AND hardware" — Thalian uses this extension for AI agent and NHI governance across the platform. NIST CSF 2.0 appears as a third tab on the Compliance page alongside SOC 2 and ISO 27001.

- **ISO 42001 compliance framework**: Thalian now maps detection rules to ISO/IEC 42001:2023, the first international AI management system standard. Seven Annex A controls surface on the Compliance page: **A.4.2** (AI resource documentation), **A.6.2.2** (AI system requirements and specification — the sanctioning gate for AI tools entering the org), **A.6.2.6** (AI system operation and monitoring), **A.6.2.8** (AI system event logs — backed by the immutable audit log), **A.7.3** (data acquired by AI systems via OAuth grants), **A.9.2** (responsible use processes including offboarding lifecycle), and **A.10.3** (third-party AI suppliers). The framework surfaces as a fourth tab on the Compliance page and gives security teams a defensible answer when customers or regulators ask how AI is governed in production. See the [Compliance page documentation](./compliance.md) for the full control coverage table.

- **NHI-only access review scopes**: Access review campaigns now support two new scope filters — **Non-human identities only** (scopes the campaign to all service accounts, bots, and AI agents) and **AI agents only** (narrowest scope, fastest review for agent-heavy workspaces). These scopes fulfill SOC 2 CC6.3 and NIST CSF 2.0 PR.AA-05 review obligations for non-human principals, which previously fell outside the scope of typical human-identity-focused access certifications.

### Improvements

- **GitHub personal access token connection documented**: The [GitHub integration guide](./integrations/github.md) now describes how to connect with a classic personal access token as an alternative to OAuth, including the exact token scopes the sync uses (`read:org` for members and collaborators, `repo` for private repositories and security signals, `admin:org_hook` for org webhooks, `read:audit_log` for Enterprise audit log, and `admin:org` for remediation writes), plus a fine-grained token note for read-only sync. The product has supported this connection method since launch; only OAuth was documented.

- **Slack bot and app integrations surfaced as non-human identities**: Thalian now inventories every bot and app installed in your Slack workspace instead of skipping them. Each appears as a service account on the Identities page, so you can see which automations have access to your channels. A new finding, *"Slack has N active bot integrations, none reviewed,"* flags workspaces with five or more active bot integrations that have never been inventoried. Bots whose names match known AI agent frameworks (Cursor, Gumloop, CrewAI, n8n, and others) are surfaced separately under AI governance for classification. Bot identities never count toward your plan identity limit. Connect or re-sync Slack to populate the inventory. See the [Slack integration guide](./integrations/slack.md).

- **Trend findings are drillable**: Workspace-level trend and aggregate findings now attach the underlying entities so you can act from the finding panel instead of hunting through the Identities, Devices, or Applications pages. Rules updated: *Unapproved application count is growing*, *EDR sensor coverage is declining*, *Unmanaged (BYOD) device count is growing*, *MDM enrollment covers less than 60% of active IDP users*, *Admin MFA coverage is declining over time*, *Service account count growing faster than headcount*, *Applications bypassing SSO is growing over time*, *Legacy authentication protocols detected in Entra sign-in logs*, *Cloud infrastructure admin authenticating via legacy protocols*, *External users represent more than 15% of total identities*, *Potential password spray from one IP*, *Multiple overlapping apps with significant combined spend*. The shadow-IT growth finding flags newly discovered apps as the delta drivers behind the growth rate. Older non-drillable findings auto-resolve atomically via the Superseded tier of the auto-resolve loop on the next analysis run.

- **Compliance Trend chart plots all four frameworks**: The Compliance Trend chart on the Reports page now plots SOC 2, ISO 27001, NIST CSF 2.0, and ISO 42001 scores side-by-side over time, so you can spot which framework is moving and when. Each framework writes its own score column to `drift_snapshots` on every analysis run. Historical gaps appear as breaks in the line until enough snapshots accumulate after the new frameworks land.

- **Exposure trend card uses a consistent 7-day window**: On the dashboard, the **Exposure trend** card's risk score change now compares the current score against the score from seven days ago, matching the new and resolved finding counts shown directly beneath it. Previously the change spanned your full snapshot history while the counts covered only the last seven days, so the two could disagree. The headline score, the sparkline, and the change indicator now all reflect your most recent analysis history.

- **Remediation actions for PingOne (suspend/unsuspend users)**: PingOne findings now expose **Suspend user** and **Unsuspend user** actions inline, calling `PATCH /v1/environments/{envId}/users/{userId}` under the Worker application credentials configured during connect. The Worker role must include write access to the users resource. Other lifecycle actions (`force_password_change`, `revoke_sessions`, `force_mfa_enroll`, `remove_admin_role`) remain admin-console only. See the [PingOne integration guide](./integrations/ping-identity.md) for scope details.

- **Remediation actions for Datadog (disable/enable users)**: Datadog findings now expose **Suspend user** and **Unsuspend user** actions, calling `PATCH /api/v2/users/{userId}` to flip the user's `disabled` attribute. The Application key must have the `user_access_manage` permission. MFA enforcement and session revocation are not exposed by Datadog's API; suspend/unsuspend will terminate active sessions as a side effect. See the [Datadog integration guide](./integrations/datadog.md).

- **Remediation actions for GitLab (remove from group)**: GitLab findings now expose a **Remove from group** action that calls `DELETE /api/v4/groups/{id}/members/{user_id}` under the Group Access Token configured during connect. The user's GitLab account stays active; only the group membership is revoked. Full account suspension still requires Admin API access, which is not available to GATs. See the [GitLab integration guide](./integrations/gitlab.md).

- **Session revocation for Outlook and SharePoint findings**: Outlook and SharePoint findings now support a **Revoke sessions** action that delegates to Microsoft Graph to invalidate the user's refresh tokens. The Outlook and SharePoint integrations intentionally do not surface account-wide lifecycle actions (suspend, force password change, MFA enroll, admin role removal) because those would disable the entire Microsoft 365 identity; perform those from the [Entra ID integration](./integrations/microsoft-entra-id.md) instead.

- **LiteLLM personal-email admin detection**: The `litellm_personal_email_admin` rule now fires when a LiteLLM `proxy_admin` is registered on a personal email domain (such as `@gmail.com` or `@outlook.com`), matching the existing Anthropic and OpenAI versions of the same detection. The LiteLLM sync handler now pulls the user roster via `/user/list` so each identity carries its role, since the existing `/user/daily/activity` endpoint does not expose role data. Older LiteLLM gateways (before v1.50) and master keys without admin scope still sync identities from activity records but skip the role enrichment without failing. Severity is high for `proxy_admin` and medium for other roles on personal email domains. The rule maps to SOC 2 CC6.1 and CC6.2, ISO 27001 A.5.16 and A.5.18, NIST CSF 2.0 PR.AA-01, and ISO 42001 A.6.2.2.

- **MCP server — action tools (v1.1.0)**: The MCP server at `mcp.thalian.ai` adds six new tools. Query tools (all keys): `list_rules` returns all active detection rules with severity and category; `check_app_policy` returns an application's policy status. Action tools (write-scope keys only): `snooze_finding`, `dismiss_finding`, `remediate_finding`, and `set_app_policy`. All action tools are recorded in the audit log. See [MCP Server](./mcp-server.md) for setup and the write-scope key guide.

- **AI chat grouped finding context**: The AI assistant can now identify and act on all affected users within grouped findings — suspended members in Google Workspace groups, users without MFA, stale admins, and other multi-identity findings. Previously, the chat could not surface specific emails from grouped findings and could not initiate targeted remediation for them. Now when you ask the assistant to remediate a grouped finding, it names each affected user and can act on them individually.

- **AI chat `remove_admin_role` action**: You can now ask the AI assistant to remove admin or privileged roles from a specific user directly from chat. The action is confirmation-gated (same as suspend, revoke sessions, and other high-impact actions) and is supported across all connected identity providers: Okta, Google Workspace, Entra ID, JumpCloud, and OneLogin. The user's account stays active — only elevated roles are removed.

- **Compliance control mapping expansion**: The SOC 2, ISO 27001, NIST CSF 2.0, and ISO 42001 control mappings now cover nearly twice as many detection rules as before. Notable additions: access review evidence rules (`access_review::overdue`, `access_review::rubber_stamped`) mapped to CC6.8 so completed access reviews directly contribute to that control's pass/fail; all 15 `ai_governance::*` rules mapped to ISO 42001 and NIST CSF 2.0 for the first time; terminated employee rules expanded across CC6.7 and A.6.5 to include compound multi-platform offboarding failures; cloud IAM stale access and privilege rules (AWS, GCP, Azure) mapped across CC6.5, CC9.1, and A.8.2; and audit trail gap rules added to CC8.1 and A.8.15. All mappings were validated against the live rule set to confirm no broken references.

- **Audit log chain of custody**: Every audit log row now carries `prev_hash`, `row_hash`, and `chain_version` columns forming a per-workspace SHA-256 hash chain. Each new row is cryptographically linked to its predecessor by `sequence_number` order, so any insertion of a forged historical row, any reordering, and any silent deletion of a row from the middle of the log is detectable by walking the chain and verifying link integrity. Workspace creation auto-creates a `audit_chain.genesis` row that seals the pre-cutover row history with a Merkle-style hash root. The chain enhances the evidence quality of SOC 2 CC8.1, ISO 27001 A.8.15, and ISO 42001 A.6.2.8 control mappings. See [Audit log chain of custody](./audit-log-chain-of-custody.md) for the design, threat model, and verification procedure.

- **Audit log chain integrity panel**: The Compliance page audit log tab now shows a live chain integrity panel. On first open it calls the `verify_audit_chain` database function and displays a status badge (**Verified**, **Broken**, or checking), six key chain stats (rows verified, pre-cutover sealed rows, link breaks, hash breaks, chain tip hash, seal root hash), and the specific broken sequence numbers if any integrity failures are found. A **Verify now** button lets you re-run the check on demand. The panel is visible to all roles with audit log access; remediation for any detected breaks happens through your GRC workflow.

- **Audit log JSONL chain export** (Enterprise): A new **Download Chain Export** button on the Compliance page audit log tab downloads the complete audit log as a JSONL file with a chain-metadata header (`thalian-audit-export/1` schema). The header contains the genesis row hash, chain tip hash, pre-cutover seal root, cutover timestamp, and the SHA-256 of the public Python verifier so an auditor can confirm they have the right tool version. Rows appear oldest-first with `row_hash` and `prev_hash` included so the chain can be reconstructed locally. Designed for offline verification with `verify-audit-export.py` in [thalian-public](https://github.com/thalian-ai/thalian). Every download is recorded in the audit log. Requires the Enterprise plan.

- **Workspace AI memory retention**: The `workspace_ai_memory` table (persistent facts the AI observes about your organization across chat sessions) is now subject to your plan's data retention period. Entries older than your retention window are deleted by the daily retention cron, matching the existing handling for chat history (`ai_messages`). Workspace deletion still removes all entries via cascade, and all writes and deletions remain audit-logged as `ai_memory_saved` and `ai_memory_deleted` actions. See [AI Transparency](./ai-transparency.md) for the data class details.

### Fixes

- **License overrides no longer auto-classify apps as Approved**: Uploading a contract for an app (which creates a license override row used for cost, renewal date, and tier metadata) was pinning that app into the **Approved** tab on the **Applications** page regardless of policy state. Clicking **Revert** had no visible effect because Revert only clears the policy row, not the license override. License data is contract and cost tracking, not a policy approval, so an app with a license override now falls through to **Unsanctioned** unless it is explicitly sanctioned, SSO-provisioned, or stamped sanctioned by sync data. Contract data is preserved and continues to drive the **Source**, **Monthly Cost**, **Tier**, and **Renewal** columns.

- **Compliance PDF export now includes all controls**: The PDF download on the Compliance page was capturing only the score summary (compliance percentage, passing count, failing count) and leaving the controls table blank. The export now produces a complete report including the full controls table with status badges and open finding counts, regardless of any search or filter currently applied in the UI.

- **AI provider integrations now validate admin keys at save time**: Anthropic, OpenAI, and LiteLLM connect attempts were accepting any non-empty key string and only surfacing the error on the first sync, typically as a 401 from the provider's organization usage endpoint. The connect modal now probes the same endpoint the sync handler uses and rejects the credential immediately with a specific error. The most common cause is pasting a regular API key (such as `sk-ant-api03-...`) instead of an Admin API Key (`sk-ant-admin01-...`); the new error message tells you which type you need and where to create it. The Cloudflare credential probe timeout was also tightened from 30s to 10s across every integration validator so a slow upstream cannot consume the full request budget, and the shared SSRF blocklist was expanded to cover Alibaba Cloud, Oracle Cloud, IPv6 link-local, and several AWS / GCP metadata aliases.

- **Per-app findings now surface on the affected identity's card**: OAuth app findings (`oauth_scope_creep`, `ai_tool_data_access`, `ai_tool_write_access`, `broad_oauth_write_access`) and never-used entitlement findings store affected users in the `affected_entities` payload rather than via a direct identity FK column. The identity page filter was skipping these findings entirely because it required the FK to be set before consulting the payload. The filter now reads user emails out of `affected_entities` directly, so asking "what findings affect alice@company.com?" returns app-attributed findings she appears in. Two rules (`oauth_scope_creep` and `access::entitlement_never_used`) now also populate the FK column when exactly one identity resolves from the user list, so single-user app findings attribute to their identity without changing the finding key or type. The Applications page now indexes grouped findings by the UUID of each member app so per-app finding lists are complete for grouped-finding types.

---

## April 2026

### New Features

- **Attack Surface Map**: The Causality Insights panel now shows an interactive SVG graph connecting your identity entry points, attack vectors, and at-risk platforms in a single view. Root cause and systemic pattern cards are clickable and pre-fill the AI assistant with a targeted question. Replaces the previous card-based layout.

- **Attack chain redesign**: The attack chain card is rebuilt as a structured diagram — entry point avatar → animated attack vector pill → fanned platform list. Each platform row shows an intensity-colored tick proportional to entitlement exposure. Hover any count to see "X entitlements at risk."

- **Login geolocation**: Finding detail panels now show city, region, country, and ISP for login-based context (unmanaged access panel and behavioral findings). Processed in-house via MaxMind GeoLite2 on Cloudflare R2 — no customer IP is sent to third parties.

- **Grouped findings**: Eight high-volume rules now emit a single grouped finding with all affected users inside, instead of one finding per identity. Severity scales with scope: 3+ users without a device escalates to critical; MFA coverage gap crossing 50% escalates to critical. Affected rules: active user with no MDM device, admin without MFA, stale admin, MFA coverage gap, password-only auth, cross-platform privilege drift, cross-platform offboarding gap, cross-platform MFA gap.

- **Finding consolidation**: Shadow IT sensitive-scope and broad OAuth write-access findings now consolidate into one finding per workspace (5+ apps escalates to critical). GitHub org owner not-in-IDP and GCP owner/member not-in-IDP findings similarly grouped.

- **Per-admin OAuth revoke**: Admin excessive OAuth findings are now one finding per admin, each with per-app inline revoke buttons. Revocation runs through your IDP by app name — no stored OAuth client ID required.

- **Custom entity names in titles**: 11 per-entity rules now include the specific person's name in the finding title for faster scanning at a glance.

- **Slack App Directory**: Thalian is listed in the Slack App Directory. Install Slack directly from `app.thalian.ai/api/slack-install` without leaving Slack.

- **Security Posture Timeline**: new History page (accessible via **Reports → Timeline** tab or the dashboard "Monitoring since" badge) shows your posture score over time, MFA coverage trend, compliance rate, and a narrative event log of significant changes: grade shifts, MFA drops, new integrations, and remediation milestones.

- **Integration Coverage Widget**: dashboard now shows a 6-category coverage bar (Identity, Endpoint, HR, Security, Cloud, Comms) with per-category status dots and a CTA for the highest-priority gap.

- **MCP server**: Query your Thalian workspace from Claude Code using the Model Context Protocol. Generate an API key in **Settings → API Keys**, configure the local MCP server, and ask questions like "What's our risk score?" or "What does alice@company.com have access to?" directly in the Claude Code chat. Six tools available: `get_risk_score`, `list_findings`, `lookup_identity`, `get_integrations`, `get_posture_summary`, and `trigger_sync`. See [MCP Server](./mcp-server.md) for setup instructions.

- **API Keys**: New **Settings → API Keys** tab for workspace admins. Create workspace-scoped read-only API keys (`thal_` prefix, SHA-256 hashed in storage) for use with the MCP server and future API integrations. Keys show a display prefix and last-used timestamp; the plaintext key is shown once on creation. Up to 10 active keys per workspace.

- **Remediation playbooks**: Multi-step automated response sequences on the Policies page. Build a playbook (e.g., "Offboard terminated employee") with ordered steps across platforms: suspend IDP account, revoke OAuth tokens, wipe device, create ITSM ticket. Each step can be set to auto-execute or require approval. Run a playbook against any finding or entity from the detail panel. Available on Pro and Enterprise.

- **PDF evidence export for access reviews**: Access review campaigns now include a PDF export button that generates a formatted evidence pack for each campaign: reviewer decisions, timestamps, and entity details. Useful for audit evidence and compliance documentation.

- **Compliance Trend tab**: The trend chart tracks **SOC 2** and **ISO 27001** compliance scores over time, computed from live open findings mapped to each framework's controls. Scores range from 0–100 (passing controls / total controls). Historical data is backfilled automatically from existing finding timestamps.

- **In-app service status banner**: A dismissible banner now appears when an active incident or service degradation is detected. Powered by the status page: no configuration required.

- **Claude Opus 4.7 on Enterprise**: Enterprise workspaces now include **Claude Opus 4.7** as the model powering the AI contextual analysis engine, replacing Opus 4.6. No action required. See [AI Transparency](./ai-transparency.md) for the full model matrix.

### Integrations

- **Zoom**: Connect your Zoom organization to detect users and admins not in your corporate IDP, SSO enforcement gaps, offboarded employees with active Zoom accounts, and stale unused seats. Read-only OAuth integration with `user:read:admin` and `account:read:admin` scopes. Syncs users, roles, and account security settings. 5 detection rules.

- **Box**: Connect Box to detect IDP gaps, offboarded employees retaining file access, and external sharing activity. OAuth integration with admin-level scopes. Syncs enterprise users and admin event logs (external sharing events). Cross-references with IDP data to surface departed employees who still have access to corporate files. 4 detection rules.

- **Cross-platform brute-force detection**: New `brute_force_login_detected` rule detects credential stuffing attacks by correlating failed login events across platforms. Fires when 3+ distinct IPs attempt 10+ failed logins against the same account on an identity provider. Escalates to a cross-platform finding when the same email also shows 5+ failures on SaaS apps (Salesforce, Box, Zoom, Slack): a coordinated attack that no single tool can see. Failed login sync added to Salesforce (LoginHistory API), Box (FAILED_LOGIN admin events), and Zoom (sign-in activity reports).

- **341 detection rules**: The analysis engine now runs 341 rules (up from 173 in mid-March), covering identity security, access hygiene, device posture, behavioral anomalies, shadow IT, license waste, compound cross-platform risks, and drift signals. Every rule fires with real data from existing API integrations.

- **Cross-platform compound rules**: 14 new rules that require data from 3+ connected platforms to fire. These are findings that no single tool can surface: terminated employee with active device not wiped (HR + IDP + MDM), EDR threat on a cloud admin's device (EDR + MDM + Cloud IAM), admin in IDP + cloud + CRM simultaneously (IDP + Cloud + Salesforce), and more.

- **AWS IAM deep analysis**: AWS sync now pulls the IAM Credential Report (access key rotation, usage, multi-key detection), root account MFA status via GetAccountSummary, root account activity via CloudTrail LookupEvents, and IAM role trust policy analysis (detects wildcard trust and cross-account principals). 11 AWS rules total.

- **GCP service account key monitoring**: GCP sync now fetches user-managed key metadata for each service account, enabling detection of keys not rotated in 90+ days and service accounts not using Workload Identity Federation.

- **Salesforce session and export detection**: Salesforce sync now checks admin profile permissions (ModifyAllData, ViewAllData), session IP restriction configuration, and SetupAuditTrail for bulk data export events. Enables 9 Salesforce rules total.

- **Entra ID Identity Protection and PIM**: Entra sync now fetches risky users from Identity Protection, PIM permanent role assignments, admin authentication methods (weak MFA detection), and guest invitation policies. All 6 Microsoft Phase 2 rules fire with existing scopes. No reconnection required for customers who already granted the scopes.

- **Okta security configuration analysis**: Okta sync fetches ThreatInsight status, MFA enrollment policies, password policies, API token hygiene, session settings, and network zones. 14 Okta-specific config rules plus AI context.

- **AI context for all platform metadata**: The AI assistant now surfaces AWS access key stats, IAM role trust policy data, GCP SA key status, Salesforce profile permissions and export activity, Okta security config, Entra credential expiry and legacy auth count, GitHub deploy keys and branch protection, Zoom SSO enforcement, and Box external sharing events.

- **Platform category registry**: New integrations are now classified via a single `PLATFORM_CATEGORY` map in the rule engine. Adding a new IDP, HR, MDM, or EDR platform is a one-line change that automatically propagates to all 300+ rules. No more maintaining 130+ duplicate platform lists across individual rules.

- **Access review bulk decisions + overdue reminders**: Access review campaigns now support bulk approve and bulk revoke across multiple items at once. Campaigns past their due date automatically send overdue reminder emails to reviewers with a direct link to the campaign.

- **Trial extension + compliance preview**: Free-tier users can now self-serve a trial extension from the billing page. The Compliance page is now visible to free users in a preview mode with a plan gate, so prospects can see the value before upgrading.

- **GitLab**: Connect a GitLab group to detect developer access risks across your source code platform. Syncs group members, projects, deploy keys, and group access tokens using a Group Access Token (no OAuth required). Works with both GitLab.com and self-hosted instances. 8 detection rules: admins and maintainers without MFA, Maintainer/Owner accounts not in your IDP (critical), developers not in your IDP, external members with elevated access, deploy keys with write permissions, non-expiring group tokens, offboarded employees with active membership, and stale members with no recent contribution activity. Audit event sync available on GitLab Premium and Ultimate.

- **GitHub secret scanning**: GitHub sync now pulls open secret scanning alerts and push protection bypass events. Two new detection rules: unresolved secret scanning alerts (critical, workspace-level finding) and developers who bypassed push protection 2+ times (high, per-user finding).

- **CrowdStrike Spotlight vulnerability detection**: CrowdStrike sync now fetches open critical and high CVEs from Spotlight after each device sync. Two new cross-platform rules: unpatched critical CVEs on managed devices (one finding per device, severity mirrors worst CVE) and high-severity vulnerabilities on admin-access devices (critical, highest blast-radius signal).

- **Entra ID PIM activation monitoring**: Entra sync now pulls Privileged Identity Management activation history (last 30 days). Two new detection rules: PIM role activated without a justification field populated, and PIM activation outside business hours (weekend or before 6am/after 10pm UTC).

- **Okta high-risk signin detection**: New rule fires when Okta System Log records a completed sign-in flagged as high risk by Okta's risk engine. Requires Okta System Log sync (active since launch): no new connection steps.

- **Identity profile enrichment**: Identities now carry 10 new profile fields including department, title, manager, employee type, and hire date. Three new detection rules fire on enriched data: provisioning anomaly (identity in IDP with no corresponding HR record), department with no offboarding workflow, and role-title mismatch (admin title in HR but no IDP admin privileges, or vice versa).

- **Box Shield threat detection**: Box sync now fetches Shield anomaly alerts and file download events. Two new rules: unresolved Box Shield threat alerts (high, workspace-level) and departing employee mass download detection (critical, fires when an offboarded user downloaded 50+ files in 7 days).

- **Salesforce permission set escalation**: Salesforce sync now fetches permission set assignments granting admin-equivalent access and currently-active session-based permission set activations. Two new rules: non-admin user with admin-equivalent permission set (high) and active session permission escalation (high).

- **SentinelOne Ranger unmanaged device detection**: SentinelOne sync now pulls Ranger gateway data to discover network-connected devices without a SentinelOne agent installed. Single workspace-level finding with count and sample hostnames.

- **Confluence and Jira audit log rules**: Confluence sync now fetches space permission change events from the audit API (last 30 days). Jira sync fetches permission scheme and role change events. Two new rules surface permission changes that may indicate privilege escalation: `confluence::space_permission_change` and `jira::permission_scheme_changed`.

- **390 detection rules**: Rule count reaches 390 (up from 350) with 40 new platform-depth rules across 7 areas, all firing on existing synced data with no new connection steps required:
  - **MDM platform-specific rules**: Intune (non-compliant admin with SaaS access), Jamf (FileVault disabled, stale check-in), Fleet (policy cascade failures, unpatched critical CVEs, CVE on sensitive-app owner, admin device vulnerabilities, offline device with active entitlements), Workspace ONE (non-enrollment, compliance violations, stale check-in, enrollment gap vs. Entra), Iru (Gatekeeper disabled, FileVault off, MDM profile inactive, stale check-in, unassigned device)
  - **Workday-specific rules**: Contingent worker with permanent-employee app access, position eliminated with active IDP account, rehire with prior entitlements restored, department change with no access review triggered
  - **Microsoft 365 communication security**: Teams: external guest in sensitive channel, open guest federation policy, offboarded user still a guest, admin using personal account. SharePoint: anonymous link policy open, offboarded user accessing files. Outlook: mailbox delegation to external address, shared mailbox with no MFA, auto-reply with internal information visible externally. Compound: external comms spike pre-offboarding, file share + messaging exfiltration in offboarding window
  - **MDM × IDP compound rules**: Active IDP user with no device in any MDM, terminated identity with device still enrolled, admin with cloud admin rights on non-compliant device, IDP-deactivated identity with recent MDM activity, MDM enrollment coverage below 60% vs. IDP user count, BYOD device owner with admin entitlements
  - **JumpCloud-specific rules**: Admin without MFA, users without MFA, accounts with non-expiring passwords
  - **OneLogin-specific rules**: Users not enrolled in any MFA factor, admin accounts with no security policy assigned, offboarded primary IDP user still active in OneLogin

### Improvements

- **Bulk app policy actions**: select multiple apps on the **Applications** page and approve, flag, or block them all at once. A floating action bar appears when you select one or more apps. Use the **Select all** shortcut to apply a policy across an entire tab in one click. Available to admins and security roles.

- **Workspace risk score rebuilt**: The 0–100 workspace risk score now uses a linear, CVSS-aligned formula (`severity weight × finding count ÷ 150`, capped at 100) instead of a sigmoid curve that saturated at high risk levels. This means every finding change produces a meaningful, proportional score movement: each critical finding is worth ~6.7 pts, high ~3.3 pts, medium ~1.3 pts, low ~0.7 pts. The previous formula caused all **Recommended Actions** point deltas to display as ±0 for workspaces with moderate-to-high risk: that bug is fixed. Additionally, cost-only findings (license waste) and metadata-noise rules that don't represent active security risk are now excluded from the workspace score, so the score reflects actual security exposure. Six rules reclassified from low to medium severity to better reflect their real impact: Okta factor enrollment optional, Okta network zone bypass, Okta ThreatInsight disabled, Entra Security Defaults disabled, GitHub default branch protection missing, and SharePoint external sharing activity.

- **Webhook destination picker**: Connecting a webhook or SIEM integration now opens a destination picker before the configuration form. Two sections: **Workflow Automation** (Workato, Zapier, n8n, Gumloop, Make) and **SIEM & Observability** (Datadog, Splunk, Elastic, Panther, Sumo Logic, Custom). Each destination includes a setup hint explaining how to get your endpoint URL from that platform.

- **Webhook event improvements**: The webhook/SIEM integration now fires three distinct events. `finding_detected` fires only for genuinely new findings discovered in that analysis run: not all open findings on every run. `finding_resolved` fires when a finding is automatically resolved because its condition no longer holds. `analysis_completed` now includes a `new_findings_count` field so automation workflows can skip processing when nothing changed. The `finding_detected` payload now includes `finding_id`, `action_type`, and `source_integrations` for richer downstream automation.

- **Drift trend sparkline**: finding detail panel for drift-signal findings (MFA coverage declining, SSO coverage declining, etc.) now shows a mini sparkline with a 3-step forward projection.

- **Ghost finding cards**: Findings page surfaces projected findings for missing integration categories so you can see what Thalian would detect if you connected a missing tool.

- **Post-remediation nudge**: after completing a remediation action, Thalian suggests the next integration category most relevant to that action type.

- **Weekly digest**: now includes integration coverage gap nudge and cumulative workspace stats (identities monitored, findings resolved, integrations connected).

- **Sync Events tab**: Integrations page now has a Sync Events tab showing the full entity diff log (creates, updates, deletes) across all connected platforms.

- **Milestone lifecycle emails**: workspaces now receive a progress email at 30, 60, and 90 days summarizing integrations connected, findings resolved, MFA coverage, and posture grade.

- **Remediation action buttons**: Finding detail panels now show actionable buttons (suspend, revoke, create ticket, etc.) across all finding types. Previously, some finding types only showed a text recommendation with no buttons. Safe actions auto-execute; risky actions queue for approval.

- **Application sanctions**: The Applications page now has sanction, block, and revert buttons directly on application rows for faster policy enforcement.

- **Finding deduplication**: Findings that have been actioned (dismissed, snoozed, resolved) are no longer re-created on the next analysis run. Previously, dismissing a finding and re-running analysis would surface it again.

- **AI remediation prompt tuning**: The AI remediation planner now generates more direct, actionable plans and filters out service account targets that can't be meaningfully remediated.

- **GCP IAM role names in findings**: GCP IAM findings now display specific role names (Owner, Editor, Viewer) instead of generic "IAM access" descriptions.

- **Analysis cooldown reduced**: Analysis cooldown reduced from 5 minutes to 1 minute for faster iteration.

- **Sidebar findings count**: Sidebar findings badge now updates immediately after running analysis without requiring a page refresh.

- **Analysis error reporting**: Finding insert errors are now reported to Sentry and the audit log for better observability.

- **Landing page overhaul**: Added Open Graph image for social sharing, "See Live Demo" CTA in hero and navigation, rendered FAQ section, pricing visibility, mobile navigation fix (primary CTA now visible at all breakpoints), and Docs link in top nav.

- **Demo experience**: New guided 5-step walkthrough for first-time demo visitors, persistent demo banner with conversion CTA, and login simplified to email/password only (with a request access form for new visitors).

- **Signup hardening**: Company name is now required on the signup form. Personal email domains (Gmail, Yahoo, etc.) are blocked: work email required.

- **SSO error messaging**: SSO login errors now include admin self-service guidance with a link to Settings when no SSO is configured for the domain.

- **Iru naming**: Kandji references across the app and docs updated to "Iru (formerly Kandji)" for consistency with the platform rebrand.

- **MDM AI context**: The AI assistant now understands device management data from all 9 MDM platforms (Intune, Jamf, Kandji, Hexnode, Mosyle, Fleet, SimpleMDM, Workspace ONE, Scalefusion). Ask about device compliance, encryption status, stale check-ins, or problem devices and get platform-aware answers.

- **Paused integration coverage**: Paused integrations now count toward coverage gap calculations on the dashboard. Previously, pausing an integration would trigger a "missing coverage" alert even though the integration was still protecting that domain.

- **11 CI drift checks**: Automated checks now verify that platform registries, permission maps, severity scores, analysis rule categories, and compliance framework mappings stay in sync between frontend and backend. Any drift is caught before merge.

- **FAIR-aligned entity risk scoring**: Entity risk scores now use a FAIR-aligned model that weighs privilege level, platform exposure, finding severity, and behavioral signals. Scores are more meaningful and comparable across identity types.

- **Integration error classification**: Integration errors are now classified by type (authentication expired, configuration invalid, rate limited, API error). A sidebar badge shows the count of integrations needing attention, and a dismissible app-wide banner appears when critical auth errors require reconnection.

- **Blast radius orbit visualization**: The blast radius view on entity details now renders as an interactive orbit diagram instead of a flat list, making it easier to see the scope of access and downstream impact at a glance.

- **Behavioral baseline accuracy**: Directory-sourced login events (Google Workspace, Microsoft 365) are now excluded from behavioral anomaly baselines when a dedicated IDP (Okta, Entra, JumpCloud, OneLogin) is connected. This prevents noisy directory sync events from inflating login frequency baselines and triggering false anomalies.

- **Finding category consolidation**: The "Configuration" finding category has been folded into "Access Risk" and "Identity Security" for a cleaner, more actionable grouping. Rules previously classified as configuration findings now appear under the category that best matches their remediation path.

- **Free plan identity usage bar**: Free plan workspaces now see a live usage bar showing monitored identities against the plan limit, with a one-click upgrade path.

- **Posture timeline: stale accounts and null-gap rendering**: The posture over time chart now includes stale account coverage as a tracked metric and handles null data gaps (periods with no analysis run) with correct interpolation instead of chart artifacts.

- **Pro data retention**: Pro plan data retention extended from 90 days to 1 year for all workspace data and audit logs.

### Fixes

- **MFA enforcement**: enabling MFA on a workspace no longer immediately blocks active sessions. Users currently signed in can finish their session normally; MFA enrollment is required on their next login.

- **Automation policy matching**: Automation policies were silently matching against the wrong finding scope — the engine was falling back to category-wide match for policies with an empty rule ID list while the UI counted violations strictly by rule ID. Fixed via an explicit `match_mode` column. Check the **Policies** page — violation counts may change.
- **Fleet encryption and compliance data**: Encryption status and compliance were showing as "Unknown" for Fleet-managed devices. Fixed by fetching per-host detail endpoints after the initial list.
- **Device OS column**: Devices table now shows the OS version string instead of the raw internal platform identifier (e.g., `darwin`).
- **AI chat response truncation**: Max token limit raised to 4096 with pagination for long responses.
- **Finding panel "Authorized by" on OAuth findings**: Now aggregates unique users across all apps in a consolidated finding, scoped to a selected app when one is clicked.
- **Entity show more / show less**: Finding detail panels with 10+ entities now show a toggle instead of an unbounded list.

- **SSO finding accuracy for Google Workspace-only environments**: When Google Workspace is your only identity provider (no Okta, Entra ID, JumpCloud, OneLogin, or PingOne), SSO-related findings now correctly identify ungoverned OAuth grants rather than reporting apps as "bypassing SSO." Apps that authenticate via Google sign-in are not bypassing SSO: they're outside centralized OAuth governance. Finding titles, descriptions, and remediation guidance have been updated accordingly, directing admins to **Google Admin → Security → API controls** to review and restrict OAuth app access.

- **SSO finding accuracy: Google-only ratio and entity payload corrections**: Three follow-up fixes to Google Workspace-only SSO findings: `sso::high_direct_auth_ratio` now requires OAuth grants to represent a majority of an identity's app portfolio before firing (previously could fire at any count and incorrectly state "over half"); the `sso::offboarded_with_direct_auth_apps` and `behavioral_off_hours_direct_auth` findings now correctly report `oauth_count` in their entity payloads rather than `direct_count: 0`, ensuring the AI assistant and entity detail panels reflect the actual risk signal.

- **Findings suppression**: Actioned and dismissed findings are now properly excluded from re-creation during analysis, preventing duplicate findings after remediation.
- **Remediation denied actions**: Actions that were denied by an approver no longer resurface in the remediation queue.
- **Reports sparklines**: Posture trend sparklines are now correctly constrained and show accurate drift projections.
- **Integration removal cleanup**: Removing an integration now properly anonymizes associated findings (PII redacted) while preserving them for audit trail.
- **Light mode readability**: Teal text elements are now darker for better readability on white backgrounds.
- **Integration removal**: Fixed integration removal failing with an internal error.
- **GCP IAM project discovery**: Fixed GCP IAM sync not discovering projects by adding v1 API fallback for environments without org-level permissions.
- **GCP IAM identity sync**: Fixed GCP IAM identities not syncing due to database constraint violation on service account identity type.
- **Finding batch insert**: Fixed analysis silently dropping findings when duplicate finding_keys existed in a batch. One bad row no longer kills the entire batch.
- **Native client false positives**: Fixed native client apps (iOS Mail, Android) being incorrectly flagged as shadow IT OAuth risks.
- **Ghost identities**: Fixed ghost "[removed]" identities appearing on the Identities page after integration removal.
- **Compound finding links**: Fixed "Related findings" links pointing to stale IDs on compound findings.
- **Orphaned entities**: Fixed orphaned entities persisting after integration removal by changing FK constraints to CASCADE.
- **Stale entity analysis**: Fixed entities from disconnected or paused integrations loading into analysis and generating stale findings. Entity data now scoped to connected integrations only.
- **Device enrollment remediation**: Fixed "Send enrollment invite" button failing silently due to a naming mismatch between analysis rules and the remediation executor.
- **Finding categories**: Fixed findings with `access_risk` and `configuration` categories rendering without category labels or icons.
- **Platform action buttons**: Fixed action buttons appearing on platforms that don't support them (read-only platforms like Outlook, SharePoint, Confluence, cloud IAM, HR tools). Corrected Zoom, Box, and Salesforce to show both suspend and revoke session actions.
- **GCP IAM remediation buttons**: Fixed remediation buttons showing app-centric actions instead of identity actions on GCP IAM findings.
- **Billable identity count**: Billable identity count now only includes IDP and directory users, excluding SaaS-only accounts that inflated the count beyond what customers expected.
- **MTTR calculations**: Mean time to resolution now excludes auto-resolved findings (findings that resolved themselves on the next sync), giving a more accurate picture of actual remediation effort.
- **Device page simplification**: Removed managed/unmanaged device tab split in favor of a single unified view. Fixed compliance status string mismatch between frontend and backend.
- **Behavioral baseline scope**: Fixed behavioral baseline suppression applying too broadly, which could mask legitimate anomalies on identities that share a platform with a suppressed directory source.
- **Programmatic login severity**: Downgraded `suspicious_programmatic_login` finding from high to medium severity to reduce alert fatigue for service account activity that is unusual but not inherently risky.
- **Google OAuth WebViews warning**: Fixed a console warning triggered by Google's updated OAuth policies when signing in from embedded web views (Claude desktop app, Electron-based tools).
- **Billing flow correctness**: Fixed plan gate logic incorrectly blocking some Pro workspace actions, and hardened trial expiry detection to handle clock skew edge cases.
- **Approval request emails**: Fixed approval request emails sending twice in some cases, and corrected missing template variable substitution.

### Security

- **HMAC key isolation**: Webhook and AI chat action HMAC keys are now domain-separated via HKDF. A key compromise in one context cannot be used in another.
- **Single-use action tokens**: HMAC confirmation tokens for AI-initiated remediation actions are consumed on first use.
- **AI chat tool gating**: `trigger_sync` requires `manage_integrations` role; `run_analysis` requires `analyze` role.

- **npm supply chain hardening**: In response to the March 30 Axios npm supply chain attack (CVE pending, attributed to North Korean threat actor UNC1069), we audited all dependencies and confirmed Thalian is not affected: axios is not in our dependency tree. We've additionally hardened our build pipeline: npm audit now blocks deployments on high-severity findings, postinstall scripts from transitive dependencies are disabled by default, all dependency versions are pinned exactly, and lockfile integrity validation has been added to CI.

- **AI chat prompt injection hardening**: Added topic-scoping guardrails to the AI assistant to prevent prompt injection attempts from manipulating responses. The AI is now constrained to IT security and identity governance topics, with explicit rejection of attempts to override system instructions or extract internal configuration.

- **RBAC audit: role gating hardened**: A full audit of all role-based access control surfaces closed gaps where certain UI actions and API endpoints were accessible to roles below the required permission level. Affected surfaces: integration management, workspace settings, billing actions, and agentic remediation approvals. No data was exposed: the gaps were UI/action availability only.

- **Workspace ONE vendor reference**: Updated all in-app and documentation references from "VMware Workspace ONE" to "Omnissa Workspace ONE" following VMware's acquisition and rebranding by Omnissa.

---

## March 31, 2026

### New Features

- **Entra ID: Conditional Access policy detection**: Thalian now fetches and analyzes your Entra ID Conditional Access policies automatically after each sync (requires re-authorizing the Microsoft connection to grant `Policy.Read.All`). Three new detection rules fire when CA policies are available: MFA policy in report-only mode (logs violations but never blocks), disabled MFA policy (potential regression if previously enforced), and admin accounts explicitly excluded from all MFA-requiring CA policies. The AI assistant also gains a Conditional Access context block and can answer questions about which policies are enforced vs report-only, whether MFA is actually blocking sign-ins, and which admins aren't covered. Existing Entra connections without the new scope continue working: CA rules stay silent until re-auth.

- **Okta System Log correlation**: Three new Okta-specific detection rules now use System Log data to surface credential and authentication risks that event-by-event inspection misses: failed MFA spike (5+ failed MFA challenges per user in the sync window: potential credential stuffing), MFA factor disabled (user or admin disabled an MFA factor: elevated severity for admin accounts), and user-reported compromise (user clicked "This wasn't me" in Okta: highest-confidence indicator of active account takeover). Okta System Log has been synced since launch; these rules make that data actionable without any new connection steps.

### Improvements

- **GCP IAM privilege analysis**: GCP IAM now detects 4 new privilege and configuration risks beyond IDP gap detection: owner role sprawl per project, service accounts with admin-level roles, users with admin access across 3+ projects, and systemic over-provisioning (>50% of users at Editor or higher). These rules fire even when Google Workspace is the IDP: previously, GCP findings only appeared when users existed outside the corporate identity provider.

---

## March 30, 2026

### Integrations

- **Workday HR**: Connect Workday to cross-reference employee lifecycle data against your identity providers and SaaS access. Thalian syncs active and terminated workers from Workday and detects terminated employees who still have active IDP accounts, SaaS entitlements, or managed devices. Joins with Okta, Entra ID, Google Workspace, JumpCloud, OneLogin, Intune, Jamf, and all connected SaaS platforms. Read-only: uses Workday's REST API with basic auth credentials. Adds to the existing HR intelligence layer alongside Rippling and BambooHR.

### New Features

- **Compliance: evidence export (Enterprise)**: The Compliance page now includes PDF and Excel evidence pack export for Enterprise workspaces. Pro users see a locked "Export evidence" button with an upgrade prompt. The export includes control status, mapped rules, open findings, and a timestamp: formatted for handing directly to an auditor.

- **Compliance: audit log tab**: A dedicated Audit Log tab is now available on the Compliance page, showing a filterable, searchable feed of all user and system actions. Pro workspaces see 30-day history with CSV export; Enterprise sees 1-year retention. Free users see the plan gate.

- **Integration library redesign**: The integration browser has a new card layout and filter bar. Cards now show category, connection status, and sync stats. Filter by category (Identity, Device, HR, Cloud, etc.) to find what you need faster.

- **Clickable stat pills on integrations**: Stat pills on connected integration cards (e.g. "42 identities", "7 findings") now navigate directly to the relevant filtered view: Identities, Applications, Devices, or Findings: scoped to that integration.

### Improvements

- **Plan tier copy**: The billing and upgrade pages now accurately reflect what each plan includes. Pro lists SOC 2 & ISO 27001 control mapping, access review campaigns, and executive reports under a Compliance category. Enterprise lists IP allowlisting, MFA enforcement policy, configurable session timeout, and SSO/SAML enforcement under Security & Governance, plus 1-year audit log retention and SIEM export under Compliance & Audit.

- **Compliance tab gating**: All three tabs on the Compliance page (Controls, Access Reviews, Playbooks) now correctly require Pro or Enterprise. Previously, free users could navigate directly to the Access Reviews and Audit Log tabs via URL and bypass the plan gate.

### Fixes

- **Display labels throughout**: Raw internal identifiers (e.g. `notify_user`, `google_workspace`, `non_compliant`, `in_progress`) no longer appear in the UI. Action types, remediation statuses, compliance statuses, and audit event types are now shown as readable labels everywhere: in finding history, the identity timeline, notification badges, the devices table, and the remediation queue.
- **Audit log retention**: The audit log now correctly shows 90-day history for Pro workspaces (previously displayed "30-day history" due to a hardcoded mismatch).
- **Pricing constants**: Billing page pricing is now sourced from a single constant, so it stays consistent if pricing changes.
- **Compliance deep links**: "View findings →" links inside expanded compliance controls now navigate correctly to the Findings page filtered to that rule.
- **AI chat MFA/login accuracy**: The AI assistant no longer reports MFA or login status for platforms that don't expose that data (e.g. GitHub, Slack), preventing misleading "no MFA" statements for accounts where MFA is managed by an IDP.

---

## March 29, 2026

### New Features

- **SSO coverage per identity**: The Identities page now shows how many of each user's apps are SSO-managed vs direct-auth. Hover over the app count to see the breakdown (e.g., "3 SSO-managed, 2 direct auth"). When an IDP is connected and a user has direct-auth apps, the count highlights in purple to flag the gap.
- **Login source attribution**: The Last Login column now prefers IDP login data over SaaS timestamps. Hover over any login timestamp to see which platform it came from (e.g., "Last seen in Okta"). If all apps are SSO-covered, the tooltip says so; if some bypass SSO, it tells you how many.
- **7 new SSO coverage findings**: Five identity-level rules detect SSO gaps: admin with direct-auth apps (critical), executive with direct-auth apps (critical), offboarded user with direct-auth apps that SSO deprovisioning can't reach (critical), majority of apps bypassing SSO (high), and new hire provisioned outside the IDP workflow (medium). Two behavioral anomaly rules detect spikes in direct-auth app grants and off-hours activity on non-SSO apps.
- **Identity type filtering**: The Identities page now has an All / Users / Service & Shared toggle. Service accounts (shared inboxes, bots, automation accounts) are identified using a 40+ prefix list and separated from human users in the view. The total stat card shows an inline service account count alongside the human identity count.
- **Sync engine: stale-before-AI**: AI Brief and AI Chat now trigger a background sync for any integration that hasn't synced recently before generating content, ensuring responses reflect current posture. Concurrent requests share the same in-flight sync to avoid redundant API calls.

### Improvements

- **MFA accuracy**: MFA findings (admin without MFA, MFA coverage gap, executive without MFA) now only fire when an identity provider is connected and the IDP confirms MFA is disabled. Previously, SaaS-only workspaces could see false-positive "No MFA" findings because SaaS platforms don't report MFA status.
- **MFA column clarity**: When no IDP is connected, the MFA column shows a dash with a hover tooltip explaining that an identity provider is needed to verify MFA status, instead of misleading "No MFA" badges.
- **AI chat SSO context**: The AI assistant now includes per-admin SSO coverage (e.g., "SSO: 3/5 apps") and a workspace-level SSO coverage stat in its analysis context.
- **Disconnect cleanup**: Removing an integration now cancels all pending approval actions tied to that integration's entities in a single two-pass cleanup (fetch all entities → find all related pending actions → cancel). Previously, pending actions could linger after disconnection.
- **Audit timestamp precision**: Remediation actions and settings audit entries now show absolute timestamps (e.g., "Mar 28, 2026, 2:34 PM PST") instead of relative time ("2 days ago"). Improves readability for audit and compliance review.
- **AI chat deep links**: When AI responses reference a connected integration, they now include direct links to the relevant admin console (Google Workspace Admin, Microsoft Entra ID portal, GitHub org settings, etc.) so you can navigate directly from chat to take action.
- **Status page: immediate incident creation**: The status health check now creates an incident on the first degraded check rather than waiting for a recovery cycle. Reduces lag between a real outage and an incident appearing on status.thalian.ai.

### Fixes

- **Signup workspace naming**: Personal email signups (Gmail, Outlook, iCloud, etc.) now get "Personal Workspace" as their workspace name instead of the email domain (e.g., "gmail.com Workspace").

---

### Integrations

- **Salesforce**: Connect Salesforce to detect CRM access gaps between your Salesforce org and your corporate identity provider. Thalian syncs all active and inactive Salesforce users and cross-references against Okta, Entra ID, Google Workspace, JumpCloud, and OneLogin. Fires four new findings: Salesforce admin not in IDP (critical), Salesforce user not in IDP (high), stale Salesforce user whose IDP account is suspended or deprovisioned but CRM access remains active (high), and connected app authorized by an unknown user outside the IDP (medium). Salesforce is read-only: Thalian uses the `api refresh_token offline_access id` OAuth scopes to sync users and OAuth grants. Auth uses a Salesforce Connected App with an OAuth flow. No write permissions are requested or required.

### Improvements

- **Slack alerts: Dismiss and Snooze from Slack**: Security alerts sent to Slack now include Dismiss and Snooze 7d buttons directly on every alert card. Clicking Dismiss marks the finding as dismissed; Snooze silences it for 7 days. Both actions update the original Slack message in place with a confirmation line (who took the action, when). All actions are signature-verified and written to the immutable audit log: same security model as the existing Approve Remediation button.

- **Okta: upgraded to OAuth 2.0 client credentials**: Okta sync and all 7 Okta remediation functions now authenticate using OAuth client credentials instead of a static SSWS API token. If you have Okta connected, you'll need to reconnect it with your Okta Client ID and Client Secret. The new flow is more secure, token rotation is handled automatically, and it removes the need to manage long-lived API tokens.

---

## March 28, 2026

### Integrations

- **AWS IAM**: Connect AWS Identity and Access Management to detect IAM users that exist outside your corporate IDP lifecycle controls. Thalian syncs all IAM users via `GetAccountAuthorizationDetails`, detects admin-level accounts (AdministratorAccess / PowerUserAccess) with no matching IDP identity, IAM users without MFA enrolled, and stale IAM users whose IDP account has been suspended or deprovisioned. AWS IAM does not auto-sync with Okta, Entra, or any other directory: credentials persist after offboarding until explicitly deleted. Auth uses Access Key ID + Secret Access Key, no OAuth required.

- **GCP IAM**: Connect Google Cloud Platform to detect IAM access gaps between your GCP projects and your corporate identity provider. Thalian maps every human member across all your GCP projects and cross-references against Okta, Entra ID, Google Workspace, JumpCloud, and OneLogin. Fires four new findings: GCP project owner not in IDP (critical), GCP member not in IDP (high), public IAM binding via `allUsers` or `allAuthenticatedUsers` (critical), and stale IAM binding for a suspended or deprovisioned user (high). GCP IAM does not auto-sync with corporate directories: access must be removed from IAM policies separately when employees leave. Thalian now surfaces exactly which accounts are falling through the gap.

### New Features

- **Access review campaigns**: IT and security teams can now run structured user access certification campaigns directly in Thalian. Create a campaign, scope it by department or application, and work through a paginated list of entitlements: approving access, revoking it, or granting a time-limited exception. Revoke decisions automatically open an ITSM ticket in Jira, ServiceNow, Freshservice, or Zendesk. Completed campaigns export to CSV as audit evidence for SOC 2 (CC6.3), ISO 27001, and HIPAA reviews. A sidebar badge shows pending items. The AI is aware of campaign status and can answer questions like "how many items are still pending in my Q1 review?"

- **Claude-driven agentic remediation planner**: After every sync, Claude Sonnet now reviews all new findings and decides which ones need action, what action to take, and why. Queued actions in the Remediation page now include a "Claude's reasoning" block explaining the recommendation: e.g., "This account hasn't logged into the IDP in 47 days but retains admin roles in Salesforce and GitHub. Suspending reduces blast radius while the offboarding gap is investigated." The planner sequences actions correctly (root cause first), groups multi-finding actions on the same identity, and skips findings it judges low-priority or already handled.

---

## March 26, 2026

### New Features

- **"Since your last session" AI brief**: The dashboard AI brief now opens with what changed since you last logged in. "Since yesterday, 2 new findings surfaced: the most urgent is..." Makes every login feel personalized instead of seeing the same static posture summary.

- **Cross-platform perspective view on findings**: Expanding a cross-platform finding now shows a side-by-side "What each tool sees" breakdown. Each connected platform's view is shown alongside what Thalian sees by joining them: making it immediately obvious why the finding is invisible in any single tool.

- **Live cost counter on integration discovery**: When connecting a new integration that reveals license waste findings, the discovery stream animates the cost counting up as findings are scored. Turns abstract waste into a visceral dollar amount.

- **Drift velocity projection**: Posture sparklines on the Reports page now show a dashed forward projection computed via linear regression on recent data points. Each metric shows where it's heading (e.g., "→92%") alongside the historical delta, giving early warning before thresholds are breached.

- **AI chat contextual actions**: When the AI mentions a high-severity finding about a specific person or device, it now notes available remediation actions: "You can suspend her in Okta directly from here if needed." The AI only offers when the finding is critical/high, the entity is specific, and the platform supports the action.

- **Behavioral anomaly detection**: New analysis rules detect unusual login patterns, off-hours access spikes, and sudden app access changes by comparing against per-user behavioral baselines. Fires when current behavior exceeds 2× the historical norm.

- **7 new cross-platform compound rules**: New findings that require 3+ connected data sources to detect, including: terminated employee with dual exfiltration paths (Slack + GitHub), admin on compromised device with active EDR threat, dormant account with active OAuth abuse, and coordinated multi-platform admin actions within 30 minutes.

- **Integration discovery reveal**: After connecting your 2nd or later integration, a dedicated interstitial shows the cross-platform findings that only Thalian can see. Displays the pipeline visualization (e.g., "Okta + Google Workspace → detected by Thalian") and lists the specific findings that are invisible in any single tool.

- **Shadow admin detection**: New rule identifies users who are standard users in the identity provider but hold admin roles in 3+ connected SaaS apps. These "effective admins" bypass IDP-level MFA and session controls: Thalian is the only tool that can see the privilege gap.

- **Benchmark SaaS pricing**: License waste findings now estimate cost impact using industry-standard per-user pricing for 40+ common SaaS apps. The dashboard cost banner works on day 1 without requiring contract uploads.

- **Cross-platform badge on findings**: Findings generated by joining data from multiple platforms now show a teal badge with the contributing platform names (e.g., "Okta + Slack"). Applied retroactively to all existing cross-platform rules, not just new compound rules.

### Improvements

- **Teams alerts: "Approve in Thalian" button**: Security finding cards sent to Microsoft Teams now include a green "Approve in Thalian" button for approvable actions (suspend user, revoke access, block app, contain host, etc.). Clicking it opens the Remediation page filtered to that exact finding so you can approve with one click.

- **Findings page**: Value badges consolidated: cost estimate ($X/yr) shown when available, risk impact (−N pts) as fallback: never both. Sort buttons merged into single "Sort: value". Type filter removed (rarely used). Net result: cleaner rows, fewer buttons.

- **Entity detail panel tightened**: Data Sources chips hidden when attack chain is visible (redundant). Finding count removed from header meta line (already in section header). AI dossier hidden when attack chain narrative exists (attack chain is more structured).

- **Remediation cards collapsed by default**: Approval cards now show just the action label + approve/deny buttons without expanding. Click to see full plan details. Reduces visual noise when reviewing a queue of pending actions.

- **Hourly auto-sync**: Connected integrations now sync every hour (previously every 6 hours). Findings, drift snapshots, and remediation queues update more frequently.

### Fixes

- **Google OAuth app deduplication**: OAuth apps discovered through Google Workspace sync are now deduplicated by normalized name, preventing duplicate entries for apps with slight naming variations.

- **Scoring consistency**: All scoring imports cleaned up to use a single source of truth. Agentic plan endpoint and cron schedules updated to match.

---

## March 25, 2026

### New Features

- **Three new cross-platform drift signal rules**: The intelligence engine now detects three additional trend-level findings that require data from multiple platform types simultaneously:
  - **SSO coverage declining**: Fires when the proportion of apps authenticating via SSO is trending downward over time. Includes a breach date projection and lists the top non-SSO apps. Requires an IDP and app discovery data.
  - **Termination-to-access-removal lag growing**: Measures the average time between an employee's HR termination date and their IDP account suspension across recent offboardings. Fires when the lag exceeds 2 days across 3+ offboarding events, and flags whether the lag is getting worse over time. Also surfaces any terminated employees who still have an enrolled device. Requires HR (Rippling or BambooHR) and an IDP; MDM data is incorporated when available.
  - **Ghost identity growth**: Detects users being added to SaaS tools (Slack, GitHub, Jira, etc.) without a corresponding account in any connected IDP. Fires when 3 or more such users appear in the last 60 days, or when the rate is accelerating. Requires an IDP and at least 2 connected SaaS platforms.

- **Automatic remediation now fires after every sync**: Workspaces with auto-remediation enabled will now have eligible findings automatically actioned immediately after each scheduled sync (every 6 hours and at 8am daily). Safe actions (create ticket, notify user, sanction app) execute automatically; higher-risk actions (suspend user, revoke OAuth token, block app) are queued for admin approval and trigger an email notification. All agentic actions appear in the Remediation page under the "Automatic" filter tab.

- **AI Risk Summary on identity detail**: Opening any identity with open findings now shows a "Risk Summary" block powered by Claude: a concise narrative covering their risk score, MFA status, app access breadth, device compliance, and blast radius. The summary loads in place in the identity detail panel, using the same visual style as the dashboard AI brief.

- **OAuth scope risk labels on unsanctioned apps**: The Applications page Unsanctioned tab now shows inline scope risk chips (e.g. "Writes email", "Reads calendar files") sourced from server-side OAuth scope interpretation. These labels surface the actual permission risk of each unsanctioned OAuth app, not just the finding category.

---

## March 23, 2026

### Improvements

- **In-app support form**: "Contact support" in the user menu now opens a proper support form (category, subject, message) instead of the Plain chat widget. Tickets are submitted directly to the support inbox and include your workspace name and plan for context. You'll receive a confirmation email when your ticket is submitted.

---

## March 21, 2026

### Improvements

- **Cross-platform identity context in findings**: Findings that affect a user active across multiple identity providers now say so explicitly. For example, "Tori Ferrante is active across Google Workspace and Okta but is logging in from a device not enrolled in your MDM." The finding detail panel also shows teal platform badges on the affected entity chip so the cross-platform scope is immediately visible.

- **One finding per person, not per platform record**: The intelligence engine now understands that the same email address across multiple integrations represents the same human being. Findings like "Active users without managed devices" now generate a single finding per person with full cross-platform context, rather than one finding per integration record. The underlying identity data model now has three distinct tiers: authoritative IDP only (for MFA rules), raw all-platform records (for cross-platform mismatch detection), and a new canonical per-person layer (for per-human findings).

### Fixes

- **OAuth scope false positives eliminated**: The platform was incorrectly flagging Figma, Slack, and other apps as having email inbox access because the basic OIDC `email` scope (which only provides a user's email address for authentication) was being matched as a substring of "mail." A centralized scope interpretation engine now tests each OAuth scope individually with precise patterns. `email` no longer implies inbox access: only explicit grants like `gmail.readonly` or `https://mail.google.com/` do. This fix applies across findings, AI analysis, briefs, and dossiers.

- **Shadow IT findings no longer duplicated**: Apps flagged as both "Unvetted app with sensitive data access" and "Unvetted OAuth applications detected" were appearing twice in the findings list. Apps already covered by the more specific sensitive-scope finding are now excluded from the general unvetted apps finding.

- **Impact analysis scores now match the Security Posture score**: The "Current" and "After action" scores shown in the "Simulate a fix" panel on finding detail were using an old raw severity-weighted formula. They now use the same sigmoid-normalized 0–100 Security Posture formula as the dashboard, so the numbers are consistent across the product. The panel badge now reads "+X posture pts" / "−X posture pts" to make the direction unambiguous.

---

## March 20, 2026

### New Features

- **SAML 2.0 SSO for enterprise workspaces**: Enterprise plan workspaces can now configure SAML 2.0 single sign-on directly from Settings → Security → SSO/SAML. Admins provide their IdP metadata URL and email domain; Thalian registers the SAML connection and displays the ACS URL and Entity ID to share with the IdP administrator. Supports both SP-initiated login (via the "Sign in with SSO" button on the login page) and IdP-initiated login (clicking the Thalian tile in Okta, Azure AD, or any SAML-compatible IdP). SSO users are automatically provisioned into the correct workspace on first sign-in. The SAML ACS endpoint is served from `auth.thalian.ai`.

### Improvements

- **Security Posture score history and sparkline**: The Security Posture stat on the dashboard now shows a live sparkline trend line (up to 30 data points) and a delta indicator (e.g., +5 or −3) comparing your current score to the previous analysis run. A point-in-time snapshot is recorded automatically after every analysis. The AI assistant also now has access to the full posture trend history, so it can tell you whether your security posture has been improving or declining over time.

### New Features

- **Status page issue report notifications**: When a user submits a report via the "Report an Issue" form on the status page, two emails are now triggered: an internal notification to support@thalian.ai with the reporter's email, affected service, and description, and a confirmation email to the reporter acknowledging receipt.

- **In-app feedback form**: A "Give feedback" option is now available in the bottom-left user menu. Submit bug reports, feature requests, or general feedback without leaving the app.

### Improvements

- **AI chat context: remediation history and prior assessments**: The AI assistant now has full visibility into remediation actions taken in the last 30 days, including pending approvals, completed actions, and Claude's prior reasoning from AI-generated assessments. If actions are awaiting approval, the assistant will proactively surface them.
- **AI chat context: richer entity data**: Findings in the AI prompt are now explicitly linked to the named user or app they affect, rather than relying on IDs. Device detail now includes OS platform and version. Application listings include category. Admin account lines include department where available.
- **Security Posture score**: The "Risk Score" stat on the dashboard has been replaced with a unified **Security Posture** score (0–100, higher is better). Uses sigmoid normalization so a single critical finding doesn't catastrophically tank the number, and noise (many low findings) doesn't equal a real critical threat. Displayed with a letter grade (A–F) and color-coded green/amber/red. The same formula now drives both the dashboard and the AI assistant: they will always agree on your score.
- **AI chat context: finding descriptions**: The assistant now receives the full description of each open finding (why the finding exists and what makes it risky), enabling it to explain findings in plain language rather than just naming them.
- **AI chat context: stale access analysis**: The assistant can now identify entitlements where the app hasn't been used in 90+ days, surfacing which apps have the most stale seats and which users have access they never use.
- **AI chat: what-if simulation tool**: The assistant can now answer "what would happen if I suspended this user?" or "how many findings would close if I revoked this access?" by running a live simulation against real workspace data before any action is taken.

### Fixes

- **Slack alert deduplication fixed: alerts no longer fire on every sync**: Two compounding bugs were causing alert rules to re-fire on every sync run. First, the dedup key used internal Supabase UUIDs as entity identifiers; since email-discovered apps were being deleted and re-inserted on each sync, they received new UUIDs each time, making every finding appear "new" and bypassing the dedup log. Second, the Google Workspace email app sync was deleting all email-discovered apps on every run, not just stale ones, causing apps to disappear and reappear with new IDs every other sync. Both are now fixed: dedup keys use stable external identifiers (OAuth client IDs, external system IDs, names) and the email app sync only removes apps that are genuinely absent from the current scan.
- **Integration sync failures now surface on the status page**: A sync error in any connected integration (Okta, Google Workspace, etc.) now correctly flips the integration's status to `error`, which the status page health check reads. Previously, failed syncs wrote a `sync_error` message but left the status field as `connected`, so the status page remained green. In addition, integrations that haven't synced in over 10 hours are now flagged as degraded even if no explicit error was recorded: catching silent failures where the sync process dies without writing an error.

---

## March 19, 2026

### New Features

- **AI-reasoned remediation**: For entities with multiple overlapping findings, ask Claude to analyze all open risks in context and propose a sequenced action plan with reasoning. Available from the Remediation page under "AI Recommendations" and from Identities via a one-click deep-link
- **AI vs. Auto badge differentiation**: Remediation action log now distinguishes Claude-reasoned plans ("AI" badge, violet) from deterministic rule-based automation ("Auto" badge, teal)
- **AI reasoning in approval cards**: When reviewing Claude-proposed actions, the approval card now shows Claude's assessment: what the situation is, what risk it poses, and why the recommended actions are the right response
- **Public status page**: Real-time platform health at [status.thalian.ai](https://status.thalian.ai) with incident history and email subscription for downtime alerts
- **Layer 3 behavioral baselines**: Per-entity anomaly detection that builds individual baselines for users, devices, and apps and flags deviations from established patterns
- **Findings notification filtering**: Alert notifications now fire only on new findings; recurring open findings no longer generate repeat alerts

### Improvements

- **Status page**: Per-service 90-day uptime bars now visible under each service row without expanding; incident cards show affected services chips; uptime bar tooltips include status label; social sharing previews (OG/Twitter cards) now supported
- **Findings page**: Each tab now includes a short descriptor explaining what the tab tracks, reducing onboarding friction
- **Causality Insights**: Redesigned compact card layout for the insights tab; remediation suggestions are now entity-specific based on the finding subject
- **Performance**: Initial app bundle reduced by 69%; database queries parallelized for faster page load
- **Auto-sync reliability**: Integrations now sync in parallel, preventing worker timeouts on workspaces with many connected platforms

### Security

- Auth guards and workspace scoping added to 7 previously unguarded API endpoints

### Changes

- Status page moved from an in-app route to a standalone Cloudflare Worker, accessible without authentication

---

## March 2026

### New Features

- **Impact Analysis page**: Model remediation scenarios before executing them with the scenario builder. Simulate "what if?" actions and see projected risk score changes, findings that would close, and downstream effects
- **KPI Dashboard and Goals Tracker**: Set measurable security goals (OKRs) with target values and deadlines. Track velocity, get AI-recommended actions, and monitor progress across 14 metrics
- **Policy auto-generation**: Auto-generated security policy drafts based on your connected integrations and active findings. Review, edit, approve, and export as PDF
- **Causality Insights**: Cross-platform finding correlation that surfaces connections between related findings sharing the same affected entity
- **Agentic remediation**: Automated remediation with three tiers (`auto_execute`, `auto_queue`, `never`) for Pro and Enterprise workspaces
- **Drift snapshots**: Point-in-time posture snapshots captured after each analysis run, powering trend charts across the Reports page

### Integrations

- Added **Cisco Meraki**: Network devices, clients, and VPN status
- Added **Confluence**: Spaces, external sharing, and content exposure
- Added **SharePoint**: Sites, external sharing, and document permissions
- Added **Freshservice**: Tickets, agents, and assets
- Added **Zendesk**: Tickets, users, and organizations
- 24 platforms now supported across 7 categories

### Improvements

- **What-if simulation** on finding detail panels: Preview impact before taking remediation action
- **Audit log export**: Download the full audit log as JSON for SIEM integration
- **Workspace export**: Export all workspace data as JSON
- **IP allowlisting**: Restrict API access to approved IP addresses or CIDR ranges
- **Session timeout options**: Configurable session durations (1h, 4h, 8h, 24h, 72h)

---

## February 2026

### New Features

- **AI Brief**: AI-generated natural language summary of your workspace's security posture, available on the Dashboard and Reports pages
- **Remediation approval workflow**: Agent-initiated actions on high/critical findings require Security Analyst approval
- **Sync events log**: View all data changes (created, updated, deleted) detected during integration syncs

### Integrations

- Added **SentinelOne**: Agents, threats, and device health
- Added **Hexnode**: Cross-platform devices and policies
- Added **Jira Service Management**: Service requests, agents, and queues

### Improvements

- **MFA enforcement**: Workspace admins can now require TOTP-based MFA for all members
- **MTTR tracking**: Mean Time to Remediate metrics by severity, with trend charts

---

## January 2026

### Launch

- **Thalian platform launch** with support for 15 initial integrations
- **79 analysis rules** across 7 categories: Identity Security, Shadow IT, Compound Risk, Device Posture, License Waste, Drift Signal, Access Hygiene
- **25 remediation action types** across identity, application, and device categories
- **6 RBAC roles** with enforced permission hierarchy
- **AES-256-GCM encryption** for all integration credentials
- **Immutable audit log** with SHA-256 integrity hashing
- **Free, Pro, and Enterprise plans** with 30-day Pro trial for new workspaces

---

*For information on getting started with the platform, see [Getting Started](./getting-started.md).*
