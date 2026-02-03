# RFC 0002: Verification Protocol

## Summary

This RFC defines the verification protocol used by KORA to establish behavioral equivalence between source and target database systems after migration.

## Status

Accepted

## Authors

Sensei Architecture Team

## Date

2025-08-01

---

## Problem Statement

Standard migration validation -- row counts, spot checks, sample queries -- is demonstrably insufficient. Migrations passing these checks routinely contain latent defects: semantic corruption, logic loss, temporal inconsistencies, and silent truncation. A rigorous protocol must answer: "does the target system produce the same business outcomes as the source under equivalent conditions?"

---

## Protocol Phases

```text
CAPTURE -> REPLAY -> COMPARE -> CERTIFY
```

Each phase produces immutable, content-addressed (SHA-256) artifacts consumed by the next.

### Phase 1: Capture

Records source system behavior at defined points in time:

| Component           | Description                                                                     |
| ------------------- | ------------------------------------------------------------------------------- |
| **Schema snapshot** | Complete DDL: tables, columns, constraints, triggers, stored procedures, views. |
| **Query corpus**    | Workload queries from logs, reporting, and synthetic edge cases.                |
| **Result set**      | Query outputs in canonical format (normalized ordering, precision, timestamps). |
| **Temporal marker** | Transaction ID or LSN for point-in-time consistency.                            |

Multiple snapshots at different historical points enable time-travel verification.

### Phase 2: Replay

For each snapshot:

1. Align target state to equivalent logical point via checkpoint metadata.
2. Translate queries from source to target SQL dialect.
3. Execute against the target with statement-level timeouts.
4. Serialize results in canonical format.

Executed by kora-core (Rust). Failed queries are recorded as findings, never skipped.

### Phase 3: Compare

Three levels of comparison:

**Structural.** Same column count, compatible types, same row count.

**Value.** Cell-by-cell with tolerances:

| Data Type | Tolerance                            |
| --------- | ------------------------------------ |
| Integer   | Exact match                          |
| Float     | Configurable epsilon (default: 1e-6) |
| String    | Exact after NFC normalization        |
| Timestamp | Configurable precision (default: ms) |
| NULL      | NULL equals NULL                     |
| BLOB/CLOB | SHA-256 hash comparison              |

**Semantic.** For aggregates and computed results, verifies business meaning is preserved.

Each comparison produces a finding: PASS, INFO, WARNING, or FAILURE.

### Phase 4: Certify

Produces a signed certificate:

```json
{
  "certificate_id": "cert_a1b2c3d4",
  "protocol_version": "1.0",
  "migration_id": "mig_abc123def456",
  "queries_replayed": 12847,
  "findings": { "pass": 12831, "info": 12, "warning": 4, "failure": 0 },
  "behavioral_equivalence": true,
  "hash_chain": "sha256:abc123...",
  "signature": "ed25519:def456..."
}
```

The `hash_chain` is a Merkle root over all artifacts. Certificates are Ed25519-signed for SOX, HIPAA, and GDPR audit packages.

---

## Versioning

- **Patch** (1.0.x): Bug fixes. Backward compatible.
- **Minor** (1.x.0): New capabilities. Backward compatible.
- **Major** (x.0.0): Breaking changes. Previous versions verified for 36 months.

---

## Implementation

- Protocol: `packages/kora-core/src/protocol/`
- Canonical serialization: `packages/kora-core/src/protocol/canonical.rs`
- Signing: `ed25519-dalek` crate
- Proto definitions: `packages/shared/proto/verification.proto`

## References

- RFC-0001: Agent Architecture
- ADR-0002: Language Choices
- ADR-0003: Database Selection
