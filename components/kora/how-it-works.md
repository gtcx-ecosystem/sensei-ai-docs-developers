---
description: KORA verification engine internals
---

# How KORA Works

## Source to Verification Certificate

This page walks through KORA's complete verification process — from pre-migration preparation through post-migration certification.

> For platform overview and business context, see [KORA Overview](../../../product/platform/kora/README.md).

---

### The Verification Timeline

```text
BEFORE MIGRATION
┌─────────────────────────────────┐
│ 1. TEMPORAL STATE CAPTURE       │
│    Snapshot source at multiple  │
│    checkpoints. Record queries, │
│    transactions, business logic │
└─────────────────────────────────┘
                ↓
DURING MIGRATION
┌─────────────────────────────────┐
│ 2. INLINE VALIDATION            │
│    Continuous structural,       │
│    statistical, referential,    │
│    and semantic checks as data  │
│    flows through the pipeline   │
└─────────────────────────────────┘
                ↓
AFTER MIGRATION
┌─────────────────────────────────┐
│ 3. TIME-TRAVEL TESTING          │
│    Reconstruct historical       │
│    states in target. Replay     │
│    queries. Compare outputs.    │
├─────────────────────────────────┤
│ 4. CERTIFICATION                │
│    Generate cryptographically   │
│    signed Behavioral            │
│    Equivalence Certificate      │
└─────────────────────────────────┘
```

---

### Phase 1: Before Migration — Temporal State Capture

Before any data moves, KORA instruments the source system:

**Database snapshots:** Full or incremental snapshots at defined intervals (hourly for high-churn systems, daily/weekly for stable systems). Each snapshot is cryptographically hashed to prevent tampering.

**Query log sampling:** KORA samples representative queries and transactions from the system's query log. These become the test suite for behavioral verification later.

**Business logic inventory:** Stored procedure definitions, trigger logic, view definitions, and computed column formulas are captured and versioned.

**Checkpoint chain:** Each snapshot links to the previous via cryptographic hash, creating a verifiable timeline of source system state evolution.

→ [Temporal State Capture Details](temporal-state-capture.md)

---

### Phase 2: During Migration — Inline Validation

As MABA transforms and loads data, KORA's [Validator Agents](../../architecture/validator-agents.md) run continuous checks:

**Structural validation:** Does the target schema match the planned structure? Are constraints, indexes, and relationships correctly created?

**Statistical validation:** Do column-level statistics (mean, null rate, distinct count, distribution shape) match between source and target within tolerance?

**Referential validation:** Does every foreign key reference resolve? Are there orphan records?

**Semantic validation:** Do values pass AI-driven business rule checks? Are there syntactically valid but semantically implausible values?

Errors caught during inline validation trigger immediate response — the Worker Agent receives the error, applies a fix (from the pattern library or generated on-the-fly), and resubmits the record for validation. This feedback loop runs in seconds, not days.

→ [Multi-Source Validation Details](multi-source-validation.md)

---

### Phase 3: After Migration — Time-Travel Testing

This is KORA's patent-pending innovation. After migration completes:

**Step 1: Reconstruct historical states**
Using the migrated data as a base, KORA applies reverse transformations to recreate the source system's state at each pre-migration checkpoint. These reconstructed states are isolated in temporary schemas.

**Step 2: Replay historical queries**
The sampled queries and transactions from Phase 1 are replayed against both the original source checkpoint data and the reconstructed target state.

**Step 3: Compare outputs**
Output comparison operates at three fidelity levels:

- **Exact match:** Byte-identical output
- **Semantic match:** Numerically equivalent within epsilon, string equivalent after normalization
- **Business match:** Produces the same business decision even if intermediate values differ

Any divergence generates a detailed report including affected records, divergence magnitude, and AI-driven analysis of whether the divergence represents a defect or an acceptable transformation artifact.

→ [Time-Travel Testing Details](../../../product/platform/kora/time-travel-testing.md)

---

### Phase 4: Certification

When all validation checks pass, KORA generates a **Behavioral Equivalence Certificate** — a cryptographically signed document attesting to migration correctness.

The certificate includes:

- List of temporal checkpoints validated
- Queries and transactions replayed with results
- Comparison outcomes at each fidelity level
- Hash chain linking source checkpoints to target reconstructions
- Confidence score based on validation coverage
- Digital signature preventing post-issuance modification

This certificate is suitable for regulatory audit submission under GDPR, HIPAA, SOX, Basel III, and similar frameworks.

→ [Behavioral Equivalence Certification Details](behavioral-equivalence-certification.md)

---

### Example: Real Verification Walkthrough

**Migration:** Oracle 19c (247 tables, 52M rows) → PostgreSQL 16

1. **Pre-migration** (Day -7 to Day 0): KORA captures 7 daily snapshots, samples 12,000 queries from Oracle audit log, records 89 stored procedure definitions
2. **Inline validation** (18-24 hours): 52M records validated as they migrate. 847 errors caught and auto-corrected. 3 errors escalated to human review.
3. **Time-Travel Testing** (6 hours): 7 checkpoint states reconstructed in temporary PostgreSQL schemas. 12,000 queries replayed. 11,943 exact matches. 52 semantic matches (floating-point precision differences). 5 divergences flagged for review.
4. **Certification** (10 minutes): Certificate generated with 99.96% behavioral equivalence score. 5 flagged divergences documented with AI analysis (all determined to be acceptable precision changes).

**Total verification time:** ~7 hours post-migration
**Traditional equivalent:** 2-4 weeks of manual UAT testing with ~15-30% residual undetected issues
