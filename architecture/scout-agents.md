---
description: Discovery and schema analysis agents
---

# Scout Agents

## Understanding Before Changing

Scout Agents are the first agents to engage with a migration. Before any data moves, Scouts analyze the source and target systems to build a complete understanding of what exists, what it means, and what could go wrong.

---

### Role in the Hierarchy

Scout Agents are one of two Cognitive Agent types (alongside [Architect Agents](architect-agents.md)). They use frontier-class language models (GPT-4-turbo or Claude-3) because their work requires genuine reasoning — interpreting cryptic column names, inferring business logic from data patterns, and predicting risks that aren't visible in metadata alone.

```text
Scout Agents (YOU ARE HERE)
  ↓ Discovery Report
Architect Agents
  ↓ Migration Plan
Worker Agents → Validator Agents
  ↓ Execution Logs
Meta Agents (Learning)
```

---

### Capabilities

#### 1. Schema Reading and Interpretation

Scouts connect to source databases and extract structural metadata: tables, columns, types, constraints, indexes, relationships. But metadata extraction is the easy part — any ETL tool can do it.

What makes Scouts different is **semantic interpretation**. Given a column named `cust_ref` with type `VARCHAR(20)`, a Scout Agent examines:

- Sample values: `CR-2024-00147`, `CR-2024-00148`, `CR-2024-00149`
- Value distribution: sequential, zero nulls, 100% unique
- Pattern: `[A-Z]{2}-\d{4}-\d{5}`
- Context: table name `orders`, adjacent columns include `order_date`, `total_amount`

The Scout infers: "This is a customer reference number, likely a foreign key to a customers table even though no formal FK constraint exists. The `CR-` prefix suggests it's a composite key with a year component."

This inference is impossible with syntactic matching. It requires the kind of reasoning that frontier LLMs provide.

#### 2. Business Logic Discovery

Legacy systems encode critical business logic in places that schema analysis can't reach:

- **Stored procedures** contain transformation rules, validation checks, and workflow logic
- **Triggers** enforce constraints and cascade updates that aren't in foreign key definitions
- **Views** encode business calculations and reporting logic
- **Application configuration tables** store enumerations, status codes, and business rules

Scout Agents read and interpret these artifacts using LLM analysis. A stored procedure that checks `IF status = 99 THEN skip_record` tells the Scout that status code 99 means "soft deleted" — critical knowledge for migration correctness that exists nowhere in the schema metadata.

#### 3. Data Quality Profiling

For every column in the source system, Scouts compute a **distributional fingerprint** — a statistical profile that characterizes the data:

- Null rate and uniqueness ratio
- Value range, central tendency, standard deviation
- Character-level pattern frequencies (email, phone, date, currency, URL)
- Data type distribution (what percentage of values parse as integer, float, date, boolean)
- Correlation coefficients with other columns (mutual information)

These fingerprints serve two purposes: they feed into [MABA's Semantic Pattern Recognition](../../product/platform/maba/ingestion-pipeline/semantic-pattern-recognition.md) for schema mapping, and they establish baseline quality metrics that [KORA](../../product/platform/kora/README.md) uses for post-migration validation.

#### 4. Complexity and Risk Prediction

Using the combined schema structure, business logic inventory, and data quality profiles, Scouts generate a migration complexity score and risk assessment:

- **Complexity factors:** Number of tables, relationship density, stored procedure count, data volume, transformation requirements, encoding diversity
- **Risk factors:** PII presence, data quality issues, undocumented business logic, missing constraints, temporal dependencies
- **Historical comparison:** Similarity search against the pattern library for migrations with similar profiles, including their actual outcomes

The prediction isn't just a number — it's a structured risk register with specific items, severities, and recommended mitigations.

---

### Output: Migration Discovery Report

Scout Agents produce a structured Discovery Report that feeds into [Architect Agent](architect-agents.md) planning:

```yaml
DiscoveryReport:
  source_system:
    engine: Oracle 19c
    tables: 247
    total_columns: 3,891
    total_rows: ~52M
    stored_procedures: 89
    triggers: 34
    views: 56

  schema_map:
    tables:
      - name: CUSTOMER_DATA_V2_FINAL
        inferred_purpose: "Active customer records (V2 is current version)"
        columns: 23
        pii_detected: [email, phone, ssn_encrypted]
        quality_issues:
          - "phone column: 12% null, 3% invalid format"
          - "email column: 0.5% malformed addresses"
        hidden_relationships:
          - "cust_ref links to ORDERS.customer_id (no formal FK)"
        business_logic:
          - "status=99 means soft deleted (from SP: proc_cleanup_customers)"
          - "tier column uses codes: 1=Bronze, 2=Silver, 3=Gold (from config table)"

  complexity_score: 7.2 / 10
  estimated_duration: 18-24 hours
  risk_register:
    high:
      - "89 stored procedures contain business logic not reflected in schema"
      - "3 tables use EBCDIC encoding mixed with UTF-8"
    medium:
      - "12% null rate in phone column exceeds target schema NOT NULL constraint"
      - "DATE columns use 4 different format patterns"
    low:
      - "Table naming inconsistency (mix of singular/plural)"

  pattern_library_matches:
    - migration_id: 847 (Oracle HR → PostgreSQL, similarity: 0.91)
    - migration_id: 612 (Oracle ERP → Snowflake, similarity: 0.87)
```

---

### Memory and Learning

Every Scout discovery is embedded as a vector and stored in the persistent knowledge base. This means:

- The first time a Scout encounters Oracle's `NUMBER(38,0)` being used as a boolean (0/1 values), it takes LLM analysis to recognize the pattern
- The 100th time, the pattern library returns a high-confidence match instantly, skipping LLM inference entirely
- The pattern library also knows that this situation correlates with Oracle-to-PostgreSQL migrations, that the correct target type is `BOOLEAN`, and that a `CASE WHEN col = 1 THEN true ELSE false END` transformation has a 99.7% success rate

Scouts get faster and more accurate with every migration the platform processes.

→ [Architect Agents](architect-agents.md)
→ [MABA: Semantic Pattern Recognition](../../product/platform/maba/ingestion-pipeline/semantic-pattern-recognition.md)
→ [Cognitive Agents Overview](three-layer-architecture/cognitive-agents.md)
