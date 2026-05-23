# Audit Log Chain of Custody

Thalian's audit log uses a per-workspace SHA-256 hash chain to provide
cryptographic tamper-evidence for SOC 2 Type II evidence collection.

This page documents the design, threat model, and verification procedure.
Aimed at security questionnaires, prospect audits, and the SOC 2 audit binder.

## Overview

Every row in `public.audit_log` carries three chain-related columns introduced
by migration `20260523_audit_log_chain_root.sql`:

- `content_hash` — SHA-256 over the canonical JSON of the row's content fields
  (action, created_at, details, target_id, target_type, user_id, workspace_id).
  Provides per-row tamper detection.
- `row_hash` — SHA-256 over the canonical JSON of all content fields PLUS the
  chain linkage fields (`id`, `prev_hash`, `chain_version`, `sequence_number`,
  microsecond-epoch timestamp). Provides chain linkage.
- `prev_hash` — `row_hash` of the predecessor row within the same workspace
  ordered by `sequence_number DESC`. NULL for the workspace's genesis row.

Each workspace's chain begins with a single `audit_chain.genesis` row containing
a `precutover_seal_root` — a Merkle-style hash over the `(id, row_hash)` pairs
of all pre-cutover rows in that workspace. After cutover, every audit event is
appended as a chained row.

## Design choices

### Per-workspace chain
Multi-tenant isolation. Two workspaces never share chain state. No cross-tenant
contention on inserts. Auditor verifies each workspace's chain independently.

### Ordering via `sequence_number`
A bigint sequence assigned at INSERT time. Strictly monotonic, gap-tolerant
(sequence increments survive transaction rollback). Avoids timestamp ambiguity
on sub-microsecond concurrent inserts.

### Cryptographic primitive: SHA-256
Industry standard for audit trails. Implemented via `extensions.digest()` from
pgcrypto. Backed by OpenSSL on the server.

### Serialization format
The hash input is the UTF-8 text output of PostgreSQL's `json_build_object(...)`,
with keys declared in alphabetical order. The serialization is:

- `{"key1" : value1, "key2" : value2, ...}` (single space around colon, comma
  followed by single space)
- Keys preserve declaration order (we declare them alphabetically)
- Null encoded as `null` (no quotes)
- This format is reproducible in Python via
  `json.dumps(d, separators=(', ', ' : '))` with `d` constructed in alphabetical
  key order

We use `json_build_object` rather than `jsonb_build_object` because the latter
reorders keys by internal storage order (length-bucketed, hash-ordered) and
uses different whitespace. Empirically verified 2026-05-23 against production
rows: the two formats produce different SHA-256 hashes.

### Immutability enforcement
Three triggers on `audit_log` enforce append-only semantics:

- `audit_log_compute_hash` (BEFORE INSERT) — populates `content_hash`,
  `prev_hash`, and `row_hash` per the design above
- `audit_log_no_update` (BEFORE UPDATE) — raises an exception on any UPDATE
- `audit_log_no_delete` (BEFORE DELETE) — raises an exception on any DELETE
- `audit_log_forbid_multi_row` (AFTER INSERT, statement-level) — raises if any
  single INSERT statement adds more than one row, defending against bulk
  insert that would skip chain ordering

Workspace creation auto-creates the genesis row via an AFTER INSERT trigger on
`workspaces`.

## Known limitations

1. **Anonymized rows excluded from chain coverage.** When a workspace is
   deleted, the workspace_id on its audit log rows is set to NULL (per
   migration `20260501_audit_log_preserve_on_workspace_delete.sql`). Individual
   row_hash and content_hash tamper detection survives, but chain linkability
   does not. This is intentional to honor GDPR Article 17 right-to-erasure.

2. **Pre-trigger rows.** Approximately 1,500 rows in the production audit_log
   were inserted between 2026-02-27 and the trigger creation date of 2026-05-18.
   These rows have `content_hash` backfilled at chain migration apply time using
   the canonical formula. They are tamper-detection-protected from migration
   date forward but cannot be retroactively verified to have been unmodified
   before the trigger existed.

3. **Service-role and authenticated-member authenticity.** The chain proves the
   integrity of stored rows. It does not prove the authenticity of the actor
   who inserted them. A compromised service-role key or a malicious workspace
   member with INSERT permission could write chain-valid rows containing false
   information. Mitigated by the principle of least privilege on service-role
   keys and by RLS scoping of authenticated inserts.

4. **Platform-level (workspace_id IS NULL) audit events.** Events not tied to a
   specific workspace (such as `audit_chain.migration_applied`) bypass the
   per-workspace chain but retain individual content_hash and row_hash tamper
   detection. Same orphan-row semantic as (1).

## Verification

An external Python verifier is on the SOC 2 fieldwork roadmap, scheduled to
ship before 2026-08-15 (in advance of the Type II audit fieldwork window).

The verifier takes a CSV or JSONL snapshot of `audit_log` rows and:

1. Recomputes `content_hash` for each row from the canonical fields and
   compares against the stored value
2. Recomputes `row_hash` for each row including chain linkage fields and
   compares against the stored value
3. Verifies each non-genesis row's `prev_hash` matches the predecessor's
   `row_hash` in `(workspace_id, sequence_number DESC)` order
4. Recomputes the `precutover_seal_root` in each workspace's genesis row
   from the (id, row_hash) pairs of pre-cutover rows
5. Reports per-workspace pass/fail + count of verified rows + any break points

Until the verifier ships, manual verification can be performed in psql:

```sql
-- Verify the chain links within a single workspace
SELECT
  a.sequence_number,
  a.id,
  a.action,
  a.prev_hash,
  LAG(a.row_hash) OVER (
    PARTITION BY a.workspace_id
    ORDER BY a.sequence_number
  ) AS expected_prev_hash,
  a.prev_hash = LAG(a.row_hash) OVER (
    PARTITION BY a.workspace_id
    ORDER BY a.sequence_number
  ) AS link_intact
FROM public.audit_log a
WHERE a.workspace_id = $1
  AND a.action != 'audit_chain.genesis'
ORDER BY a.sequence_number;
```

## Cutover date

The chain became active on 2026-05-23 with the application of migration
`20260523_audit_log_chain_root.sql`. Pre-cutover rows are sealed by each
workspace's genesis row's `precutover_seal_root`. Post-cutover rows are
linked into the chain.

## Compliance mapping

The audit log with chain of custody is referenced from:

- SOC 2 CC8.1 (Change Management) — audit_log with SHA-256 tamper detection is
  the primary evidence
- ISO 27001 A.8.15 (Logging) — same
- ISO 42001 A.6.2.8 (AI System Recording of Event Logs) — same

The chain enhances the evidence quality of all three control mappings without
adding new controls.
