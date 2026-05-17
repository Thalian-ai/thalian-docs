# Data Processing Agreement

**Thalian, LLC**
**Effective Date:** April 16, 2026
**Last Reviewed:** April 16, 2026

---

## 1. Introduction

This Data Processing Agreement ("DPA") supplements the [Terms of Service](./terms-of-service.md) and governs the processing of personal data by Thalian, LLC ("Processor") on behalf of the customer organization ("Controller") when using the Thalian platform ("Service").

This DPA applies where Thalian processes personal data on the Controller's behalf in the course of providing the Service. For purposes of this DPA, "personal data," "processing," "data subject," and "supervisory authority" have the meanings given to them in the EU General Data Protection Regulation (GDPR) or equivalent applicable data protection law.

## 2. Roles

- **Controller:** The customer organization that connects integrations and manages workspace data
- **Processor:** Thalian, LLC, which processes data on the Controller's behalf to provide the Service

## 3. Scope of Processing

### Categories of Data Subjects

- Employees and contractors of the Controller's organization
- Users of the Controller's IT systems (identity provider accounts, device users)

### Categories of Personal Data

| Category | Examples |
|---|---|
| **Identity data** | Names, email addresses, usernames, employee IDs |
| **Access data** | Roles, group memberships, application entitlements, MFA status |
| **Device data** | Device names, serial numbers, OS versions, compliance status |
| **Activity data** | Last login dates, application usage timestamps, sign-in events |
| **Security data** | MFA enrollment status, password change dates, session activity |
| **HR data** (if HR integration connected) | Employment status, start and termination dates, department, job title, manager — synced from HR systems (Rippling, BambooHR) to enable offboarding gap detection and identity lifecycle analysis |

### Purpose of Processing

Processing is performed solely to provide the Service as described in the Terms of Service, including:

- Syncing and storing data from connected integrations
- Running AI analysis to generate findings and insights
- Enabling remediation workflows
- Generating reports and audit trails

## 4. Processor Obligations

Thalian shall:

- Process personal data only on documented instructions from the Controller (i.e., through the Service configuration)
- Ensure that persons authorized to process personal data have committed themselves to confidentiality
- Implement appropriate technical and organizational measures as described in the [Information Security Policy](./information-security-policy.md)
- Not engage a sub-processor without prior authorization (see Section 5)
- Assist the Controller in responding to data subject requests (access, deletion, portability)
- Delete or return all personal data upon termination of the Service, after the 30-day retention period
- Make available all information necessary to demonstrate compliance with this DPA

## 5. Sub-Processors

Thalian uses the following sub-processors:

| Sub-Processor | Service | Data Processed | Location |
|---|---|---|---|
| **Supabase** | Database and authentication | All Customer Data | United States (AWS us-east-1) |
| **Cloudflare** | Application hosting, CDN, Workers | Request routing, static assets, backend execution | Global edge; customer data at rest remains US-only |
| **Anthropic** | AI inference (Claude API) | Workspace context in AI prompts ([details](./ai-transparency.md#3-what-data-enters-ai-prompts)) | United States |
| **Stripe** | Payment processing | Billing data | United States |
| **Loops** | Transactional and lifecycle email | Account email address, display name, workspace plan | United States |
| **Sentry** | Error monitoring | Technical error data (PII scrubbed; 10% trace sampling) | United States |
| **Plain.com** | Support chat widget | Name, email address, support conversation content | United States |

The Controller authorizes the use of the sub-processors listed above. Thalian will notify the Controller of any intended changes to sub-processors (additions or replacements) with at least **14 days' advance notice** before the change takes effect, giving the Controller the opportunity to object. The complete sub-processor registry is maintained at [thalian.ai/subprocessors](https://thalian.ai/subprocessors).

## 6. Security Measures

Thalian implements the following technical and organizational measures:

- **Encryption at rest:** AES-256-GCM for integration credentials; Supabase transparent encryption for database
- **Encryption in transit:** TLS 1.2+ on all connections
- **Access control:** Role-based access with six permission levels; Row Level Security on all tables
- **Audit logging:** Immutable, SHA-256 hashed logs of all security-relevant actions
- **Data isolation:** Workspace-scoped queries at application and database layers prevent cross-tenant access
- **Credential management:** Secrets stored as environment variables, never in source code
- **Incident response:** Thalian maintains a documented incident response plan; available to customers upon request under NDA

## 7. Data Subject Rights

Thalian will assist the Controller in fulfilling data subject requests:

- **Access:** Workspace data export is available via the Settings page
- **Deletion:** Contact support@thalian.ai to request data deletion
- **Portability:** Workspace export produces a JSON file of all data
- **Rectification:** Data can be updated through the Service or by re-syncing integrations

## 8. Data Breach Notification

For purposes of this DPA, "personal data breach" has the meaning given in GDPR Article 4(12): a breach of security leading to the accidental or unlawful destruction, loss, alteration, unauthorized disclosure of, or access to, personal data.

In the event of a personal data breach:

- Thalian will notify the Controller without undue delay, and no later than 72 hours after becoming aware of the breach
- Notification will include: the nature of the breach, categories of data affected, likely consequences, and measures taken or proposed
- Full breach response procedures are maintained internally and available to customers upon request under NDA

## 9. International Transfers

Where personal data is transferred outside the European Economic Area (EEA) or the United Kingdom (UK):

- **EU transfers:** Transfers of personal data from the EEA to the United States are covered by the EU Standard Contractual Clauses (EU 2021/914, Module 2: Controller to Processor) incorporated herein by reference, or equivalent safeguards provided by each sub-processor
- **UK transfers:** For transfers from the United Kingdom, Thalian relies on the UK International Data Transfer Agreement (IDTA) or the UK Addendum to the EU SCCs, as applicable under ICO guidance
- **Sub-processor compliance:** All US-based sub-processors are required to maintain equivalent transfer mechanisms. Details available on request
- The Controller acknowledges that AI processing (Anthropic) occurs in the United States

## 10. Duration and Termination

- This DPA remains in effect for the duration of the Service agreement
- Upon termination, Thalian will delete Customer Data within 30 days
- Audit logs are retained for a minimum of 365 days. Upon account termination, audit logs are anonymized (personal identifiers removed) and retained for an additional 12 months for legal and compliance purposes, then permanently deleted

## 11. Contact

For DPA inquiries, contact legal@thalian.ai.

---

*For the complete privacy practices, see [Privacy Policy](./privacy-policy.md).*
