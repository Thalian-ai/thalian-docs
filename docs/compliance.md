# Compliance

The Compliance page maps Thalian's findings and controls directly to SOC 2 Type II, ISO 27001, NIST CSF 2.0, ISO 42001, NIST AI RMF (GOVERN), and the EU AI Act. Your team can see at a glance which controls are covered, which are at risk, and what evidence Thalian has already collected. All six frameworks render side-by-side as tabs, and the Compliance Trend chart tracks per-framework scores over time.

---

## What is the Compliance page?

The Compliance page (`/compliance`) translates Thalian's raw findings into control-level coverage across six frameworks:

- **SOC 2 Type II** — Trust Services Criteria (Security, Availability, Confidentiality, Processing Integrity, Privacy)
- **ISO 27001** — Annex A controls
- **NIST CSF 2.0** — Cybersecurity Framework 2.0, including non-human identity and AI agent governance
- **ISO 42001** — AI Management System (ISO/IEC 42001:2023) controls for AI inventory, monitoring, data flow, responsible use, and third-party AI suppliers
- **NIST AI RMF (GOVERN)** covers the GOVERN function of the NIST AI Risk Management Framework: AI risk policies, risk-proportionate oversight, AI risk culture, and third-party AI supply chain
- **EU AI Act** maps key obligations for high-risk AI systems: risk management (Art. 9), data governance (Art. 10), transparency (Art. 13), and human oversight (Art. 14)

For each control, Thalian shows:

- **Status** — Passing, Failing, Attention, or Not Monitored
- **Related findings** — open findings that affect this control's compliance posture
- **Evidence** — links to the data Thalian has collected that supports the control (audit logs, identity records, device compliance status, etc.)

---

## Availability

The Compliance page is available on **Pro and Enterprise** plans. It is accessible to users with the **Security Analyst**, **Admin**, **Super Admin**, or **Auditor** role.

---

![The Compliance page showing SOC 2 controls with passing, failing, and attention status indicators](./assets/screenshots/compliance/compliance-soc2.png)

## SOC 2 coverage

Thalian maps its findings across the five Trust Services Criteria:

### CC6 — Logical and Physical Access Controls

| Control | What Thalian monitors |
|---|---|
| CC6.1 — Logical and Physical Access Controls | MFA enforcement across all connected IDPs and SaaS apps |
| CC6.2 — Credential Issuance | User provisioning, admin elevation, service account and shared account detection |
| CC6.3 — Authorization-Based Access | Role-based access, OAuth scope management, privilege drift across platforms |
| CC6.5 — Access Restrictions | Stale accounts, dormant admins, ghost entitlements, departed user access |
| CC6.6 — Threats and Vulnerabilities | Endpoint protection, EDR coverage, infected devices, OS currency, admin devices without management |
| CC6.7 — Access Removal Upon Termination | Terminated user access detection across all connected platforms |
| CC6.8 — Access Review | Periodic access review coverage — stale admins, unused entitlements, dormant OAuth apps, license clawback |

### CC7 — System Operations

| Control | What Thalian monitors |
|---|---|
| CC7.2 — Anomaly Monitoring | Behavioral anomaly detection, unusual login patterns, off-hours activity, mailbox forwarding rules |
| CC7.3 — Anomaly Evaluation | Drift signal analysis — MFA coverage decline, admin sprawl expansion, compliance degradation |

### CC8 — Change Management

| Control | What Thalian monitors |
|---|---|
| CC8.1 — Change Management | Integration sync health, audit trail with SHA-256 tamper detection |

### CC9 — Risk Management

| Control | What Thalian monitors |
|---|---|
| CC9.1 — Risk Mitigation | Cross-platform risk correlation, shadow IT detection, privilege accumulation, departing user data access |

---

## ISO 27001 coverage

![The Compliance page showing ISO 27001 Annex A controls with coverage status and related findings](./assets/screenshots/compliance/compliance-iso27001.png)

Thalian maps findings to 12 Annex A controls across organizational, people, physical, and technological domains:

### Organizational Controls

| Annex A Control | What Thalian monitors |
|---|---|
| A.5.9 — Inventory of Information and Other Associated Assets | Shadow IT discovery, unvetted applications, unmanaged devices, EDR coverage gaps |
| A.5.15 — Access Control | IDP-to-SaaS entitlement coverage and gaps, MFA enforcement, SSO bypass detection |
| A.5.18 — Access Rights | Privilege minimization, admin sprawl, privilege accumulation, ghost entitlements, cross-platform privilege drift |
| A.5.23 — Information Security for Use of Cloud Services | OAuth scope management, scope creep detection, dormant OAuth apps, excessive app grants |

### People Controls

| Annex A Control | What Thalian monitors |
|---|---|
| A.6.1 — Screening | Users not found in identity provider, service account human pattern detection, shared account detection |
| A.6.5 — Responsibilities After Termination | Offboarding gaps, departed user entitlements, cross-platform terminated user access |

### Physical Controls

| Annex A Control | What Thalian monitors |
|---|---|
| A.7.9 — Security of Assets Off-Premises | Unmanaged devices, unencrypted endpoints, end-of-life OS versions, stale MDM check-ins |

### Technological Controls

| Annex A Control | What Thalian monitors |
|---|---|
| A.8.1 — User Endpoint Devices | Admin devices without compliance, EDR coverage, infected devices, CrowdStrike/SentinelOne sensor health |
| A.8.2 — Privileged Access Rights | Admin MFA enforcement, stale admins, admin excessive OAuth, admin elevation bursts |
| A.8.5 — Secure Authentication | MFA coverage gaps, department-level MFA gaps, MFA regression, MFA coverage decline trends |
| A.8.15 — Logging | Integration sync health, audit trail with SHA-256 tamper detection |
| A.8.16 — Monitoring Activities | Behavioral anomaly detection — login frequency, location diversity, off-hours activity, failed authentication, mailbox forwarding rules |

---

## Using compliance status in reviews

The Compliance page is designed to be used alongside your access reviews and audit preparation:

- **Before an audit:** run the Compliance page to identify controls that have open findings — these are the areas an auditor is most likely to probe
- **During an access review:** link your Access Review campaign decisions as evidence against CC6.7 (terminated user access), CC6.8 (access review), and A.5.18 (access rights)
- **For a penetration test prep:** use the At Risk controls to prioritize remediation before external testing begins

---

## NIST CSF 2.0 coverage

Thalian maps findings to NIST Cybersecurity Framework 2.0 — the third framework tab on the Compliance page alongside SOC 2 and ISO 27001. NIST CSF 2.0 extended the Protect.AA (Privileged Access) category to cover "users, services, AND hardware," making it the canonical framework for AI agent and non-human identity (NHI) governance.

Thalian surfaces six NIST CSF 2.0 controls:

### PR.AA — Privileged Access and Authentication

| Control | What Thalian monitors |
|---|---|
| PR.AA-01 — Non-human identity lifecycle | AI agent and service account lifecycle governance — sync from Okta AI Agents, classification enforcement, owner attribution, orphan agent detection, and agent population growth monitoring |
| PR.AA-03 — Authentication strength | MFA enforcement across all connected identity providers and SaaS applications |
| PR.AA-05 — Access reviews | Periodic review of entitlements including "AI agents only" and "NHI only" scopes — NHI-specific certification obligation explicit in NIST CSF 2.0 |

### ID.AM — Asset Management

| Control | What Thalian monitors |
|---|---|
| ID.AM-01 — Asset inventory | Identity and application inventory completeness — unclassified AI agents, unmanaged devices, shadow IT applications |
| ID.AM-05 — Resource criticality | Classification of high-priority identities (admins, executives) and high-risk assets for prioritized remediation |

### DE.CM — Monitoring

| Control | What Thalian monitors |
|---|---|
| DE.CM-03 — Activity monitoring | Behavioral anomaly detection, off-hours activity, login frequency baselines, mailbox forwarding rule detection |

### Using NIST CSF 2.0 for AI agent governance

NIST CSF 2.0 PR.AA-01 is the most direct control for AI agent governance obligations. When Thalian surfaces the **Possible AI agent unclassified** or **AI agent count growing** findings, both map to PR.AA-01 in the Compliance page — so you can use Thalian's finding status as live evidence that you are actively monitoring your NHI population. For PR.AA-05, running an access review campaign scoped to "AI agents only" or "NHIs only" produces a timestamped evidence record directly attachable to audit requests.

---

## ISO 42001 coverage

ISO/IEC 42001:2023 is the first international standard for AI management systems. Thalian maps findings to the Annex A controls that cover the operational surface where shadow AI, ungoverned AI tools, and AI agent sprawl create real exposure. Formal AI policy authoring, impact assessments, and AI development controls live outside the platform.

### AI System Resources

| Control | What Thalian monitors |
|---|---|
| A.4.2 — Resource Documentation | Inventory of AI systems and tools in use across the org, including unclassified AI agents and widespread unsanctioned AI tool adoption |

### AI System Life Cycle

| Control | What Thalian monitors |
|---|---|
| A.6.2.2 — AI System Requirements and Specification | Sanctioning gate for AI tools — unreviewed OAuth grants with corporate data access surface as control gaps |
| A.6.2.6 — AI System Operation and Monitoring | AI agent population velocity, write-scope AI grants, and unclassified machine identities flagged as ongoing operational risk |
| A.6.2.8 — AI System Recording of Event Logs | Immutable audit log (SHA-256 tamper detection) covering AI agent sync, classification, and remediation events |

### Data for AI Systems

| Control | What Thalian monitors |
|---|---|
| A.7.3 — Acquisition of Data | Corporate data flowing to third-party AI tools via OAuth grants — both read access and write-back surfaces |

### Use of AI Systems

| Control | What Thalian monitors |
|---|---|
| A.9.2 — Processes for Responsible Use | Terminated employees with active AI tool grants, widespread unsanctioned AI use, write-scope grants without review |

### Third-Party and Customer Relationships

| Control | What Thalian monitors |
|---|---|
| A.10.3 — Suppliers | Unsanctioned AI vendors in widespread use surface as de-facto suppliers without formal supplier management |

### Using ISO 42001 for AI governance programs

ISO 42001 is the cleanest framework to point to when a customer or regulator asks how you govern AI in production. Run the Compliance page filtered to ISO 42001 before an AI risk assessment — failing controls there name the specific tools, agents, or data flows driving the gap, which becomes the input list for your AI inventory and risk register.

---

## NIST AI RMF (GOVERN) coverage

The NIST AI Risk Management Framework GOVERN function is where AI risk policy and oversight live. Thalian maps its cross-platform AI agent governance findings to four GOVERN outcomes:

| Control | What Thalian monitors |
|---|---|
| **GOVERN 1.1** — AI risk policies in place | Confirmed AI agents with no governance record, no declared purpose, or an expired authorization |
| **GOVERN 2.2** — Risk-proportionate oversight | Agents with no autonomy classification, and high-risk agents on an infrequent review cadence |
| **GOVERN 2.4** — AI risk culture and inter-agent dependencies | One AI agent acting on another agent's identity (an unreviewed trust chain between non-human identities) |
| **GOVERN 5.1** — Third-party AI supply chain | AI providers receiving prompts and data with no sanctioned vendor record, and unsanctioned MCP gateways |

The GOVERN tab pairs with the NIST CSF 2.0 tab: CSF 2.0 covers the operational PR.AA / ID.AM controls, while NIST AI RMF GOVERN covers the policy and oversight layer above them.

---

## EU AI Act coverage

Thalian maps the agent governance surface to the EU AI Act obligations most relevant to deployers and providers of AI systems:

| Control | What Thalian monitors |
|---|---|
| **Art. 9** — Risk management system | Behavioral drift and scope-exceeded findings on agents operating outside their declared mandate |
| **Art. 10** — Data governance | AI providers receiving data with no sanctioned vendor record (no evidence of a data processing agreement) |
| **Art. 13** — Transparency | Agents with no declared purpose or governance record on file |
| **Art. 14** — Human oversight | Agents with no autonomy classification, so the human-oversight model cannot be assessed |

---

## Exporting compliance evidence

Each control in the Compliance page has an **Export Evidence** option that generates a summary of:

- Control description
- Thalian's coverage status
- Supporting data (finding count, affected entities, last sync timestamp)
- Links to related audit log entries

This output can be attached directly to audit request responses or included in a security review package.

---

*For information on running structured access certifications, see [Access Reviews](./access-reviews.md).*
*For information on Thalian's own security posture, see [Information Security Policy](./information-security-policy.md).*
