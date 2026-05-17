# Privacy Policy

**Thalian, LLC**
**Effective Date:** April 16, 2026
**Last Reviewed:** April 16, 2026

---

## 1. Introduction

This Privacy Policy describes how Thalian, LLC ("Thalian," "we," "us," or "our") collects, uses, and protects information when you use the Thalian platform ("Service"). We are committed to protecting your privacy and handling your data transparently.

## 2. Information We Collect

### Account Information

- Email address
- Name (if provided)
- Google account information (if using Google OAuth sign-in)
- Workspace name and configuration

### Customer Data (from connected integrations)

When you connect third-party platforms, Thalian syncs data including:

- **Identities:** User accounts, roles, MFA status, group memberships, last login dates
- **Applications:** SaaS apps, SSO configuration, OAuth grants, license assignments
- **Devices:** Managed endpoints, compliance status, encryption state, OS versions
- **Entitlements:** Access relationships between identities and applications
- **Audit events:** Platform-specific logs (sign-in events, admin actions) where available

### Integration Credentials

- API tokens, OAuth tokens, and other authentication credentials you provide to connect platforms
- These are encrypted with AES-256-GCM before storage — plaintext is never saved to the database

### HR Data (if HR integration connected)

When you connect an HR system (Rippling, BambooHR), Thalian syncs:

- Employment status, start and termination dates
- Department, job title, and manager relationships

This data is used solely to enable offboarding gap detection and identity lifecycle analysis across connected platforms.

### Usage Data

- Pages visited within the app
- Analysis runs triggered
- AI conversation history (within your workspace)
- Remediation actions taken

### Technical Data

- Browser type and version
- IP address (for session management and optional IP allowlisting)
- Error reports via Sentry (10% trace sampling)

## 3. How We Use Your Information

| Purpose | Data Used |
|---|---|
| Provide the Service | Customer Data, Account Information |
| Generate findings and analysis | Customer Data (processed by AI engine) |
| AI conversations | Workspace context, conversation history |
| Account management | Account Information, Usage Data |
| Security and fraud prevention | Technical Data, IP addresses |
| Error monitoring | Technical Data (via Sentry) |
| Billing | Account Information (via Stripe) |

## 4. AI Data Processing

### How AI Analysis Works

- The AI engine processes your workspace data to generate findings, insights, and recommendations
- When you use the AI chat feature, relevant workspace context (identities, applications, findings, etc.) is included in prompts sent to AI providers

### Third-Party AI Providers

- **Anthropic (Claude):** AI provider for analysis and chat

### AI Data Commitments

- Your data is not used to train AI models by our providers (per their enterprise/API terms)
- AI prompts contain workspace-scoped data only — no cross-tenant data is included
- AI conversation history is stored within your workspace and subject to your plan's data retention period
- We do not share AI outputs with other customers

For a detailed breakdown of what data enters AI prompts, what is excluded, human oversight controls, and accuracy limitations, see [AI Transparency](./ai-transparency.md).

## 5. Data Storage and Security

### Where Data Is Stored

- **Database:** Supabase (PostgreSQL), hosted in the US
- **Application:** Cloudflare Pages and Workers (global edge network)
- **File storage:** Cloudflare R2 (if applicable)

### Security Measures

- AES-256-GCM encryption for integration credentials at rest
- TLS 1.2+ for all data in transit
- Row Level Security (RLS) on all database tables
- Workspace-scoped queries at both application and database layers
- SHA-256 hashed, immutable audit logs
- Configurable session timeouts and MFA enforcement
- Full details in our [Information Security Policy](./information-security-policy.md)

## 6. Data Retention

| Data Type | Free Plan | Pro Plan | Enterprise |
|---|---|---|---|
| Customer Data | 7 days | 1 year | Unlimited |
| AI conversation history | 7 days | 1 year | Unlimited |
| Audit logs | 365 days minimum | 365 days minimum | 365 days minimum |
| Account information | Duration of account | Duration of account | Duration of account |

- Data exceeding the retention window is automatically deleted by the retention enforcement system
- Audit logs are exempt from automated deletion — minimum 365-day retention regardless of plan
- Upon account termination, data is retained for 30 days and then permanently deleted

## 7. Data Sharing

We do not sell your data. We share data only in these circumstances:

| Recipient | Purpose | Data Shared |
|---|---|---|
| **Anthropic** | AI inference (Claude API) | Workspace context in AI prompts |
| **Stripe** | Payment processing | Billing information |
| **Loops** | Transactional and lifecycle email | Account email, display name, workspace plan |
| **Sentry** | Error monitoring | Technical error data (PII scrubbed; no Customer Data) |
| **Supabase** | Database hosting | All Customer Data (encrypted at rest) |
| **Cloudflare** | Application hosting and CDN | Request routing metadata |
| **Plain.com** | Support chat widget | Name, email, support conversation content |

The complete sub-processor registry, including locations and data processed, is maintained at [thalian.ai/subprocessors](https://thalian.ai/subprocessors).

We may also disclose data if required by law, court order, or to protect our legal rights.

## 8. Your Rights

Depending on your jurisdiction, you may have the following rights:

- **Access:** Request a copy of your data (available via workspace export)
- **Correction:** Update your account information at any time
- **Deletion:** Request deletion of your account and data
- **Portability:** Export your workspace data as JSON
- **Objection:** Object to processing of your data for specific purposes

To exercise these rights, contact privacy@thalian.ai.

### California Residents (CCPA)

- We do not sell personal information
- We do not share personal information for cross-context behavioral advertising
- You have the right to know, delete, and opt-out

### European Residents (GDPR)

- Our legal basis for processing is contract performance (providing the Service) and legitimate interest (security, fraud prevention)
- For data processing on behalf of your organization, see our [Data Processing Agreement](./data-processing-agreement.md)
- You have the right to lodge a complaint with your local data protection authority

## 9. Cookies

The Thalian platform uses only essential cookies required for authentication and session management. We do not use tracking cookies, advertising cookies, or analytics cookies. We respect your browser's Do Not Track settings.

## 10. Children's Privacy

The Service is not intended for individuals under 18 years of age. We do not knowingly collect personal information from children.

## 11. Changes to This Policy

We may update this Privacy Policy from time to time. Material changes will be communicated via email to workspace administrators. The "Last Reviewed" date at the top indicates the most recent revision.

## 12. Contact

For privacy questions or data requests, contact us at privacy@thalian.ai.

---

*For information on how we protect your data technically, see [Information Security Policy](./information-security-policy.md).*
