# Information Security Policy

**Thalian, Inc.**
**Effective Date:** May 28, 2026
**Last Reviewed:** May 28, 2026
**Owner:** Engineering & Security Team
**Classification:** Public

---

## 1. Purpose

This policy establishes the information security requirements for the Thalian platform, its infrastructure, and the data it processes. Thalian is an AI-powered IT intelligence platform that ingests data from customer identity providers, endpoint managers, security tools, and SaaS applications to surface cross-platform insights.

Because Thalian processes sensitive IT infrastructure data — including user identities, access entitlements, device inventory, and integration credentials — security is a foundational requirement, not a feature.

## 2. Scope

This policy applies to:

- All Thalian employees, contractors, and agents with access to production systems
- All customer data processed by the Thalian platform
- All infrastructure components: Cloudflare Pages (frontend), Cloudflare Workers (backend), Supabase (database and auth), and third-party API integrations
- All environments: production, staging, and development

## 3. Data Classification

| Classification | Description | Examples | Handling |
|---|---|---|---|
| **Restricted** | Credentials and secrets | Integration API tokens, OAuth refresh tokens, encryption keys | AES-256-GCM encrypted at rest; never logged; env vars only |
| **Confidential** | Customer workspace data | Identities, entitlements, findings, audit logs, device inventory | Encrypted in transit (TLS 1.2+); Row Level Security enforced; workspace-scoped queries |
| **Internal** | Platform operational data | Sync logs, drift snapshots, AI conversation history | Retained per plan tier; deleted by automated retention enforcement |
| **Public** | Marketing and documentation | Landing page content, API docs, this policy | No special handling required |

## 4. Access Control

### 4.1 Authentication

- **Primary auth:** Supabase Auth with email/password or Google OAuth
- **Multi-factor authentication:** TOTP-based MFA via Supabase Auth MFA. Workspace administrators can enforce MFA for all members — when enabled, users without a verified TOTP factor are blocked from accessing the dashboard until they enroll
- **Session management:** Configurable session timeouts (1h, 4h, 8h, 24h, 72h) enforced via `useSessionTimeout` hook. Sessions are invalidated server-side on timeout
- **IP allowlisting:** Workspace-level IP allowlist restricts API access to approved networks

### 4.2 Role-Based Access Control (RBAC)

Thalian implements six roles with strictly ordered privilege levels:

| Role | Rank | Key Permissions |
|---|---|---|
| **Super Admin** | 0 | Full workspace control including ownership transfer |
| **Admin** | 1 | Manage workspace, integrations, members, and all operations |
| **Security Analyst** | 2 | View all data, initiate and approve remediation, manage app policies |
| **Agent** | 3 | View all data, initiate remediation (high/critical require approval) |
| **Auditor** | 4 | Read-only access with audit log and export capabilities |
| **Viewer** | 5 | View dashboards, findings, and asset data only |

**Enforcement:**
- Backend: Every API endpoint calls `verifyPermission()` which checks both workspace membership and role-specific permissions before processing any request
- Frontend: UI elements are conditionally rendered based on the `can()` helper from `WorkspaceContext`
- Role hierarchy: Users can only manage roles ranked below their own. No role can modify or elevate itself

### 4.3 Principle of Least Privilege

- New workspace members default to the **Viewer** role
- Super Admin is not available in the invitation flow — it can only be assigned by an existing Super Admin
- Integration credentials are accessible only to the backend service role, never exposed to frontend clients
- API endpoints reject requests that lack the required permission, returning a generic 403 with no internal details

## 5. Encryption

### 5.1 Data at Rest

- **Integration credentials:** Encrypted with AES-256-GCM before storage. A consistent encryption key is derived from the `ENCRYPTION_KEY` environment variable via SHA-256 key derivation. Each encryption operation uses a unique 12-byte random IV. Encrypted records carry an `_enc: 1` flag for identification
- **Database:** Supabase provides transparent encryption at rest for all PostgreSQL data
- **Backward compatibility:** The decryption function handles both encrypted (flagged) and legacy plaintext records gracefully

### 5.2 Data in Transit

- All client-server communication uses TLS 1.2 or higher, enforced by Cloudflare
- HSTS header (`max-age=31536000; includeSubDomains`) prevents protocol downgrade attacks
- WebSocket connections to Supabase Realtime also use WSS (TLS-encrypted WebSocket)

## 6. Audit Logging

### 6.1 Coverage

All security-relevant actions are logged to the `audit_log` table, including:

- Authentication events (login, logout, MFA enrollment)
- Integration lifecycle (connected, disconnected, synced)
- Finding lifecycle (resolved, dismissed, reopened)
- Remediation actions (requested, approved, rejected, executed)
- Team management (invited, joined, role changed, removed)
- Security setting changes (MFA enforcement, session timeout, IP allowlist)
- Billing events (checkout, upgrade, cancellation, payment failure)
- Data operations (workspace export, workspace deletion)

### 6.2 Immutability

Every audit log entry is hashed with SHA-256 using a canonical key representation. The `content_hash` is computed from a deterministic JSON serialization of the entry's core fields (`action`, `created_at`, `details`, `target_id`, `target_type`, `user_id`, `workspace_id`), ensuring:

- **Tamper detection:** Any modification to a log entry invalidates its hash
- **Determinism:** The same entry always produces the same hash regardless of key insertion order
- **Completeness:** Backend inserts use the `insertAuditLog()` helper; frontend inserts are hashed by a PostgreSQL `BEFORE INSERT` trigger

### 6.3 Retention

- Audit log entries are **never deleted** by the automated data retention enforcement system, regardless of workspace plan tier
- Minimum retention guarantee: **365 days** (formalized as `AUDIT_LOG_MIN_RETENTION_DAYS`)
- Audit logs are exportable via the `/api/export-audit-log` endpoint for SIEM integration or offline archival

## 7. Network Security

The following security headers are applied to all routes via `public/_headers`:

| Header | Value | Purpose |
|---|---|---|
| `X-Frame-Options` | `DENY` | Prevents clickjacking |
| `X-Content-Type-Options` | `nosniff` | Prevents MIME-type sniffing |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limits referrer information leakage |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` | Disables unnecessary browser APIs |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Enforces HTTPS |
| `Content-Security-Policy` | Scoped to actual dependencies | Prevents XSS and unauthorized script execution |

**CSP details:** `script-src` allows self and jsdelivr CDN only; `connect-src` is scoped to Supabase, Anthropic, and Stripe endpoints; `frame-ancestors: none` prevents embedding.

## 8. Database Security

### 8.1 Row Level Security (RLS)

All Supabase tables have Row Level Security enabled. Every RLS policy:

- Targets the `authenticated` or `service_role` — never the `public` role
- Uses `(SELECT auth.uid() AS uid)` wrapper for InitPlan caching (performance optimization)
- Scopes access to the authenticated user's workspace(s) via `workspace_members` join

### 8.2 Application-Level Scoping

In addition to RLS (defense in depth), every application query is explicitly scoped with `workspace_id`:

- Backend API endpoints extract `workspaceId` from the request body and verify the authenticated user is a member of that workspace before executing any query
- Cross-tenant data leakage is prevented at both the application layer and the database layer

## 9. Secrets Management

- All secrets (API keys, database credentials, encryption keys, webhook secrets) are stored as **environment variables** in Cloudflare Pages / Workers
- Secrets are **never committed to source code** — the CI pipeline includes an automated secrets scanner that fails the build if potential secrets are detected in `src/`, `functions/`, or `public/`
- Error messages returned to clients are sanitized — no stack traces, internal paths, or credential fragments are ever exposed
- Sensitive data (credentials, tokens, PII) is **never written to application logs**

## 10. Vendor Management

| Vendor | Service | Data Processed | DPA Status |
|---|---|---|---|
| **Supabase** | Database, Auth, Realtime | All workspace data | Covered under Supabase Terms of Service and DPA |
| **Cloudflare** | CDN, Pages, Workers, R2 | Request routing, static assets, backend execution | Covered under Cloudflare DPA |
| **Anthropic** | AI inference (Claude API) | Workspace context sent in AI prompts ([details](./ai-transparency.md)) | 30-day retention for trust & safety review; not used for model training |
| **Stripe** | Payment processing | Customer billing data | Covered under Stripe DPA |
| **Loops** | Transactional and lifecycle email | Account email, display name, workspace plan | Covered under Loops DPA |
| **Sentry** | Error monitoring | Technical error data (PII scrubbed; 10% trace sampling) | Covered under Sentry DPA |
| **Plain.com** | Support chat widget | Name, email, support conversation content | Covered under Plain.com DPA |

## 11. Vulnerability Management

- **Dependency auditing:** `npm audit --audit-level=high` runs on every CI build
- **Secrets scanning:** Automated grep-based scanner checks for leaked credentials on every push
- **Health monitoring:** `/api/health` endpoint checks Supabase connectivity; Sentry captures frontend errors with 10% trace sampling
- **Error boundaries:** React `Sentry.ErrorBoundary` wraps the entire application to prevent unhandled errors from exposing internal state

## 12. Policy Review

This policy is reviewed quarterly or upon any significant change to infrastructure, data handling practices, or regulatory requirements. All changes are tracked via version control (git).

---

*For information on how we handle your data, see [Privacy Policy](./privacy-policy.md).*
