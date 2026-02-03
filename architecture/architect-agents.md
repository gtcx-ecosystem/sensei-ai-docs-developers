# Architect Agents

## From Understanding to Executable Plan

Architect Agents consume the Discovery Reports produced by [Scout Agents](scout-agents.md) and transform them into executable migration plans — complete with DAGs, transformation code, performance estimates, and rollback strategies.

---

### Role in the Hierarchy

Architect Agents are the second Cognitive Agent type. Where Scouts answer "what exists?", Architects answer "how do we move it safely?"

```text
Scout Agents
  ↓ Discovery Report
Architect Agents (YOU ARE HERE)
  ↓ Executable Migration Plan
Worker Agents → Validator Agents
  ↓ Execution Logs
Meta Agents (Learning)
```

Like Scouts, Architects use frontier-class language models (GPT-4-turbo or Claude-3) because their work requires multi-step reasoning about dependencies, tradeoffs, and risk.

---

### Capabilities

#### 1. Migration DAG Generation

A migration isn't a flat list of "copy Table A, then Table B." It's a directed acyclic graph (DAG) where table dependencies, foreign key relationships, and business logic constraints define execution order.

Architect Agents generate DAGs dynamically by:

- Analyzing foreign key relationships to establish dependency ordering
- Identifying independent table groups that can migrate in parallel
- Inserting validation checkpoints at critical junctures
- Planning reference data tables first (lookup tables, configuration tables) so dependent tables can validate during migration
- Accounting for circular dependencies through staged migration with deferred constraint checking

The output is an executable workflow definition for Temporal or Airflow — not a diagram that a human must translate into code.

#### 2. Transformation Code Generation

For every source-to-target field mapping identified by MABA's semantic analysis, the Architect generates executable transformation code in both SQL and Python:

- **Type conversions:** Oracle `NUMBER(38,0)` → PostgreSQL `BIGINT`, with overflow detection
- **Encoding normalization:** Latin-1 → UTF-8, with unmappable character handling
- **Datetime transformations:** Timezone-aware conversions, format standardization, epoch ↔ timestamp
- **Value transformations:** Currency conversion, unit normalization, code-to-description lookups
- **Structural transformations:** Denormalization, pivot/unpivot, nested JSON flattening, array decomposition
- **Data quality remediation:** Null handling, deduplication logic, format standardization

All generated code is **unit-tested against sample data** before being included in the plan. Test failures trigger regeneration with adjusted approach — the Architect doesn't ship untested transformations.

#### 3. Strategy Optimization

Not all migrations optimize for the same objective. Architect Agents configure strategy based on customer priorities:

**Speed-First Strategy**
Maximum parallelism, aggressive batching, deferred validation. For migrations with tight cutover windows where downtime cost exceeds error remediation cost.

**Safety-First Strategy**
Sequential execution, continuous validation, conservative checkpointing. For regulated environments where data integrity is non-negotiable and migration window is flexible.

**Balanced Strategy (Default)**
Parallel execution with inline validation, moderate checkpointing. Optimizes for total wall-clock time including error recovery. This is the default for most migrations.

The Architect selects a base strategy from the [evolutionary strategy pool](tournament-selection.md) and customizes it based on the specific migration's Discovery Report.

#### 4. Rollback Strategy Design

Every migration plan includes a rollback strategy — a tested path to restore the source system to its pre-migration state if the migration fails catastrophically.

Rollback planning includes:

- Pre-migration source snapshots with cryptographic hash verification
- Incremental rollback points at configurable intervals
- Dependency-aware rollback ordering (reverse of migration order)
- Partial rollback capability (roll back specific tables while preserving successfully migrated ones)
- Estimated rollback duration and resource requirements

The rollback strategy is not an afterthought added to the plan — it's generated simultaneously with the forward migration plan, because the Architect reasons about both directions of every transformation.

---

### Output: Executable Migration Plan

```yaml
MigrationPlan:
  id: plan-2026-02-001
  source: Oracle 19c (247 tables, 52M rows)
  target: PostgreSQL 16 (Snowflake)
  strategy: balanced
  estimated_duration: 18-24 hours
  estimated_cost: $420 (compute + LLM inference)

  dag:
    phases:
      - phase: 1 (Reference Data)
        tables: [config_tables, lookup_codes, region_master]
        parallel: true
        validation: post-phase

      - phase: 2 (Core Entities)
        tables: [customers, products, vendors]
        parallel: true
        depends_on: phase_1
        validation: continuous

      - phase: 3 (Transactional)
        tables: [orders, order_lines, invoices, payments]
        parallel: [orders + invoices] then [order_lines + payments]
        depends_on: phase_2
        validation: continuous
        checkpoint_every: 500,000 records

      - phase: 4 (Historical / Archive)
        tables: [audit_log, change_history, archived_orders]
        parallel: true
        depends_on: phase_3
        validation: post-phase

  transformations: 187 total
    type_conversions: 89
    encoding_fixes: 12
    datetime_transforms: 34
    value_transforms: 28
    structural_transforms: 15
    quality_remediations: 9

  unit_tests: 187 (all passing against sample data)

  rollback:
    snapshot_taken: pre-migration
    rollback_points: every 500,000 records
    estimated_rollback_time: 4-6 hours

  resource_requirements:
    cognitive_agents: 2
    worker_agents: 16 (scaling to 64 during phase 3)
    validator_agents: 4
    estimated_gpu_hours: 8
    estimated_cpu_hours: 384

  risk_mitigations:
    - "89 stored procedures: Transformation code generated and tested for all business logic"
    - "EBCDIC encoding: Pre-conversion step added in phase 1"
    - "Null constraint violations: Default value injection with audit logging"
```

---

### Collaboration with Other Agents

Architect Agents don't work in isolation. They engage in structured multi-agent conversations (via AutoGen) with:

- **Scout Agents:** Requesting additional analysis when the Discovery Report reveals ambiguities
- **Performance Optimizer:** Negotiating resource allocation and throughput targets
- **Validator Agents:** Confirming that validation checkpoints are sufficient for compliance requirements
- **AMANI:** Generating human-readable plan summaries for stakeholder approval

These conversations are logged and analyzed by Meta Agents to improve future planning accuracy.

→ [Worker Agents](worker-agents.md)
→ [Evolutionary Strategy Selection](tournament-selection.md)
→ [Cognitive Agents Overview](three-layer-architecture/cognitive-agents.md)
