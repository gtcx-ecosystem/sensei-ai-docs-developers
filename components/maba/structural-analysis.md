# Structural Analysis

## Phase 1: Cataloging What Exists

Structural analysis is the first phase of MABA's [Schema Analysis Engine](../../../product/platform/maba/ingestion-pipeline/schema-analysis.md). It extracts everything that can be learned from database catalogs and file metadata — the foundation upon which statistical and semantic analysis build.

---

### What Gets Extracted

#### From Relational Databases

| Element                | Source                                            | Why It Matters                                          |
| ---------------------- | ------------------------------------------------- | ------------------------------------------------------- |
| **Tables**             | `information_schema.tables` or equivalent catalog | Migration scope, row counts, dependencies               |
| **Columns**            | `information_schema.columns`                      | Type mapping, nullability, default values               |
| **Primary keys**       | Constraint catalog                                | Record identity, deduplication strategy                 |
| **Foreign keys**       | Constraint catalog                                | Dependency ordering, referential integrity requirements |
| **Unique constraints** | Constraint catalog                                | Business key identification                             |
| **Check constraints**  | Constraint catalog                                | Business rule detection                                 |
| **Indexes**            | Index catalog                                     | Performance optimization, covering index decisions      |
| **Stored procedures**  | Procedure catalog + definition text               | Business logic discovery (analyzed by Scout Agents)     |
| **Triggers**           | Trigger catalog + definition text                 | Implicit cascade rules, audit logic                     |
| **Views**              | View definitions                                  | Computed columns, reporting logic, derived tables       |
| **Sequences**          | Sequence catalog                                  | Auto-increment behavior, ID generation strategy         |
| **Column comments**    | Comment metadata                                  | Documentation hints (often outdated but still useful)   |

#### From File-Based Sources

For CSV, JSON, Excel, and other file formats, structural analysis infers schema from content:

- **Column names:** From headers (CSV/Excel) or keys (JSON)
- **Data types:** Inferred from value inspection across sample rows
- **Nullability:** Observed null/empty rates
- **Relationships:** Inferred from naming patterns and value overlap between files
- **Constraints:** Inferred from uniqueness analysis and range checking

---

### Implicit Relationship Discovery

Formal foreign keys are the exception, not the rule, in legacy systems. Many databases have relationships that exist by convention rather than constraint.

Structural analysis identifies implicit relationships through:

**Name matching:** Columns in different tables with similar names (`customer_id` in `orders` → `id` in `customers`)

**Value overlap analysis:** Columns whose distinct value sets substantially overlap, even with different names (`cust_ref` in `orders` → `reference_number` in `customers`)

**Pattern consistency:** Columns with the same format pattern across tables (e.g., `CR-YYYY-NNNNN` format appearing in both `orders` and `invoices`)

These implicit relationships are flagged with lower confidence than formal FKs but are critical for building correct migration DAGs.

---

### Business Logic Extraction

Stored procedures, triggers, and views contain business logic invisible to schema-level analysis. Structural analysis extracts and catalogs these artifacts:

**Stored procedures:** The full definition text is extracted and passed to [Scout Agents](../../architecture/scout-agents.md) for LLM analysis. A procedure containing `IF status = 99 THEN skip_record` reveals that status 99 means "soft deleted" — critical knowledge for correct migration.

**Triggers:** Trigger definitions reveal cascade behavior, audit logging, and validation rules not encoded in constraints. A `BEFORE INSERT` trigger that enforces `email LIKE '%@%.%'` reveals a validation rule that must be preserved in the target system.

**Views:** View definitions reveal computed columns, business calculations, and reporting derivations. A view that calculates `total_with_tax AS (amount * (1 + tax_rate))` reveals business logic that must be replicated in the target schema.

---

### Output

Structural analysis produces a schema catalog that serves as input to both the Statistical Phase and the Scout Agent Discovery Report:

```yaml
StructuralCatalog:
  database: Oracle 19c
  schema: HR_PROD
  tables: 247
  total_columns: 3,891
  explicit_foreign_keys: 189
  implicit_relationships_detected: 47
  stored_procedures: 89
  triggers: 34
  views: 56
  sequences: 23
  check_constraints: 145
```

→ [Statistical Analysis](statistical-analysis.md)
→ [Schema Analysis Overview](../../../product/platform/maba/ingestion-pipeline/schema-analysis.md)
