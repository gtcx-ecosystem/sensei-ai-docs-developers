# Validator Agents

## Trust, But Verify — Continuously

Validator Agents ensure that every record migrated by [Worker Agents](worker-agents.md) meets quality, integrity, and compliance requirements. They don't wait until the migration is complete — they validate continuously, catching errors in seconds rather than days.

---

### Role in the Hierarchy

Validator Agents are the second Execution Agent type. They use fine-tuned BERT-class models for fast semantic classification, combined with rule engines for deterministic checks.

```text
Scout Agents → Architect Agents
  ↓ Executable Migration Plan
Worker Agents → Validator Agents (YOU ARE HERE)
  ↓ Execution Logs + Validation Results
Meta Agents (Learning)
```

Validators serve as the quality gate between transformation and target system insertion. Nothing reaches the target without passing validation.

---

### Validation Perspectives

Validators apply five independent validation perspectives. Each catches errors that the others miss. Together, they provide comprehensive coverage.

#### 1. Structural Validation

**Method:** Deterministic comparison
**Checks:** Schema equivalence between source and target — same tables, columns, constraints, indexes, and relationships were created correctly.
**Accuracy:** 100% (deterministic)
**Speed:** 50,000 tables/hour

Does the target schema faithfully represent the source schema after planned transformations? Are all constraints, indexes, and relationships correctly established?

#### 2. Statistical Validation

**Method:** Distributional comparison
**Checks:** Column-level statistical profiles (mean, median, min, max, null rate, distinct count, distribution shape) match between source and target within configurable tolerances.
**Accuracy:** 99.9%
**Speed:** 100,000 columns/hour

If the source column has a mean of 47.3 and null rate of 2.1%, the target should match within epsilon. Statistical drift beyond tolerance indicates systematic transformation errors.

#### 3. Referential Validation

**Method:** Foreign key traversal
**Checks:** Every foreign key reference in the target resolves to an existing parent record. No orphan records. Cascade behaviors (UPDATE, DELETE) produce correct results.
**Accuracy:** 100% (deterministic)
**Speed:** 80,000 relationships/hour

This catches one of the most common migration defects: records that reference parent rows that either weren't migrated or were migrated with different key values.

#### 4. Semantic Validation

**Method:** AI-driven inference using fine-tuned BERT
**Checks:** Business rules that aren't encoded in schema constraints. Values that are syntactically valid but semantically wrong. Patterns that violate inferred business logic.
**Accuracy:** 94-98%
**Speed:** 10,000 rules/hour

A date of birth of `2025-01-15` is structurally valid but semantically wrong for a customer record created in 2020. A salary of `$0.01` passes type checking but fails semantic plausibility. Semantic validation catches what rule engines can't.

#### 5. Behavioral Validation (KORA Time-Travel Testing)

**Method:** Historical replay — see [KORA: Time-Travel Testing](../../product/platform/kora/time-travel-testing.md)
**Checks:** Replaying historical queries and transactions against the migrated system produces identical business outcomes as the source system.
**Accuracy:** 97-99%
**Speed:** 5,000 replays/hour

This is the most expensive and most powerful validation. It answers the question no other validation can: "Will the business get the same answers from the new system as from the old one?"

---

### Continuous Validation Pipeline

Validators don't run as a post-migration batch job. They operate inline with the migration flow:

```text
Source Records
    ↓
Worker Agent (transform)
    ↓
Validator Agent
    ├── PASS → Write to Target
    └── FAIL → Error Queue
                  ↓
              Worker Agent (retry with fix)
                  ↓
              Validator Agent (re-check)
                  ├── PASS → Write to Target
                  └── FAIL → Escalate to Cognitive Agent
```

This continuous pipeline means:

- Errors are detected within seconds of the transformation that caused them
- Error patterns are identified early, before millions of records are affected
- The migration can be paused immediately if error rates exceed thresholds
- Validation results are available in real-time on AMANI dashboards

---

### Validation Pattern Library

Validators maintain a growing library of validation patterns — rules that have proven effective across migrations:

- **Universal patterns:** Type range checks, null rate thresholds, referential integrity
- **Domain-specific patterns:** Financial precision rules, healthcare PHI validation, date range plausibility
- **Migration-specific patterns:** Rules inferred from the Scout's Discovery Report for this specific migration

The library grows through two mechanisms:

1. **Rule confirmation:** When a validation rule catches a real error (confirmed by resolution), it's marked as effective and its confidence score increases
2. **Rule generation:** When semantic validation discovers a new business rule violation, the rule is formalized and added to the library for future migrations

Rules that produce false positives are downweighted. Rules that never fire are periodically reviewed for relevance. The library self-curates toward high-signal, low-noise validation.

---

### Compliance Output

For regulated migrations, Validators produce structured compliance documentation:

- **Validation certificates:** Per-table attestation of validation pass/fail with evidence
- **Coverage reports:** Percentage of records, columns, and business logic paths validated
- **Exception reports:** Every validation failure with resolution status and audit trail
- **Regulatory mapping:** Which validation checks satisfy which regulatory requirements (GDPR Article 32, HIPAA §164.312, SOX Section 404)

These outputs feed into [KORA's Behavioral Equivalence Certificates](../components/kora/behavioral-equivalence-certification.md) — cryptographically signed documents suitable for regulatory audit submission.

→ [Worker Agents](worker-agents.md)
→ [KORA: Multi-Source Validation](../components/kora/multi-source-validation.md)
→ [Execution Agents Overview](three-layer-architecture/execution-agents.md)
