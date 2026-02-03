---
description: Competitive strategy evolution
---

# Evolutionary Tournament Selection

## Self-Evolving Migration Strategies

Tournament selection is the third and slowest of Sensei's learning loops. It operates over weeks and months, evolving migration strategies through genetic algorithm principles — producing optimization approaches that emerge from empirical selection pressure rather than human design.

---

### How It Works

The Meta Agent layer maintains a **strategy pool** — a population of migration strategy configurations that compete, reproduce, and evolve.

Each strategy encodes decisions about:

- **Parallelization:** How many tables to migrate concurrently, batch sizes, chunk sizes
- **Ordering:** Sequence of table migration (by dependency depth, by size, by complexity)
- **Error handling:** Retry thresholds, escalation triggers, fallback approaches
- **Resource allocation:** Worker count, memory allocation, compression settings
- **Checkpointing:** Snapshot frequency, rollback point granularity
- **Validation:** When to run validation (continuous, periodic, post-migration), fidelity level

---

### The Evolution Cycle

#### 1. Evaluation

After each migration, every strategy that was active during the migration is scored on a multi-objective fitness function:

```text
Fitness = w₁(speed_score) + w₂(accuracy_score) + w₃(efficiency_score) + w₄(autonomy_score)

Where:
  speed_score     = records_per_second / expected_records_per_second
  accuracy_score  = 1 - (errors / total_records)
  efficiency_score = 1 - (actual_cost / budget_cost)
  autonomy_score  = 1 - (human_interventions / total_decisions)

  w₁, w₂, w₃, w₄ = configurable weights (default: 0.25 each)
```

Different customers can weight the fitness function differently. A financial institution might set `w₂ = 0.5` (accuracy dominates). A startup migrating quickly might set `w₁ = 0.5` (speed dominates).

#### 2. Selection

The top-performing 20% of strategies survive to the next generation. The bottom 80% are discarded. This aggressive selection pressure ensures rapid convergence toward effective strategies.

Strategies are evaluated within **migration profile clusters** — a strategy that works brilliantly for Oracle-to-PostgreSQL isn't compared against strategies optimized for MongoDB-to-Snowflake. Evolution happens within similar migration profiles, allowing specialization.

#### 3. Crossover

Surviving strategies are combined to produce offspring:

```text
Parent A: {batch_size: 50000, parallel_tables: 4, checkpoint_every: 100000}
Parent B: {batch_size: 10000, parallel_tables: 16, checkpoint_every: 50000}

Offspring C: {batch_size: 50000, parallel_tables: 16, checkpoint_every: 50000}
Offspring D: {batch_size: 10000, parallel_tables: 4, checkpoint_every: 100000}
```

Crossover combines proven elements from multiple successful strategies. Parent A's large batch size paired with Parent B's high parallelism might produce a strategy that neither parent configuration would have discovered independently.

#### 4. Mutation

Random perturbations are applied to offspring:

```text
Offspring C (pre-mutation):  {batch_size: 50000, parallel_tables: 16}
Offspring C (post-mutation): {batch_size: 47500, parallel_tables: 16}
                              ↑ random ±10% perturbation
```

Most mutations are neutral or harmful. But occasionally, a mutation discovers an improvement — a batch size sweet spot that aligns with the destination database's write buffer, or a checkpoint interval that minimizes I/O contention. These improvements are preserved by selection pressure in the next generation.

#### 5. Integration

After each evolution cycle, the best-performing strategy for each migration profile becomes the new default. When a new migration begins, the system recommends the highest-fitness strategy for that profile.

The full pool remains available — Cognitive Agents can override the default if their analysis suggests an alternative strategy would perform better for a specific migration's characteristics.

---

### Emergent Strategies

Over hundreds of evolution cycles, strategies develop approaches that human engineers would not have designed:

**Pre-sorting by foreign key depth.** A strategy evolved to migrate tables in reverse topological order (leaf tables first, root tables last), reducing referential integrity violations by 94%. No engineer programmed this — it emerged because strategies that happened to sequence tables this way had higher accuracy scores and were selected more frequently.

**Dynamic partition splitting.** A strategy evolved to detect natural partition boundaries in large tables (by analyzing value distribution gaps) and split migration into parallel streams at those boundaries. This achieved 3x throughput improvement because it aligned chunk boundaries with data locality, reducing cross-partition reads.

**Concurrent validation.** A strategy evolved to run KORA validation in parallel with migration rather than sequentially after completion. This reduced total wall-clock time by 40% with no accuracy loss — the strategy discovered that validation of completed tables can overlap with migration of pending tables.

**Adaptive compression.** A strategy evolved different compression settings for different data types within the same migration: high compression for text-heavy columns (large payload, high redundancy), no compression for already-compact numeric columns (small payload, low redundancy, decompression overhead not worth it).

---

### Convergence and Diversity

A pure selection process would converge to a single "best" strategy and lose diversity. This is dangerous — a strategy optimized for the most common migration profile would perform poorly on unusual profiles.

Sensei maintains diversity through:

- **Profile-specific evolution:** Separate strategy pools for different migration profiles (relational-to-relational, document-to-relational, mainframe-to-cloud, etc.)
- **Diversity bonus:** Strategies that are structurally different from the current pool receive a fitness bonus, preventing premature convergence
- **Immigration:** Periodically, random new strategies are injected into the pool, introducing fresh genetic material
- **Hall of fame:** The best strategy ever seen for each profile is preserved permanently, immune to selection pressure

---

### Strategy Pool Size

| Stage   | Pool Size        | Evolution Cycles | Profile Clusters |
| ------- | ---------------- | ---------------- | ---------------- |
| Month 1 | 20 strategies    | ~5 cycles        | 3 clusters       |
| Month 6 | 100 strategies   | ~50 cycles       | 10 clusters      |
| Year 1  | 500 strategies   | ~200 cycles      | 25 clusters      |
| Year 3  | 2,000 strategies | ~1,000 cycles    | 50+ clusters     |

The pool grows because new migration profiles create new clusters, and each cluster maintains its own evolving population.

→ [Compound Intelligence](../../product/why-sensei/overview/compound-intelligence.md)
→ [Meta Agents](three-layer-architecture/meta-agents.md)
