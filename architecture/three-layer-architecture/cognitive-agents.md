---
description: Understanding, planning, and designing
---

# L1: Cognitive Agents

## Understanding, Planning, Designing

Cognitive Agents serve as Sensei's reasoning layer. They perform the tasks that require genuine language understanding: reading schemas, inferring business logic, designing migration strategies, and writing transformation code.

---

### Model Requirements

Cognitive tasks require frontier-class language models with strong reasoning capabilities:

| Capability                | Why Frontier Models                                                                                       |
| ------------------------- | --------------------------------------------------------------------------------------------------------- |
| Schema interpretation     | Understanding that `cst_ref` means "customer reference" requires semantic reasoning, not pattern matching |
| Stored procedure analysis | Extracting business rules from PL/SQL or T-SQL requires code comprehension                                |
| Strategy design           | Balancing speed, safety, and resource tradeoffs requires multi-objective reasoning                        |
| Code generation           | Writing correct, tested transformation code requires understanding both source and target semantics       |

**Primary models:** GPT-4-turbo, Claude-3 (configurable per deployment) **Framework:** LangChain + LangGraph for agent lifecycle, tool binding, and memory management **Multi-agent coordination:** AutoGen for collaborative agent conversations

---

### Scout Agents

Scout Agents analyze source and target systems to build a comprehensive understanding of the migration landscape.

#### Capabilities

**Schema reading and interpretation.** Scout Agents parse database catalogs to extract table definitions, column types, constraints, indexes, and relationships. But they go beyond structural extraction — using LLM inference to understand _meaning_. When a Scout encounters a column named `fld_047` of type `VARCHAR(20)` with values like `ACT`, `SUS`, `TRM`, it infers that this is likely an account status field with codes for Active, Suspended, and Terminated.

**Business logic discovery.** Scout Agents analyze stored procedures, triggers, views, and computed columns to extract implicit business rules that schema-level analysis misses. A trigger that sets `modified_date = GETDATE()` on every update, a view that filters `WHERE status != 99`, a computed column that calculates `total = quantity * unit_price * (1 - discount)` — these are all business logic that must be preserved in migration.

**Data quality profiling.** Scout Agents compute statistical profiles for every column: null rates, distinct value counts, value distributions, pattern frequencies, encoding detection, and cross-column correlations. These profiles become the distributional fingerprints used by MABA's Semantic Pattern Recognition.

**Complexity and risk prediction.** Based on discovered schema structure, business logic density, data quality issues, and pattern library matches, Scout Agents generate a migration complexity score and risk assessment. This feeds Architect Agents' strategy optimization.

#### Output

Scout Agents produce a **Migration Discovery Report** containing:

- Complete schema map with inferred semantics
- Business logic inventory with extracted rules
- Data quality profile per column
- PII detection results
- Complexity score and risk factors
- Recommended transformation approaches
- Pattern library matches from prior migrations

Every discovery is embedded as a vector and stored in the persistent knowledge base (Pinecone/Weaviate), enabling semantic retrieval in future migrations.

---

### Architect Agents

Architect Agents consume Scout discoveries to design optimal migration strategies.

#### Capabilities

**DAG generation.** Architect Agents create migration directed acyclic graphs (DAGs) that sequence operations to respect dependencies (foreign keys must be migrated before referencing tables), minimize risk (reversible operations before irreversible ones), and optimize throughput (independent tables can be migrated in parallel).

**Transformation code generation.** For each mapping in the DAG, Architect Agents generate executable transformation code in SQL and Python. The generated code handles type conversions, encoding normalization, value transformations, structural transformations (denormalization, pivoting), and data quality remediation. All generated code is unit-tested against sample data before execution.

**Strategy optimization.** Architect Agents optimize for configurable tradeoffs:

- **Speed-first:** Maximize throughput, accept higher risk of recoverable errors
- **Safety-first:** Minimize risk, accept slower throughput with more validation checkpoints
- **Balanced:** Optimize for both, dynamically adjusting based on observed error rates

**Rollback strategy design.** For every migration plan, Architect Agents generate a corresponding rollback plan that can restore the source system to its pre-migration state. The rollback plan is tested against a dry-run before the migration begins.

#### Output

Architect Agents produce an **Executable Migration Plan** containing:

- Migration DAG with dependency ordering
- Transformation code per mapping (SQL + Python)
- Unit test results for generated code
- Performance estimates based on pattern library data
- Rollback strategy with tested restore procedures
- Resource requirements (compute, network, storage)
- Workflow definition for Temporal or Airflow execution

→ [Scout Agents specification](../scout-agents.md) → [Architect Agents specification](../architect-agents.md)
