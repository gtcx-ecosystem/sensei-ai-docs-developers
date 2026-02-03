---
description: Replaying states against the target
---

# Historical Replay

## Replaying the Past to Verify the Present

Historical Replay is the execution phase of [Time-Travel Testing](../../../product/platform/kora/time-travel-testing.md). After reconstructing historical states in the target environment, KORA replays sampled queries against both source checkpoints and target reconstructions, comparing outputs to verify behavioral equivalence.

---

### State Reconstruction

Before replay can begin, KORA must reconstruct the source system's historical states in the target environment:

**Base state:** The migrated data as it exists in the target.

**Reverse transformation:** For each checkpoint, KORA applies reverse transformations to recover the pre-migration representation:

- **Stateless transformations** (type conversions, encoding changes, format normalization): Reversed algebraically. `TIMESTAMPTZ → DATE` is reversed by truncating the time component.
- **Stateful transformations** (aggregation, deduplication, normalization): Reversed using transformation logs that MABA maintains during migration. These logs record exactly which source records produced each target record.

**Isolation:** Reconstructed states are created in temporary schemas (`_kora_checkpoint_1`, `_kora_checkpoint_2`, etc.) within the target database. No interference with production data.

---

### The Replay Process

For each checkpoint:

1. **Restore source checkpoint data** to a sandboxed source environment (or use the original if still available)
2. **Build target checkpoint reconstruction** in a temporary schema
3. **Execute each sampled query** against both environments
4. **Capture outputs** with full precision
5. **Compare** at the configured fidelity level

Queries are replayed in the original execution order within each checkpoint, preserving any ordering dependencies.

---

### Three Fidelity Levels

#### Exact Match

Byte-identical output. Source says `42` and target says `42`. Source says `"John Smith"` and target says `"John Smith"`.

**Used for:** Integer operations, string lookups, count queries, boolean results.

#### Semantic Match

Equivalent within configurable tolerances:

- **Numeric:** `1247839.47` matches `1247839.48` if epsilon is `0.01`
- **String:** `"JOHN SMITH"` matches `"John Smith"` after case normalization
- **Temporal:** `2024-01-15T00:00:00Z` matches `2024-01-15T00:00:00+00:00` (same instant, different representation)
- **Null handling:** `NULL` matches `""` (empty string) if configured to treat them as equivalent

**Used for:** Floating-point calculations, timezone conversions, encoding transformations.

#### Business Match

Same business outcome, even if intermediate values differ:

- **Financial:** `$1,247,839.47` and `$1,247,839.48` produce the same rounding bucket and the same business decision
- **Threshold:** A credit score of `721` and `722` both fall in the "good" tier
- **Boolean:** Both results lead to the same approve/deny decision

**Used for:** Migration-induced precision changes where the business impact is zero.

---

### Divergence Classification

When a replayed query produces different output, KORA generates a detailed divergence report and classifies it:

**Migration Defect (Action Required)**
The divergence represents a genuine error that must be corrected:

- A SUM query returns a significantly different total (beyond tolerance)
- A JOIN query returns different record counts
- A stored procedure produces a different business decision

**Acceptable Variance (Document and Proceed)**
The divergence is expected and acceptable:

- Floating-point accumulation differences within financial rounding tolerance
- Timezone representation differences (same instant, different format)
- Whitespace or casing differences in string comparisons

**Source Change (Not a Migration Error)**
The source system changed between snapshot and replay:

- New records inserted after the checkpoint
- Records modified by ongoing business operations
- Schema changes applied after capture

---

### Parallel Replay

KORA parallelizes replay across checkpoints and within checkpoints:

- **Cross-checkpoint:** Each checkpoint is replayed independently in parallel
- **Within-checkpoint:** Read-only queries are replayed in parallel; write transactions are serialized to preserve ordering

With 7 checkpoints and 10,000 queries per checkpoint:

- Serial execution: ~14 hours (at 5,000 replays/hour)
- Parallel across checkpoints: ~2 hours
- With within-checkpoint parallelism: ~45 minutes

---

### Performance

| Metric        | Value                                                          |
| ------------- | -------------------------------------------------------------- |
| Throughput    | 5,000 replays/hour per checkpoint                              |
| Parallelism   | Up to N checkpoints simultaneously                             |
| Resource cost | Very high (requires full source and target database instances) |
| Accuracy      | 97-99% (limited by query sample representativeness)            |

Historical Replay is the most expensive validation step — it requires computational resources proportional to the source system's workload. The cost is justified by the unique guarantee it provides: behavioral equivalence, not just data equivalence.

→ [Behavioral Equivalence Certification](behavioral-equivalence-certification.md)
→ [Time-Travel Testing Overview](../../../product/platform/kora/time-travel-testing.md)
