# Learning Agents

## Extracting Durable Intelligence from Every Migration

Learning Agents are the knowledge miners of the Meta Agent layer. After each migration, they analyze execution logs to extract patterns that make future migrations faster, more accurate, and more autonomous.

---

### Role in the Hierarchy

Learning Agents are one of two Meta Agent types (alongside the [Evolutionary Selection](evolutionary-selection.md) system). They use GPT-4-class models for analysis and pattern abstraction.

```text
Scout Agents → Architect Agents
  ↓
Worker Agents → Validator Agents
  ↓ Execution Logs
Learning Agents (YOU ARE HERE) → Evolutionary Selection
  ↓ Pattern Library Updates     ↓ Strategy Pool Updates
  Knowledge Base (Pinecone + Neo4j)
```

---

### What Gets Extracted

Learning Agents process four categories of patterns from every completed migration:

#### 1. Schema Patterns

Structural characteristics of source and target schemas that recur across migrations.

**Examples:**

- "Oracle HR modules consistently use `NUMBER(38,0)` for boolean fields"
- "SAP tables with prefix `BSEG` contain financial line items with predictable column structures"
- "Legacy COBOL systems encode dates as 6-digit integers `YYMMDD`"

**Storage:** Vector embeddings in Pinecone (for similarity search) + structured nodes in Neo4j (for relationship traversal)

#### 2. Transformation Patterns

Which transformations worked reliably for specific source-target type combinations.

**Examples:**

- "Oracle `CLOB` → PostgreSQL `TEXT`: direct cast, no transformation needed, 99.99% success rate"
- "EBCDIC packed decimal → UTF-8 integer: unpack with `COMP-3` decoder, validate range, 97% success rate"
- "Mixed-encoding VARCHAR: detect encoding per-value using `chardet`, convert individually, 94% success rate"

**Storage:** Indexed by source-target type pair for fast retrieval during [Architect Agent](architect-agents.md) planning

#### 3. Error Patterns

Error types encountered, their frequency, root causes, and proven resolutions.

**Examples:**

- "Constraint violation on `NOT NULL` column: 78% caused by source data quality issues, 22% by incorrect default mapping. Best resolution: inject sentinel value with audit flag."
- "Connection timeout during large table migration: correlates with tables >10M rows on shared database servers. Best resolution: off-peak scheduling + batch size reduction."

**Storage:** Error-solution pairs in Neo4j knowledge graph, linked to schema patterns and migration profiles

#### 4. Performance Patterns

Resource utilization, throughput curves, and bottleneck classifications.

**Examples:**

- "PostgreSQL write throughput drops 40% when table has >5 indexes: disable indexes during load, rebuild after"
- "Ray worker scaling beyond 64 nodes shows diminishing returns for I/O-bound migrations"
- "Compression ratio >3:1 on text-heavy tables: worth the CPU overhead for network-constrained deployments"

**Storage:** Time-series data linked to migration profiles for [Performance Optimizer](../../product/why-sensei/overview/compound-intelligence.md) training

---

### Abstraction and Privacy

Learning Agents don't store raw migration data. They extract **abstract patterns** that capture transferable intelligence without customer-specific details.

**Raw log entry (NOT stored):**

```text
Table: ACME_CORP.employees, Column: ssn,
Error: Truncation from VARCHAR(11) to VARCHAR(9)
Value: "123-45-6789" → "123-45-67" (data loss)
```

**Abstracted pattern (stored):**

```yaml
pattern:
  category: type_truncation
  source_type: VARCHAR(11)
  target_type: VARCHAR(9)
  data_characteristic: formatted_identifier_with_separators
  risk: data_loss
solution:
  approach: strip_separators_before_mapping
  target_type_override: VARCHAR(9) sufficient after separator removal
  confidence: 0.96
```

The abstraction strips table names, column names, actual values, and customer identity. What remains is the structural pattern and its solution — transferable to any migration with the same type mismatch.

For cross-customer learning, [AMANI's privacy-preserving pipeline](../../product/platform/amani/privacy-preserving-learning.md) adds differential privacy noise before patterns enter the shared knowledge base.

---

### Knowledge Graph Construction

Learning Agents don't just store patterns — they build a knowledge graph that encodes relationships between patterns:

```text
SchemaPattern(Oracle_HR) ──[commonly_requires]──→ TransformationPattern(NUMBER_to_BOOLEAN)
                          ──[frequently_causes]──→ ErrorPattern(null_constraint_violation)

ErrorPattern(null_constraint)──[resolved_by]──→ SolutionPattern(sentinel_value_injection)
                              ──[occurs_in]──→ MigrationProfile(Oracle_to_PostgreSQL_HR)

MigrationProfile(Oracle_to_PG_HR) ──[similar_to]──→ MigrationProfile(SAP_to_PG_HR)
                                   ──[best_strategy]──→ OptimizationStrategy(safety_first_v47)
```

This graph enables structural reasoning that vector similarity alone can't provide. When a Scout Agent reports "Oracle HR schema with 47 tables," the knowledge graph traversal reveals not just similar schemas but the entire chain of likely transformations, probable errors, proven solutions, and optimal strategies.

---

### Cross-Customer Intelligence

The most powerful learning happens when patterns from different customers are aggregated:

- **Individual customer:** "This Oracle HR migration had 15 date format errors"
- **Aggregated insight:** "Oracle HR migrations have a 73% probability of date format errors, concentrated in columns matching pattern `DT_*` or `*_DATE`"

This aggregation is what makes the platform's compound intelligence possible — and it's what [AMANI's privacy guarantees](../components/amani/differential-privacy.md) protect.

Learning Agents are configured with three participation modes:

- **Full:** Customer patterns contribute to and benefit from the shared library (default)
- **Receive-only:** Customer benefits from shared library but doesn't contribute
- **Isolated:** Customer maintains a private pattern library with no cross-customer learning

→ [Evolutionary Selection](evolutionary-selection.md)
→ [Meta Agents Overview](three-layer-architecture/meta-agents.md)
→ [Cross-Migration Vector Learning](vector-learning.md)
