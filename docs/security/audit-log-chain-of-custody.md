# Audit Log — Chain of Custody Trust Model

Thalian's audit log is designed to provide tamper-evidence for SOC 2 Type II attestation. This document describes the design, the guarantees it provides, and the limitations auditors should be aware of.

---

## What the chain proves

Every audit log entry written after the chain was activated (2026-05-23) is linked to all previous entries by a per-workspace SHA-256 hash chain. The chain provides the following guarantees:

- **Append-only**: rows cannot be deleted. A PostgreSQL `BEFORE DELETE` trigger raises an exception on any `DELETE` from `audit_log`.
- **Immutable**: rows cannot be modified after insertion. A PostgreSQL `BEFORE UPDATE` trigger raises an exception on any `UPDATE` from `audit_log`, with a single documented exception (see "Migration 20260523" below).
- **Linked**: each row's `row_hash` covers the `prev_hash` of the row that preceded it in the same workspace. Deleting or inserting a row in the middle of the chain breaks the `prev_hash` linkage on every subsequent row.
- **Pre-cutover sealed**: rows that existed before the chain was activated are covered by a `precutover_seal_root` — a SHA-256 over the canonical `(id:row_hash)` pairs of all pre-cutover rows. This seal is stored in the genesis row and can be recomputed offline.

---

## Hash formula

Each row's `row_hash` is computed by the `compute_audit_chain_hash()` PostgreSQL trigger as:

```
SHA-256(
  json_build_object(
    'action',          <action>,
    'chain_version',   <chain_version>,
    'created_at_us',   <microseconds since epoch, integer-split>,
    'details',         <details JSONB>,
    'id',              <row UUID>,
    'prev_hash',       <prev_hash or null>,
    'sequence_number', <sequence_number>,
    'target_id',       <target_id>,
    'target_type',     <target_type>,
    'user_id',         <user_id>,
    'workspace_id',    <workspace_id>
  )::text
)
```

Key serialization properties (important for external verification):

- Key order is **alphabetical** as declared in `json_build_object` (PostgreSQL preserves declared order for `json`, not `jsonb`).
- Whitespace: outer object uses `" : "` (space colon space) and `", "` (comma space). Inner `details` JSONB values are serialized in PostgreSQL's compact JSONB format (sorted keys, no extra whitespace).
- `created_at_us` is a **bigint** (integer-split to avoid float64 drift on old PostgreSQL versions): `floor(epoch_seconds) * 1_000_000 + microsecond_of_second`.
- The formula uses `json_build_object` (not `jsonb_build_object`) for deterministic key-order output. The two are not bit-identical.

---

## Chain topology

```
[genesis row]
    ↓  prev_hash = genesis.row_hash
[first post-cutover event]
    ↓  prev_hash = first_event.row_hash
[second post-cutover event]
    ↓  ...
[latest event]
```

- **Genesis row** (`action = 'audit_chain.genesis'`): created automatically when a workspace is created (or backfilled for existing workspaces). Contains `precutover_seal_root` and `cutover_at` in its `details` field. `prev_hash` is NULL.
- **Pre-cutover rows**: all rows with `prev_hash IS NULL` and `action != 'audit_chain.genesis'`. Their integrity is covered by `precutover_seal_root`.
- **Chained rows**: all rows with `prev_hash IS NOT NULL`. Each links to the workspace's highest-`sequence_number` row at the time of insert.

Workspace isolation: each workspace has its own independent chain. A PostgreSQL advisory lock (`pg_advisory_xact_lock`) serializes concurrent inserts within a workspace to prevent chain forks.

---

## Verification

### In-product verifier

Compliance → Audit Log → **Chain Integrity** runs the `verify_audit_chain()` PostgreSQL function against live data. This function:

1. Recomputes `precutover_seal_root` from stored `(id, row_hash)` pairs and compares with the genesis row.
2. Walks post-cutover chained rows in sequence order, checking `prev_hash` linkage.
3. Recomputes each post-cutover `row_hash` using the same PostgreSQL formula and compares with the stored value.
4. Returns: `ok` (boolean), `row_count`, `link_break_count`, `hash_break_count`, locations of first breaks.

`verify_audit_chain` is `SECURITY INVOKER` — it runs as the calling user and performs a workspace membership check. It does not expose raw hash chains to the caller.

### Offline verifier (Python)

The public Python verifier (`verify-audit-export.py`, available at `https://github.com/thalian-ai/thalian/blob/main/tools/verify-audit-export.py`) verifies a JSONL export downloaded from Compliance → Audit Log → Download export.

The offline verifier checks:
1. Genesis row present and unique.
2. `precutover_seal_root` matches the recomputed seal over pre-cutover rows.
3. Post-cutover chain linkage (`prev_hash` consistency for every chained row).

**Limitation**: the offline verifier does **not** recompute individual `row_hash` values (hash content integrity). This requires exact reproduction of PostgreSQL's `json_build_object` text serialization, which is not straightforward in Python. Use the in-product verifier for full content integrity verification.

---

## Known limitations (auditor disclosure)

### H2: Service-role credential can fabricate authentic-looking entries

The PostgreSQL `service_role` credential (and `authenticated` users with sufficient Supabase permissions) can insert rows that pass all hash and chain checks. The chain proves **integrity of stored rows** — that no row was modified, deleted, or inserted between two existing rows after the fact. It does not prove that all rows represent genuine Thalian application events.

**Compensating controls** for this limitation:
- The `SUPABASE_SERVICE_KEY` is stored only in Cloudflare Pages environment variables, never logged or committed to source code.
- All direct service-role writes to `audit_log` from application code go through `insertAuditLog()` in `functions/api/_audit.js`, which sets `user_id` from the authenticated JWT — not from a caller-supplied parameter.
- Credential rotation is covered by Thalian's access control policy (CC6.1).
- A follow-up enhancement (app-layer HMAC over each row using a rotating signing key) is planned for after the SOC 2 observation window closes. This will provide authenticity guarantees in addition to the current integrity guarantees.

### Pre-cutover rows (rows before 2026-05-23)

Rows inserted before the chain was activated have `content_hash` computed by the application-layer `_audit.js` helper (using JavaScript's `JSON.stringify`, no extra whitespace). These rows are sealed by `precutover_seal_root` in the genesis row, which covers all of them as a batch.

Individual pre-cutover rows cannot be hash-recomputed by the PostgreSQL verifier because the PostgreSQL `json_build_object` formula produces different whitespace than JavaScript's `JSON.stringify`. The seal root provides batch-level tamper evidence for this cohort.

### Workspace deletion severs chain linkability

When a workspace is deleted, `audit_log.workspace_id` is set to NULL (the FK is `ON DELETE SET NULL`, not CASCADE). The rows are retained for 3 years per Thalian's data processing agreement, but the `workspace_id` column is nulled and the row content is anonymized (`user_id = NULL`, `details = {}`). This satisfies GDPR Article 17 right-to-erasure while preserving the audit trail.

Once `workspace_id` is NULL, chain verification by workspace is no longer possible. Chain integrity at write time was preserved; chain linkability is intentionally severed at deletion.

### Migration 20260523 was the only legitimate UPDATE on audit_log

The `audit_log_no_update` immutability trigger blocks all UPDATEs. Migration `20260523_audit_log_chain_root.sql` temporarily disabled this trigger inside a single `ACCESS EXCLUSIVE LOCK` transaction to:
1. Backfill `content_hash` for ~1,500 rows inserted before the hash trigger existed.
2. Backfill `row_hash = content_hash` for all pre-cutover rows.

No production traffic could observe `audit_log` in a mutable state during this window. The migration is the only exception to the immutability rule. All future maintenance that requires row modification must follow the same pattern (document, lock, disable trigger, update, re-enable, commit) and must be logged here.

---

## Chain version history

| Version | Activated | Description |
|---------|-----------|-------------|
| v1 | 2026-05-23 | Initial chain. Alphabetical `json_build_object`, integer-split microsecond epoch, per-workspace genesis, `precutover_seal_root`. |

A new chain version would be introduced via a cutover row with `action = 'audit_chain.version_bump'` and a new `chain_version` value. Verifiers must handle multiple chain version segments within a single workspace's history.

---

## References

- Python verifier: [github.com/thalian-ai/thalian/blob/main/tools/verify-audit-export.py](https://github.com/thalian-ai/thalian/blob/main/tools/verify-audit-export.py)
