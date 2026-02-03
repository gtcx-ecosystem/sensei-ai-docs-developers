---
description: Learning, optimizing, and evolving
---

# L3: Meta Agents

## Learning, Optimizing, Evolving

Meta Agents operate on the longest timescale. While Cognitive Agents think in minutes and Execution Agents act in milliseconds, Meta Agents learn over weeks and months — analyzing completed migrations to extract patterns, evolve strategies, and build the compound intelligence that makes Sensei increasingly powerful over time.

---

### Model Requirements

Meta Agent tasks combine language understanding with optimization:

| Capability                       | Model / Framework                                      |
| -------------------------------- | ------------------------------------------------------ |
| Pattern extraction and analysis  | GPT-4 for semantic analysis of migration logs          |
| Strategy optimization            | Ray RLlib for reinforcement learning                   |
| Cross-customer insight synthesis | Privacy-preserving federated learning pipeline         |
| Performance prediction           | Gradient-boosted models trained on migration telemetry |
| Knowledge graph construction     | Neo4j for relationship encoding                        |

**Experiment tracking:** MLflow + Weights & Biases **Knowledge store:** Neo4j graph database for pattern relationships

---

### Learning Agents

Learning Agents are responsible for the compound intelligence that accumulates across migrations.

#### Pattern Extraction

After each migration completes, Learning Agents analyze the full execution log to extract transferable patterns:

- **Schema patterns:** Structural fingerprints that characterize source-target pairs (type distributions, cardinality ratios, relationship densities)
- **Transformation patterns:** Which transformations were needed and why (type conversions, encoding fixes, structural reshaping)
- **Error patterns:** What errors occurred, how they were resolved, and what conditions triggered them
- **Performance patterns:** What throughput was achieved, where bottlenecks occurred, and which optimizations were effective

Each pattern is encoded as a high-dimensional vector and stored in the persistent vector database. Relationships between patterns are encoded in the Neo4j graph — creating a knowledge graph that enables reasoning about migration similarity at a structural level, not just vector proximity.

#### Cross-Customer Intelligence

Learning Agents synthesize insights across customers while preserving privacy:

1. **Abstract pattern extraction** — Customer-specific details (table names, column names, data values) are stripped. Only structural properties and statistical profiles are retained.
2. **Differential privacy** — Calibrated Gaussian noise is added to pattern vectors before storage, ensuring no individual customer's patterns can be reconstructed (ε = 1.0 default).
3. **Pattern aggregation** — Similar patterns from multiple customers are clustered and merged into generalized patterns with higher confidence scores.
4. **Insight generation** — Learning Agents identify trends across the migration corpus. For example: "Migrations from Oracle to PostgreSQL involving date-type columns fail at a 3x higher rate when the source uses Oracle-specific date functions. Pre-converting to standard ISO formats reduces error rate by 78%."

These insights are surfaced to Cognitive Agents as planning priors for new migrations.

---

### Evolutionary Strategy Selection

The Meta Agent layer maintains a pool of migration strategies that evolve over time using genetic algorithm principles.

#### The Evolution Cycle

1. **Evaluation.** After each migration, every strategy in the pool is scored on a multi-objective fitness function:
   - Migration speed (wall-clock time relative to data volume)
   - Error rate (errors per million records)
   - Resource efficiency (compute cost per record)
   - Human intervention rate (escalations per migration)
2. **Selection.** The top-performing 20% of strategies survive to the next generation.
3. **Crossover.** Surviving strategies are combined: Strategy A's parallelization approach paired with Strategy B's error handling produces offspring Strategy C.
4. **Mutation.** Random perturbations are applied: slightly larger batch sizes, different compression settings, altered checkpoint intervals. Most mutations are neutral or harmful, but occasionally one discovers an improvement that wasn't explicitly designed.
5. **Integration.** The best-performing strategy from the evolved pool becomes the default for new migrations matching that profile. Other high-performing strategies remain in the pool as alternatives.

#### Emergent Capabilities

Over hundreds of evolution cycles, strategies develop approaches that human engineers wouldn't have designed:

- A strategy that pre-sorts records by foreign key depth before migration, reducing referential integrity violations by 94%
- A strategy that dynamically splits large tables at natural partition boundaries (detected by value distribution analysis), achieving 3x throughput improvement
- A strategy that runs validation concurrently with migration rather than sequentially, reducing total wall-clock time by 40% with no accuracy loss

These emerged from evolutionary pressure, not from explicit programming. The strategies are discovered empirically by the system through trial, measurement, and selection.

---

### Performance Prediction

Meta Agents build predictive models that estimate migration duration, error rates, and resource requirements for new migrations based on similarity to completed migrations.

When a new migration request arrives, the system:

1. Extracts the source-target profile (schema structure, data volume, transformation complexity)
2. Finds the 10 most similar completed migrations in the pattern library
3. Generates performance estimates with confidence intervals based on the variance observed across those similar migrations
4. Recommends resource allocation (node count, GPU requirements, estimated duration)
5. Flags risk factors specific to this migration profile

Prediction accuracy improves with the pattern library's growth. After 100 completed migrations, duration estimates are typically within ±30%. After 1,000 migrations, within ±15%. After 10,000, within ±5%.

---

### The Knowledge Graph

Meta Agents maintain a Neo4j graph database that encodes relationships between all platform knowledge assets:

```text
(SchemaPattern) -[:SIMILAR_TO]-> (SchemaPattern)
(SchemaPattern) -[:REQUIRES]-> (TransformationPattern)
(TransformationPattern) -[:MAY_CAUSE]-> (ErrorPattern)
(ErrorPattern) -[:RESOLVED_BY]-> (SolutionPattern)
(SolutionPattern) -[:EFFECTIVE_FOR]-> (MigrationProfile)
(MigrationProfile) -[:BEST_STRATEGY]-> (OptimizationStrategy)
(OptimizationStrategy) -[:EVOLVED_FROM]-> (OptimizationStrategy)
```

This graph enables reasoning about migration challenges at a structural level. When a Cognitive Agent plans a migration, it doesn't just find "similar" patterns by vector proximity — it can traverse the graph to understand: "This schema pattern typically requires these transformations, which may cause these errors, which are best resolved by these solutions, and the optimal strategy for this profile has evolved through these iterations."

The graph is the platform's institutional memory — the equivalent of the senior consultant's decades of experience, encoded as navigable, queryable, and continuously growing structured knowledge.

→ [Learning Agents specification](../learning-agents.md) → [Evolutionary Strategy Selection](../evolutionary-selection.md) → [Compound Intelligence](../../../product/why-sensei/overview/compound-intelligence.md)
