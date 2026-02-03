# Data Flow

## End-to-End Journey of Data Through Sensei

This page traces the path of data from source system to target system, showing every transformation, validation, and checkpoint along the way.

---

### The Complete Pipeline

```text
SOURCE DATABASE
      │
      ↓ (1) Connection & Authentication
┌─────────────────────────────────────────────────────────────┐
│                    INGESTION                                 │
│  Connector → Parser → Normalizer → Intermediate Repr.       │
└─────────────────────────────────────────────────────────────┘
      │
      ↓ (2) Schema Analysis
┌─────────────────────────────────────────────────────────────┐
│                 SCHEMA ANALYSIS                              │
│  Structural → Statistical → Semantic → Pattern Matching     │
└─────────────────────────────────────────────────────────────┘
      │
      ↓ (3) Mapping & Planning
┌─────────────────────────────────────────────────────────────┐
│                 TRANSFORMATION PLANNING                      │
│  Pattern Library → LLM Inference → Code Generation → DAG    │
└─────────────────────────────────────────────────────────────┘
      │
      ↓ (4) Human Approval (if required)
┌─────────────────────────────────────────────────────────────┐
│                 APPROVAL CHECKPOINT                          │
│  AMANI presents plan → Stakeholder reviews → Approval/Edit  │
└─────────────────────────────────────────────────────────────┘
      │
      ↓ (5) Execution
┌─────────────────────────────────────────────────────────────┐
│               TRANSFORMATION EXECUTION                       │
│  Read Batch → Apply Transform → Validate → Write Batch      │
│       ↑                              │                       │
│       └──── Error Recovery ──────────┘                       │
└─────────────────────────────────────────────────────────────┘
      │
      ↓ (6) Post-Migration Verification
┌─────────────────────────────────────────────────────────────┐
│                 VERIFICATION                                 │
│  Structural → Statistical → Referential → Semantic →        │
│  Behavioral (Time-Travel Testing)                           │
└─────────────────────────────────────────────────────────────┘
      │
      ↓ (7) Certification
┌─────────────────────────────────────────────────────────────┐
│                 CERTIFICATION                                │
│  Evidence Assembly → Certificate Generation → Delivery      │
└─────────────────────────────────────────────────────────────┘
      │
      ↓
TARGET DATABASE
```

---

### Stage 1: Ingestion

**Input:** Source database connection credentials
**Output:** Normalized Intermediate Representation (IR)

The appropriate connector establishes a connection to the source:

```yaml
Connection:
  type: postgresql
  host: source-db.example.com
  port: 5432
  database: production
  credentials: vault://secrets/source-db
  ssl: required
```

Data is read in batches (default 10,000 rows) and normalized:

```yaml
Intermediate Representation:
  table: customers
  row:
    - column: id
      declared_type: BIGINT
      value: 12345
      inferred_type: integer
    - column: name
      declared_type: VARCHAR(100)
      value: 'Acme Corp'
      inferred_type: string
      encoding: UTF-8
```

**Data protection:** Raw values are processed in memory, never persisted to disk in unencrypted form.

---

### Stage 2: Schema Analysis

**Input:** IR + Database catalog
**Output:** Semantic schema understanding + Distributional fingerprints

Three-phase analysis builds complete understanding:

| Phase       | Duration | Output                                                    |
| ----------- | -------- | --------------------------------------------------------- |
| Structural  | Seconds  | Tables, columns, constraints, relationships               |
| Statistical | Minutes  | Fingerprints (128-dim vectors per column)                 |
| Semantic    | Minutes  | Business meaning, PII classification, mapping suggestions |

**Data protection:** Fingerprints are statistical aggregates — no individual values are exposed.

---

### Stage 3: Mapping & Planning

**Input:** Schema understanding + Target schema definition
**Output:** Migration DAG + Generated transformation code

Pattern library is queried first (for known mappings), then LLM inference for novel cases:

```yaml
Mapping:
  source: customers.cust_ref (VARCHAR(15))
  target: customers.customer_reference (VARCHAR(20))
  confidence: 0.96
  transformation: |
    CAST(source.cust_ref AS VARCHAR(20))
  validation_rules:
    - not_null
    - unique
    - max_length(20)
```

DAG orders tables by dependency to maintain referential integrity:

```text
Phase 1: reference_data (no dependencies)
Phase 2: customers (depends on regions)
Phase 3: orders (depends on customers, products)
Phase 4: order_items (depends on orders, products)
```

---

### Stage 4: Approval Checkpoint

**Input:** Migration plan
**Output:** Approved plan (possibly with modifications)

AMANI presents the plan to appropriate stakeholders:

- **High-confidence mappings (>0.95):** Auto-approved unless configured otherwise
- **Medium-confidence (0.80-0.95):** Flagged for review
- **Low-confidence (<0.80):** Require explicit approval

Stakeholders can:

- Approve as-is
- Modify specific mappings
- Reject and request re-analysis
- Adjust strategy parameters

---

### Stage 5: Transformation Execution

**Input:** Approved plan + Source data
**Output:** Transformed data in target

The execution loop:

```python
WHILE tables_remaining:
    batch = read_batch(source, size=10000)

    FOR row IN batch:
        transformed = apply_transformation(row)
        validation = validate_inline(transformed)

        IF validation.passed:
            write_buffer.add(transformed)
        ELSE:
            recovery = attempt_recovery(row, validation.error)
            IF recovery.success:
                write_buffer.add(recovery.result)
            ELSE:
                error_queue.add(row, validation.error)

    flush_buffer_to_target()
    checkpoint()
```

**Checkpointing:** Every N records (default 100,000), state is checkpointed. If the process fails, it resumes from the last checkpoint.

**Error handling:** Errors are classified, recovery attempted automatically, and unrecoverable errors queued for human review.

---

### Stage 6: Post-Migration Verification

**Input:** Completed migration + Pre-captured source state
**Output:** Validation results

Five validation perspectives run:

| Validation  | What's Compared                     | Pass Criteria        |
| ----------- | ----------------------------------- | -------------------- |
| Structural  | Target schema vs. plan              | 100% match           |
| Statistical | Column statistics source vs. target | Within tolerance     |
| Referential | FK relationships                    | 100% resolution      |
| Semantic    | Business rules                      | 94-98% pass          |
| Behavioral  | Historical query replay             | Business equivalence |

---

### Stage 7: Certification

**Input:** Validation results + Evidence chain
**Output:** Behavioral Equivalence Certificate

The certificate bundles:

- All validation results
- Hash chain from source snapshots to target state
- Coverage metrics
- Any documented exceptions
- Digital signature

Certificate is delivered to stakeholders and stored for audit.

---

### Data Retention

| Data Type          | Retention During Migration | Post-Migration Retention     |
| ------------------ | -------------------------- | ---------------------------- |
| Source data        | In-memory only             | Not retained by Sensei       |
| Transformed data   | Write-through to target    | Lives in target database     |
| Fingerprints       | Stored in pattern library  | Retained (privacy-preserved) |
| Validation results | Stored for certification   | 7 years (configurable)       |
| Checkpoints        | Stored until completion    | Deleted after certification  |
| Certificates       | Delivered to customer      | Customer controls retention  |

→ [Component Orchestration](component-orchestration.md)
→ [Agent Communication](agent-communication.md)
→ [Integration Overview](README.md)
