---
title: Sub-Processor Registry
last_updated: May 28, 2026
---

# Sub-Processor Registry

This is the canonical list of third-party sub-processors that Thalian, Inc. engages to process personal data on behalf of customers. All customer-facing legal documents (Terms of Service, Privacy Policy, Data Processing Agreement, and Security & Trust page) derive their sub-processor disclosures from this registry.

**Update process:** When adding or replacing a sub-processor, update this file first, then propagate changes to all documents listed above. Enterprise customers must receive 14 days' advance notice before any addition or replacement takes effect.

---

| Sub-Processor | Service | Personal Data Processed | Location |
|---|---|---|---|
| **Supabase** | Database and authentication | All workspace data (identities, devices, applications, entitlements, findings, audit logs, AI messages) | US (AWS us-east-1) |
| **Cloudflare** | Application hosting, CDN, Workers, R2 | Request routing metadata, static asset delivery, backend function execution | Global edge network; customer data at rest remains US-only |
| **Anthropic** | AI inference (Claude API) | Workspace context included in AI prompts (identities, applications, devices, findings, audit events) | US |
| **Stripe** | Payment processing | Billing information (email, payment method, invoice history) | US |
| **Loops** | Transactional and lifecycle email | Account email address, display name, workspace plan | US |
| **Sentry** | Error monitoring | Technical error data (stack traces, request context); PII scrubbed via beforeSend filter; 10% trace sampling | US |
| **Plain.com** | Support chat widget | Name, email address, support conversation content | US |

---

## Notes

- **SendGrid:** Fully replaced by Loops as of 2026. Not a current sub-processor.
- **GitHub:** Hosts Thalian's source code and CI/CD pipelines. Does not process customer personal data. Not listed as a DPA sub-processor.
- **Google:** Thalian reads from Google Workspace as a data **source** (customer integration). Google is not a processor of Thalian's stored customer data. Excluded from this registry.
- **Anthropic data retention:** Thalian uses the standard commercial API tier. Anthropic retains API inputs and outputs for up to 30 days for trust and safety review. This data is **not** used to train, fine-tune, or improve AI models. Thalian has not enrolled in Anthropic's Zero Data Retention (ZDR) program as of April 2026.

---

*This registry was last reviewed April 16, 2026. Next review: upon any sub-processor change or quarterly, whichever comes first.*
