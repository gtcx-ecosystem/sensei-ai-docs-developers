# L2: Execution Agents

## The Doers — Executing, Recovering, Validating, Scaling

Execution Agents implement the plans produced by Cognitive Agents. They're optimized for throughput and resilience, using smaller models that can process millions of records while autonomously handling errors, optimizing resources, and validating results in real-time.

---

### Model Requirements

Execution tasks prioritize speed over reasoning depth:

| Capability                     | Why Smaller Models                                                            |
| ------------------------------ | ----------------------------------------------------------------------------- |
| Known transformation execution | Applying a validated mapping doesn't require frontier reasoning               |
| Error classification           | Categorizing known error patterns is a classification task                    |
| Constraint checking            | Validation rules are expressible as formal specifications                     |
| Resource monitoring            | Performance optimization uses numerical telemetry, not language understanding |

**Primary models:** Llama-3-8B or Mistral-class via vLLM (fast, cost-effective) **Validation models:** Fine-tuned BERT-class for semantic classification and anomaly detection **Orchestration:** Ray 2.0 for distributed execution, auto-scaling from 4 to 128 workers **Communication:** Redis pub/sub for real-time swarm coordination

---

### Worker Agents

Worker Agents execute migration tasks — the actual data movement and transformation that constitutes the migration.

#### Autonomous Error Recovery

When a Worker Agent encounters an error, it doesn't halt the pipeline and wait for human intervention. It initiates an autonomous recovery sequence:

1. **Classify the error** against the known error pattern library
2. **Retrieve solutions** for similar errors from the pattern database
3. **Generate candidate fixes** if no exact match exists (using the local LLM)
4. **Test each fix** against sample data in an isolated sandbox
5. **Apply the successful fix** and continue processing
6. **Broadcast the solution** to all peer agents via Redis pub/sub

The first agent to solve an error ensures no subsequent agent encounters the same problem. For a migration processing 50 million records across hundreds of tables, this swarm intelligence dramatically reduces total error-handling time.

Recovery success rates improve with system maturity:

- Month 1: \~60% of errors resolved autonomously
- Month 6: \~85% (pattern library has grown from operational experience)
- Year 1: \~95% (most error patterns have been seen and resolved before)
- Year 3: \~99% (edge cases are increasingly rare)

#### Dynamic Resource Optimization

Worker Agents monitor their own resource consumption and collaborate with the Performance Optimizer to maintain throughput targets:

- **CPU-bound tasks** trigger spawning of additional Worker Agents via Ray's elastic scaling
- **I/O-bound tasks** trigger batch size optimization and write buffering
- **Network-bound tasks** trigger compression adjustment and chunk size reduction
- **Memory pressure** triggers garbage collection coordination and work redistribution

#### Self-Organizing Work Distribution

Worker Agents don't receive centrally assigned task lists. They pull tasks from a shared work queue, self-organizing based on current load, specialization, and proximity to data. Agents that have recently processed similar data types develop warm caches and preferentially pull related tasks, creating organic specialization without explicit assignment.

---

### Validator Agents

Validator Agents run continuously alongside Worker Agents, checking data quality and compliance as records are processed — not after the migration is complete.

#### Validation Types

**Structural validation.** Schema comparison, constraint verification, index equivalence. Deterministic — 100% accuracy.

**Statistical validation.** Distributional comparison of column-level statistics between source and target. Detects systematic shifts that record-level comparison misses (e.g., the average order value shifted by 3% due to a rounding transformation).

**Referential validation.** Foreign key integrity, cascade behavior, orphan record detection. Catches relationship breaks that structural validation alone doesn't cover.

**Semantic validation.** AI-driven inference of implicit business rules, using fine-tuned BERT models trained on validated rule patterns. Detects values that are syntactically valid but semantically implausible (e.g., a birth date in the future, a negative account balance in a field that should be non-negative).

**Behavioral validation.** The patent-pending Time-Travel Testing method (detailed in [KORA documentation](../../../product/platform/kora/time-travel-testing.md)). Replays historical queries and transactions against both source and target states to verify functional equivalence.

#### Continuous Validation

Unlike traditional migration validation that runs after completion, Validator Agents operate as a continuous quality gate:

```text
Records flow: Source → Transform → Validate → Target
                                    ↓ (failures)
                              Error Queue → Worker Agent retry → Re-validate
```

Records that fail validation don't proceed to the target. They're routed to an error queue where Worker Agents apply fixes and resubmit. This ensures the target database only ever contains validated data — there's no "fix it later" phase.

#### Validation Pattern Library

Validator Agents maintain a growing library of validation patterns learned from prior migrations. When a new validation rule proves effective at catching real errors (confirmed by post-migration analysis), it's added to the library and automatically applied to future migrations with similar profiles.

→ [Worker Agent specification](../worker-agents.md) → [Validator Agent specification](../validator-agents.md) → [Swarm Learning](../swarm-learning.md)
