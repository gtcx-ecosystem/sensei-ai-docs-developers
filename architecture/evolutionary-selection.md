---
description: Genetic optimization of strategies
---

# Evolutionary Selection

## Automated Strategy Optimization

Evolutionary Selection is the second Meta Agent function. While [Learning Agents](learning-agents.md) extract knowledge from individual migrations, Evolutionary Selection optimizes how that knowledge is applied — evolving migration strategies that improve with each generation.

---

### Role in the Hierarchy

Evolutionary Selection operates alongside Learning Agents in the Meta tier, maintaining and evolving the strategy pool that [Architect Agents](architect-agents.md) draw from when planning new migrations.

```text
Learning Agents → Pattern Library (what we know)
Evolutionary Selection (YOU ARE HERE) → Strategy Pool (how we apply it)
  ↓
Architect Agents (select best strategy for each migration)
```

---

### What Is a Strategy?

A migration strategy is a configuration vector that controls how a migration executes. It encodes dozens of parameters:

| Category           | Parameters                                            | Examples                                                       |
| ------------------ | ----------------------------------------------------- | -------------------------------------------------------------- |
| **Parallelism**    | Table concurrency, worker count, batch pipeline depth | `parallel_tables: 8, workers: 32, pipeline_depth: 3`           |
| **Batching**       | Batch size, chunk size, buffer size                   | `batch_size: 50000, chunk_size: 5000`                          |
| **Ordering**       | Table sequence algorithm, priority rules              | `order_by: reverse_fk_depth, prioritize: reference_tables`     |
| **Error handling** | Retry count, backoff, escalation threshold            | `max_retries: 3, backoff: exponential, escalate_after: 5min`   |
| **Checkpointing**  | Frequency, granularity, storage                       | `checkpoint_every: 100000, granularity: table_level`           |
| **Validation**     | Timing, fidelity, sampling rate                       | `validation: continuous, fidelity: semantic, sample_rate: 1.0` |
| **Resources**      | Compression, memory allocation, GPU usage             | `compression: lz4, memory_per_worker: 4GB`                     |

A single strategy might have 30-50 parameters. The search space is enormous — far too large for humans to optimize by hand, but well-suited for evolutionary search.

---

### The Evolutionary Process

See [Tournament Selection](tournament-selection.md) for the detailed algorithm. In summary:

1. **Evaluate** each strategy's fitness after migrations complete (speed × accuracy × efficiency × autonomy)
2. **Select** top 20% as survivors
3. **Crossover** survivor pairs to produce offspring with mixed parameters
4. **Mutate** offspring with random perturbations
5. **Integrate** the best strategy as the new default for its migration profile

The cycle repeats with every completed migration, creating continuous selection pressure toward better strategies.

---

### Migration Profile Clusters

Strategies evolve within clusters of similar migration profiles. A strategy that excels at Oracle-to-PostgreSQL migrations isn't evaluated against strategies optimized for MongoDB-to-Snowflake.

**Clustering dimensions:**

- Source database engine family (relational, document, mainframe, file-based)
- Target database engine family
- Data volume category (small, medium, large, very large)
- Complexity category (low, medium, high, very high)
- Domain category (HR, financial, healthcare, e-commerce, government)

As the platform processes more migrations, clusters subdivide for finer specialization. Early on, "relational-to-relational" is a single cluster. After 1,000 migrations, it may split into "Oracle-to-PostgreSQL," "SQL Server-to-Snowflake," "MySQL-to-Aurora," each with independently evolving strategy pools.

---

### Emergent vs. Designed Strategies

The most interesting outcome of evolutionary selection is that it discovers strategies no human would have designed:

| Emergent Strategy           | How It Emerged                                                                                  | Performance Gain         |
| --------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------ |
| Pre-sort by FK depth        | Strategies that accidentally ordered tables by dependency depth had fewer constraint violations | 94% fewer violations     |
| Natural partition splitting | Strategies that split large tables at value distribution gaps had better parallelism            | 3x throughput            |
| Concurrent validation       | Strategies that overlapped validation with migration had shorter total duration                 | 40% less wall-clock time |
| Adaptive compression        | Strategies with per-column compression settings outperformed uniform settings                   | 25% less network I/O     |

These discoveries are the compound intelligence in action — the system finds optimizations through empirical exploration that would require deep domain expertise and extensive experimentation for a human to discover.

---

### Strategy Diversity Maintenance

Pure selection pressure leads to convergence — all strategies become similar, losing the diversity needed to handle unusual migrations. The system prevents this through:

- **Diversity bonus:** Strategies structurally different from the current pool receive a fitness bonus
- **Immigration:** Random new strategies are periodically injected
- **Hall of fame:** The best strategy ever seen for each profile is preserved permanently
- **Novelty search:** Strategies that explore new regions of the parameter space are rewarded even if their fitness is average

This balances exploitation (optimizing known-good strategies) with exploration (discovering new approaches).

→ [Tournament Selection (detailed algorithm)](tournament-selection.md)
→ [Learning Agents](learning-agents.md)
→ [Meta Agents Overview](three-layer-architecture/meta-agents.md)
