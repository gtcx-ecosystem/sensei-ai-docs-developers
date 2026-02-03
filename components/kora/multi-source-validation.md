# Multi-Source Validation

## Five Perspectives, One Verdict

KORA validates migration correctness from five independent perspectives. Each catches errors that the others miss. Together, they provide comprehensive coverage that no single validation method can achieve.

---

### The Principle

Migration correctness should be verified the way distributed consensus protocols establish truth — through independent agreement rather than single-source authority. If five independent checks all agree the migration is correct, confidence is high. If they disagree, the disagreement itself reveals exactly where to look for problems.

---

### The Five Perspectives

| Perspective                                                             | Method                                  | What It Catches                                                  | Speed                   |
| ----------------------------------------------------------------------- | --------------------------------------- | ---------------------------------------------------------------- | ----------------------- |
| **[Structural](structural-validation.md)**                              | Deterministic schema comparison         | Missing tables, wrong types, missing constraints, broken indexes | 50,000 tables/hr        |
| **[Statistical](statistical-validation.md)**                            | Distributional comparison               | Systematic transformation errors, data loss, silent truncation   | 100,000 columns/hr      |
| **[Referential](referential-validation.md)**                            | Foreign key traversal                   | Orphan records, broken relationships, cascade failures           | 80,000 relationships/hr |
| **[Semantic](semantic-validation.md)**                                  | AI-driven business rule inference       | Syntactically valid but semantically wrong values                | 10,000 rules/hr         |
| **[Behavioral](../../../product/platform/kora/time-travel-testing.md)** | Historical replay (Time-Travel Testing) | Logic loss, implicit rule failures, business outcome divergence  | 5,000 replays/hr        |

---

### How They Work Together

Each perspective produces an independent confidence score (0.0 to 1.0). KORA combines them using a weighted consensus model:

```yaml
Overall Confidence = Σ (weight_i × confidence_i)

Default weights:
  Structural:   0.15  (necessary but not sufficient)
  Statistical:  0.20  (catches systematic issues)
  Referential:  0.20  (catches relationship breaks)
  Semantic:     0.20  (catches business logic errors)
  Behavioral:   0.25  (the gold standard — actual outcomes)
```

Weights are configurable per migration. Regulated environments might increase behavioral weight to 0.40. Speed-critical migrations might reduce behavioral weight and increase statistical weight.

---

### Conflict Resolution

When perspectives disagree — for example, statistical validation passes but semantic validation flags an anomaly — KORA invokes a conflict resolution protocol:

1. **Identify** the specific records and fields where disagreement occurs
2. **Analyze** using targeted AI to classify the disagreement:
   - **Genuine error:** Migration defect requiring correction
   - **Data evolution:** Source system changed after migration snapshot (acceptable)
   - **False positive:** Validation rule is overly strict
   - **Unknown rule:** Previously unrecognized business logic to add to the validation library
3. **Record** the resolution with full provenance — every decision is traceable to its evidence

Conflict resolution outcomes feed back into the validation pattern library, improving future accuracy.

---

### Coverage Metrics

KORA tracks validation coverage — what percentage of the migration has been verified:

| Metric                    | Definition                                             | Target |
| ------------------------- | ------------------------------------------------------ | ------ |
| **Table coverage**        | % of tables with at least one validation pass          | 100%   |
| **Column coverage**       | % of columns profiled and validated                    | 100%   |
| **Record coverage**       | % of records passing at least one validation           | 100%   |
| **Logic path coverage**   | % of stored procedures exercised by behavioral testing | >80%   |
| **Relationship coverage** | % of foreign key relationships verified                | 100%   |

Coverage gaps are flagged in the Behavioral Equivalence Certificate as areas of reduced confidence.

→ [Structural Validation](structural-validation.md)
→ [Statistical Validation](statistical-validation.md)
→ [Referential Validation](referential-validation.md)
→ [Semantic Validation](semantic-validation.md)
→ [Behavioral Validation (Time-Travel Testing)](../../../product/platform/kora/time-travel-testing.md)
