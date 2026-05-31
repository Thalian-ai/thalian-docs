# API Reference

Thalian exposes a set of backend API endpoints via Cloudflare Pages Functions. All endpoints require authentication and are scoped to your workspace.

---

## Authentication

Every API request must include a valid Supabase session token in the `Authorization` header:

```
Authorization: Bearer <your-session-token>
```

Tokens are issued by Supabase Auth on login. If your workspace has **IP allowlisting** enabled in **Settings** → **Security**, requests from non-allowed IPs are rejected.

## Authorization

All endpoints enforce role-based access control. The backend calls `verifyPermission()` on every request, checking both workspace membership and role-specific permissions before processing. Unauthorized requests receive a `403` response with no internal details.

For a full breakdown of role permissions, see [Settings & Admin](./settings-and-admin.md).

## Base URL

All API endpoints are served from:

```
https://app.thalian.ai/api/
```

## Response Format

All responses are `Content-Type: application/json`. Successful responses return the documented fields. Error responses always follow this shape:

```json
{
  "error": "Human-readable description of the error"
}
```

No stack traces, internal paths, or credential fragments are ever included in error responses.

## Common Field Types

| Type | Format |
|---|---|
| UUID | RFC 4122 — `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| Timestamp | ISO 8601 UTC — `2026-04-15T12:30:00Z` |
| Severity | `"critical"` \| `"high"` \| `"medium"` \| `"low"` \| `"info"` |
| Plan | `"free"` \| `"trial"` \| `"pro"` \| `"enterprise"` |
| Identity status | `"active"` \| `"suspended"` \| `"deprovisioned"` |
| Integration status | `"connected"` \| `"disconnected"` \| `"error"` \| `"paused"` \| `"pending"` |

---

## MCP API (API key auth)

Thalian exposes a separate set of endpoints under `/api/mcp/` that accept API keys instead of Supabase session tokens. These are designed for programmatic access from external tools like Claude Code via the [MCP server](./mcp-server.md).

### API key authentication

Generate a key in **Settings → API Keys**. Pass it as a Bearer token:

```
Authorization: Bearer thal_your_key_here
```

API keys are workspace-scoped. Read-only keys (the default) can call all query tools but cannot modify workspace data. Write-scope keys are required for action tools and are recorded in the audit log on every mutating call. They do not require Supabase Auth and work from any environment that can reach `mcp.thalian.ai`.

### MCP endpoints

All MCP endpoints accept an optional `workspaceId` query parameter. If omitted, the workspace associated with the API key is used. Endpoints marked **write** require a write-scope API key.

| Method | Path | Description | Scope |
|---|---|---|---|
| `GET` | `/api/mcp/risk-score` | Risk score (0–100), severity breakdown, top 5 open findings | read |
| `GET` | `/api/mcp/findings` | Open findings — filterable by `severity`, `category`, `limit` | read |
| `GET` | `/api/mcp/identity` | Identity details by `email` — status, MFA, app access, linked findings | read |
| `GET` | `/api/mcp/integrations` | Connected platforms and sync status | read |
| `GET` | `/api/mcp/posture-summary` | Plain-language executive summary with key metrics | read |
| `GET` | `/api/mcp/rules` | Active detection rules with severity, category, and affected entity types | read |
| `GET` | `/api/mcp/app-policy` | Application policy status by `app_name` — sanctioned, unauthorized, blocked, or none | read |
| `POST` | `/api/mcp/sync` | Trigger a background sync for all connected integrations | **write** |
| `POST` | `/api/mcp/findings/snooze` | Snooze an open finding for 1–90 days | **write** |
| `POST` | `/api/mcp/findings/dismiss` | Dismiss an open finding | **write** |
| `POST` | `/api/mcp/findings/remediate` | Queue a remediation action for an open finding | **write** |
| `POST` | `/api/mcp/app-policy` | Set an application's policy status | **write** |

### GET /api/mcp/risk-score

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | No | Defaults to key's workspace |

**Response:**

```json
{
  "score": 72,
  "label": "High",
  "severity_counts": {
    "critical": 3,
    "high": 14,
    "medium": 28,
    "low": 41
  },
  "total_findings": 86,
  "top_risks": [
    {
      "title": "string",
      "severity": "critical",
      "category": "string",
      "affected_count": 5
    }
  ]
}
```

`label` is one of `"Low"`, `"Medium"`, `"High"`, or `"Critical"`.

### GET /api/mcp/findings

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | No | Defaults to key's workspace |
| `severity` | string | No | Comma-separated filter: `"critical,high"` |
| `category` | string | No | Single category name |
| `limit` | integer | No | 1–100, default `25` |

**Response:**

```json
{
  "findings": [
    {
      "id": "uuid",
      "title": "string",
      "severity": "high",
      "category": "string",
      "status": "open",
      "rule_id": "string",
      "affected_entities": {},
      "recommended_action": "string",
      "created_at": "2026-04-15T12:00:00Z"
    }
  ],
  "total": 86,
  "returned": 25
}
```

### GET /api/mcp/identity

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | No | Defaults to key's workspace |
| `email` | string | Yes | URL-encoded email address. Case-insensitive. |

**Response:**

```json
{
  "email": "alice@example.com",
  "display_name": "string",
  "status": "active",
  "identity_type": "user",
  "mfa_enabled": true,
  "last_login_at": "2026-04-10T08:00:00Z",
  "department": "Engineering",
  "manager_email": "bob@example.com",
  "platforms": ["okta", "google_workspace"],
  "findings": [
    {
      "title": "string",
      "severity": "high",
      "category": "string"
    }
  ],
  "app_access": [
    {
      "app_name": "string",
      "role": "string",
      "last_used_at": "2026-04-12T14:00:00Z"
    }
  ]
}
```

`identity_type` is one of `"user"`, `"admin"`, `"external"`, or `"service_account"`. Returns `404` if no identity with that email exists in the workspace.

### GET /api/mcp/integrations

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | No | Defaults to key's workspace |

**Response:**

```json
{
  "integrations": [
    {
      "platform": "okta",
      "category": "idp",
      "status": "connected",
      "last_sync_at": "2026-04-15T11:00:00Z",
      "idp_role": "primary_idp"
    }
  ],
  "connected_count": 4,
  "total_count": 6
}
```

`idp_role` is present only for identity providers and is one of `"primary_idp"` or `"directory"`.

### GET /api/mcp/posture-summary

Returns a deterministic plain-language summary — no AI call is made.

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | No | Defaults to key's workspace |

**Response:**

```json
{
  "summary": "string",
  "metrics": {
    "risk_score": 72,
    "risk_label": "High",
    "total_findings": 86,
    "severity_counts": {
      "critical": 3,
      "high": 14,
      "medium": 28,
      "low": 41
    },
    "mfa_coverage_pct": 84,
    "admin_pct": 12,
    "sso_coverage_pct": 91,
    "shadow_it_count": 7,
    "connected_integrations": 4,
    "identity_count": 120
  }
}
```

Coverage metrics (`mfa_coverage_pct`, `admin_pct`, `sso_coverage_pct`, `shadow_it_count`) are omitted when the data is unavailable for the workspace.

### POST /api/mcp/sync

Triggers a background sync for all connected integrations. Returns immediately — results appear within minutes.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | No | Defaults to key's workspace |

**Response:**

```json
{
  "status": "triggered",
  "integrations_queued": 4,
  "platforms": ["okta", "google_workspace", "github", "aws"],
  "message": "Sync triggered for 4 integrations"
}
```

`status` is `"triggered"` when at least one integration was queued, or `"no_integrations"` when the workspace has no connected integrations.

### API key management

| Method | Path | Description | Min role |
|---|---|---|---|
| `POST` | `/api/api-keys` | Create a new API key (returned once in plaintext) | Admin |
| `GET` | `/api/api-keys` | List active keys (prefix and metadata only — full key never returned) | Admin |
| `DELETE` | `/api/api-keys` | Revoke a key immediately | Admin |

**GET /api/api-keys — query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | Workspace to list keys for |

**GET response:**

```json
{
  "keys": [
    {
      "id": "uuid",
      "name": "string",
      "prefix": "thal_abc1",
      "created_at": "2026-01-10T09:00:00Z",
      "last_used_at": "2026-04-14T16:30:00Z"
    }
  ]
}
```

**POST /api/api-keys — request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |
| `name` | string | Yes | 1–100 characters. Max 10 active keys per workspace. |
| `scope` | string | No | `read` or `write`. Default `read`. A `write` key can call action endpoints; a `read` key is limited to `GET`. |

**POST response:**

```json
{
  "key": "thal_xxxxxxxxxxxxxxxxxxxx",
  "prefix": "thal_xxxx",
  "name": "string",
  "created_at": "2026-04-15T12:00:00Z"
}
```

The full key is returned once and never stored in plaintext. Save it immediately. Minting a key requires the **Manage workspace** permission, and the chosen scope is recorded in the audit log.

**DELETE /api/api-keys — request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |
| `keyId` | UUID | Yes | ID of the key to revoke |

**DELETE response:**

```json
{
  "revoked": true,
  "revoked_at": "2026-04-15T12:00:00Z"
}
```

---

## Headless API (`/api/v1`)

The `/api/v1` surface exposes read-only findings and inventory data for programmatic consumers such as CI pipelines, scripts, and external tools, without the web UI. It uses the same `thal_*` API keys as the [MCP API](#mcp-api-api-key-auth) and is always scoped to the workspace the key belongs to.

### Authentication

Pass a key generated in **Settings → API Keys** as a Bearer token:

```
Authorization: Bearer thal_your_key_here
```

The key encodes its workspace. Every response is scoped to that workspace, and there is no `workspaceId` parameter to override it, so a key can never reach another workspace's data. Read-scope keys (the default) may call the `GET` endpoints; write-scope keys may also call action endpoints such as `POST /api/v1/scan`. A request whose HTTP method is not granted by the key's scope receives `403`.

### Headless endpoints

| Method | Path | Description | Scope |
|---|---|---|---|
| `GET` | `/api/v1/findings` | Open findings with a curated field set | read |
| `GET` | `/api/v1/inventory` | Identities, applications, or devices | read |
| `POST` | `/api/v1/scan` | Trigger a fresh analysis of the workspace | write |

### GET /api/v1/findings

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `status` | string | No | Finding status. Default `"open"`. |
| `severity` | string | No | One of `critical`, `high`, `medium`, `low`. Omit for all severities. Invalid values return `400`. |
| `limit` | integer | No | 1–200, default `100` |
| `offset` | integer | No | Pagination offset, default `0` |

Findings closed only because their source integration was removed are excluded. Results are ordered by `created_at` descending.

**Response:**

```json
{
  "findings": [
    {
      "id": "uuid",
      "finding_key": "string",
      "rule_id": "string",
      "title": "string",
      "severity": "high",
      "category": "string",
      "status": "open",
      "affected_entities": {},
      "source_integrations": ["okta"],
      "created_at": "2026-04-15T12:00:00Z",
      "updated_at": "2026-04-15T12:00:00Z"
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 86
  }
}
```

`pagination.total` is the count of findings matching the filter across all pages, or `null` when the count is unavailable.

### GET /api/v1/inventory

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `type` | string | No | `identities`, `applications`, or `devices`. Default `identities`. Invalid values return `400`. |
| `limit` | integer | No | 1–200, default `100` |
| `offset` | integer | No | Pagination offset, default `0` |

Each collection returns a stable, versioned field subset. The raw upstream payload (`raw_data`) and internal sync columns are never exposed, so the public contract stays decoupled from the underlying schema. Results are ordered by `created_at` descending.

**`identities` fields:** `id`, `external_id`, `email`, `display_name`, `department`, `title`, `identity_type`, `status`, `mfa_enabled`, `last_login_at`, `created_at`, `updated_at`

**`applications` fields:** `id`, `external_id`, `name`, `category`, `sso_status`, `auth_method`, `assigned_user_count`, `license_count`, `owner`, `sanction_status`, `discovery_source`, `created_at`, `updated_at`

**`devices` fields:** `id`, `external_id`, `device_name`, `device_type`, `platform`, `os_version`, `compliance_status`, `encryption_enabled`, `mdm_enrolled`, `owner_email`, `last_check_in_at`, `created_at`, `updated_at`

**Response:**

```json
{
  "type": "identities",
  "items": [
    {
      "id": "uuid",
      "external_id": "string",
      "email": "alice@example.com",
      "display_name": "string",
      "identity_type": "user",
      "status": "active",
      "mfa_enabled": true,
      "created_at": "2026-04-15T12:00:00Z",
      "updated_at": "2026-04-15T12:00:00Z"
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 120
  }
}
```

### POST /api/v1/scan

Triggers a fresh analysis of the workspace, the same run the web UI performs when you click **Re-analyze**. Requires a **write-scope** key. The request takes no body; the workspace is derived from the key.

A scan can be triggered at most once every 60 seconds per workspace. A request inside that window returns `429` with a `retryAfter` value in seconds. The call blocks until analysis completes, then returns the findings count. Every successful trigger is recorded in the audit log against the calling key.

**Response:**

```json
{
  "status": "completed",
  "findings": 86,
  "analyzed_at": "2026-05-31T23:40:00Z"
}
```

`findings` is the number of open findings after the run, or `null` when the count is unavailable.

| Status | Meaning |
|---|---|
| `200` | Analysis completed |
| `403` | Key lacks `write` scope |
| `429` | Cooldown active; retry after `retryAfter` seconds |
| `502` | Analysis backend failed |

---

## Endpoints

### Health

| Method | Path | Description | Min role |
|---|---|---|---|
| `GET` | `/api/health` | Check platform and database connectivity | — (public) |

**Response:**

```json
{
  "status": "ok",
  "timestamp": "2026-04-15T12:00:00Z",
  "services": {
    "database": "ok"
  },
  "latencyMs": 12
}
```

`status` is `"ok"` or `"degraded"`. `services.database` is `"ok"`, `"degraded"`, or `"unreachable"`. Returns `503` when the database is unreachable.

### Analysis

| Method | Path | Description | Min role |
|---|---|---|---|
| `POST` | `/api/analyze` | Trigger an analysis run for the workspace | Agent |

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |

**Response:**

```json
{
  "findings": [
    {
      "id": "uuid",
      "title": "string",
      "severity": "high",
      "category": "string",
      "status": "open",
      "rule_id": "string",
      "affected_entities": {},
      "affected_identity_id": "uuid",
      "affected_application_id": "uuid",
      "affected_device_id": "uuid",
      "recommended_action": "string",
      "action_type": "string",
      "finding_key": "string",
      "source_integrations": ["okta"],
      "confidence": 0.95,
      "created_at": "2026-04-15T12:00:00Z"
    }
  ],
  "risk_score": 72,
  "severity_counts": {
    "critical": 3,
    "high": 14,
    "medium": 28,
    "low": 41
  },
  "analysis_duration_ms": 1240,
  "last_synced_at": "2026-04-15T11:00:00Z"
}
```

`affected_identity_id`, `affected_application_id`, and `affected_device_id` are present only when the finding is linked to a specific entity.

**Additional error conditions:**

| Code | Condition |
|---|---|
| `409` | Analysis already in progress — rate limited to 1 run per 60 seconds per workspace |

### AI Chat

| Method | Path | Description | Min role |
|---|---|---|---|
| `POST` | `/api/ai-chat` | Send a message to the AI assistant with workspace context | Viewer |

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |
| `message` | string | Yes | The user message |
| `sessionId` | UUID | No | Conversation session ID for multi-turn context |
| `fileBase64` | string | No | Base64-encoded file attachment. Max 10 MB. |
| `fileName` | string | No | Filename for the attachment (used for MIME detection) |
| `confirmationToken` | string | No | Required when approving a high-risk action returned in a previous response |

Supported attachment MIME types: `application/pdf`, `image/jpeg`, `image/png`, `image/webp`, `image/gif`, `text/plain`.

**Response:**

```json
{
  "response": "string",
  "toolCalls": [
    {
      "id": "string",
      "name": "suspend_user",
      "input": {},
      "requiresConfirmation": true
    }
  ],
  "tokens_used": 1840,
  "model": "claude-opus-4-7",
  "stop_reason": "end_turn"
}
```

`toolCalls` is present only when the AI invoked one or more tools. `stop_reason` is one of `"end_turn"`, `"tool_use"`, or `"max_tokens"`.

**Confirmation flow:** When a tool call has `requiresConfirmation: true`, re-send the same request with the `confirmationToken` field populated to approve execution. Confirmation tokens expire after 5 minutes.

**Tool actions:**

Actions are split into two tiers:

- **Immediate** (no confirmation required): `acknowledge_finding`, `dismiss_finding`, `snooze_finding`, `star_finding`, `sanction_app`, `create_ticket`, `trigger_sync`
- **Requires confirmation**: `suspend_user`, `force_password_change`, `revoke_sessions`, `revoke_oauth_token`, `contain_host`, `block_app`, `create_access_review`

**Additional error conditions:**

| Code | Condition |
|---|---|
| `429` | Anthropic API rate limit exceeded |
| `503` | Anthropic API unavailable (retried 3× with exponential backoff) |

### Integrations

| Method | Path | Description | Min role |
|---|---|---|---|
| `POST` | `/api/connect-integration` | Connect a new integration | Admin |
| `POST` | `/api/sync-integration` | Trigger a manual sync for a connected integration | Admin |
| `POST` | `/api/disconnect-integration` | Remove an integration from the workspace | Admin |

#### POST /api/connect-integration

Integration credentials are validated against the provider's API before being saved. Credentials are encrypted with AES-256-GCM before storage — plaintext credentials are never persisted.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |
| `platform` | string | Yes | Platform identifier, e.g. `"okta"`, `"google_workspace"` |
| `config` | object | Yes | Platform-specific credentials object. Fields vary by platform — see [Integrations Guide](./integrations-guide.md) for required fields per platform. |

**Response:**

```json
{
  "success": true,
  "integrationId": "uuid"
}
```

**Additional error conditions:**

| Code | Condition |
|---|---|
| `403` | Integration limit reached for your plan |
| `404` | Platform not supported |
| `422` | Credential validation failed — invalid domain, API key, or credentials rejected by the provider |

#### POST /api/sync-integration

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |
| `integrationId` | UUID | Yes | ID of the integration to sync |

**Response:**

```json
{
  "status": "completed",
  "platform": "okta",
  "entities_synced": {
    "identities": 120,
    "applications": 34,
    "devices": 88,
    "entitlements": 410,
    "audit_events": 2500
  },
  "duration_ms": 3820,
  "next_auto_sync_at": "2026-04-15T14:00:00Z"
}
```

`status` is `"queued"`, `"in_progress"`, or `"completed"`.

**Additional error conditions:**

| Code | Condition |
|---|---|
| `404` | Integration not found |
| `409` | Sync already in progress for this integration |

#### POST /api/disconnect-integration

Removes the integration, wipes credentials, and cancels any pending remediation actions for that platform. Existing findings are retained — they remain open but will not be refreshed.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |
| `integrationId` | UUID | Yes | ID of the integration to disconnect |

**Response:**

```json
{
  "success": true,
  "platform": "okta",
  "disconnected_at": "2026-04-15T12:00:00Z",
  "pending_actions_cancelled": 2
}
```

**Additional error conditions:**

| Code | Condition |
|---|---|
| `404` | Integration not found |

### Remediation

| Method | Path | Description | Min role |
|---|---|---|---|
| `POST` | `/api/execute-action` | Execute a remediation action against a target platform | Agent |

**Approval queue:** Actions initiated by Agent-role users on high or critical severity findings enter a pending approval queue. Security Analysts, Admins, and Super Admins can execute without a second approver.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |
| `actionType` | string | Yes | The remediation action to execute |
| `targetPlatform` | string | Yes | Platform to act on, e.g. `"okta"`, `"google_workspace"` |
| `targetEntityId` | string | Yes | ID of the entity to act on (user ID, app ID, etc.) |
| `targetEntityLabel` | string | No | Human-readable label for audit log display |
| `riskId` | UUID | No | Finding ID this action is resolving |
| `extraParams` | object | No | Action-specific additional parameters |
| `reason` | string | No | Reason for the action, recorded in the audit log |

**Response:**

```json
{
  "success": true,
  "actionId": "uuid",
  "result": {}
}
```

`result` shape varies by `actionType`. All executions are recorded in the audit log.

**Additional error conditions:**

| Code | Condition |
|---|---|
| `403` | Trial expired |
| `404` | Action not found or confirmation token expired |

### Export

| Method | Path | Description | Min role |
|---|---|---|---|
| `POST` | `/api/export-audit-log` | Export the full audit log as JSON | Auditor |
| `POST` | `/api/export-workspace` | Export all workspace data | Admin |

#### POST /api/export-audit-log

Enterprise plan only. Returns up to 365 days of audit log entries. Each entry includes a SHA-256 content hash for tamper detection. Response includes a `Content-Disposition: attachment` header.

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |

**Response:**

```json
{
  "exported_at": "2026-04-15T12:00:00Z",
  "workspace_id": "uuid",
  "total_entries": 4820,
  "entries": [
    {
      "id": "uuid",
      "workspace_id": "uuid",
      "user_id": "uuid",
      "action": "string",
      "actor_email": "string",
      "target_type": "string",
      "target_id": "uuid",
      "details": {},
      "content_hash": "sha256-hex",
      "created_at": "2026-04-15T12:00:00Z"
    }
  ]
}
```

**Additional error conditions:**

| Code | Condition |
|---|---|
| `403` | Workspace is not on the Enterprise plan |

#### POST /api/export-workspace

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |
| `format` | string | Yes | `"json"`, `"csv"`, or `"xlsx"` |
| `includeAuditLog` | boolean | No | Include audit log entries. Default `false`. |
| `includeFull` | boolean | No | Include all raw entity metadata. Default `false`. |

CSV and XLSX responses are returned with the appropriate `Content-Type` and `Content-Disposition: attachment` headers.

**Response (JSON format):**

```json
{
  "exported_at": "2026-04-15T12:00:00Z",
  "workspace_id": "uuid",
  "format": "json",
  "data": {
    "identities": [],
    "applications": [],
    "devices": [],
    "findings": [],
    "audit_log": []
  },
  "entity_counts": {
    "identities": 120,
    "applications": 34,
    "devices": 88,
    "findings": 86
  },
  "truncated": false
}
```

If the export exceeds size limits, `truncated` is `true` and a `cursor` field is included for pagination.

### Billing

| Method | Path | Description | Min role |
|---|---|---|---|
| `POST` | `/api/create-checkout` | Create a Stripe checkout session for plan upgrade | Admin |
| `POST` | `/api/create-portal-session` | Open the Stripe customer portal for billing management | Admin |

#### POST /api/create-checkout

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |
| `email` | string | Yes | Billing email address (RFC 5321) |
| `interval` | string | No | `"monthly"` or `"annual"`. Default `"monthly"`. |

**Response:**

```json
{
  "url": "https://checkout.stripe.com/...",
  "sessionId": "string",
  "interval": "monthly",
  "priceId": "string",
  "expiresAt": "2026-04-16T12:00:00Z"
}
```

Redirect the user to `url` to complete checkout. Sessions expire after 24 hours.

**Additional error conditions:**

| Code | Condition |
|---|---|
| `409` | Workspace already has an active subscription at the requested interval |

#### POST /api/create-portal-session

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `workspaceId` | UUID | Yes | |

**Response:**

```json
{
  "url": "https://billing.stripe.com/...",
  "expiresAt": "2026-04-16T12:00:00Z"
}
```

Redirect the user to `url` to manage their subscription. Sessions expire after 24 hours.

**Additional error conditions:**

| Code | Condition |
|---|---|
| `404` | No Stripe customer record found for this workspace |
| `500` | Stripe API error |

---

## Request Format

All `POST` endpoints accept JSON request bodies with `Content-Type: application/json`. Every request must include a `workspaceId` field:

```json
{
  "workspaceId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  ...
}
```

The backend verifies that the authenticated user is a member of the specified workspace before processing any query. `workspaceId` must be a valid UUID — malformed values return `400`.

## Error Handling

API errors return standard HTTP status codes:

| Code | Meaning |
|---|---|
| `400` | Invalid request body, missing required fields, or malformed values |
| `401` | Missing or invalid session token or API key |
| `403` | Insufficient role permissions, IP not allowlisted, workspace mismatch, or trial expired |
| `404` | Resource not found within workspace scope |
| `409` | Conflict — e.g. analysis in progress, duplicate subscription, sync already running |
| `422` | Validation failed against external provider (credential check) |
| `429` | Rate limit exceeded |
| `500` | Internal server error or upstream API error |
| `503` | Upstream service unavailable (e.g. Anthropic API) |

Error responses are sanitized — no stack traces, internal paths, or credential fragments are included.

## Rate Limiting

| Endpoint | Limit | Window |
|---|---|---|
| `/api/analyze` | 1 run | 60 seconds per workspace |
| `/api/ai-chat` (Free) | 25 messages | Per day |
| `/api/ai-chat` (Pro) | 100 messages | Per day |
| `/api/ai-chat` (Enterprise) | Unlimited | — |
| `/api/api-keys` (POST) | 10 active keys | Per workspace |
| MCP endpoints | 1,000 requests | Per hour per API key |

## Security Headers

All API responses include the following security headers:

| Header | Value |
|---|---|
| `X-Frame-Options` | `DENY` |
| `X-Content-Type-Options` | `nosniff` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` |

---

*For information on connecting platforms and what data each integration syncs, see [Integrations Guide](./integrations-guide.md).*
