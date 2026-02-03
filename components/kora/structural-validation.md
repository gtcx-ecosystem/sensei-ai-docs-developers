# Structural Validation

## Did the Schema Migrate Correctly?

Structural validation is the first and simplest of KORA's five validation perspectives. It verifies that the target database's structure faithfully represents the source schema after planned transformations.

---

### What Gets Checked

| Check                  | Method                     | Pass Criteria                                                      |
| ---------------------- | -------------------------- | ------------------------------------------------------------------ |
| **Table existence**    | Catalog comparison         | Every planned source table has a corresponding target table        |
| **Column existence**   | Catalog comparison         | Every planned source column exists in the target with correct name |
| **Data types**         | Type mapping verification  | Target types match the transformation plan's specified types       |
| **Nullability**        | Constraint comparison      | NOT NULL constraints preserved or intentionally changed            |
| **Primary keys**       | Constraint comparison      | PK definitions match across source and target                      |
| **Foreign keys**       | Constraint comparison      | FK relationships correctly established in target                   |
| **Unique constraints** | Constraint comparison      | Uniqueness rules preserved                                         |
| **Check constraints**  | Constraint comparison      | Business rule constraints recreated                                |
| **Indexes**            | Index catalog comparison   | Planned indexes created with correct columns and properties        |
| **Sequences**          | Sequence comparison        | Auto-increment sequences created with correct starting values      |
| **Default values**     | Column metadata comparison | Default values correctly set per transformation plan               |

---

### What It Catches

Structural validation catches infrastructure-level migration failures:

- **Missing tables:** A table that should have been created wasn't (often due to dependency ordering errors)
- **Wrong data types:** A `DECIMAL(10,2)` that became `DECIMAL(10,0)`, silently losing precision
- **Missing constraints:** A `NOT NULL` constraint that was dropped, allowing invalid data in future operations
- **Broken indexes:** An index that references the wrong columns or was never created
- **Missing sequences:** An auto-increment that starts at 1 instead of continuing from the source's last value

---

### What It Can't Catch

Structural validation is necessary but not sufficient. It verifies the container, not the contents. A structurally perfect migration might still have:

- Wrong data values in correctly-typed columns
- Missing records (same schema, fewer rows)
- Corrupted relationships (FK constraints exist but reference wrong records)
- Lost business logic (schema is correct but stored procedures weren't migrated)

These are caught by the other four validation perspectives.

---

### Performance

| Metric                  | Value                                   |
| ----------------------- | --------------------------------------- |
| Throughput              | 50,000 tables/hour                      |
| Accuracy                | 100% (deterministic)                    |
| Resource cost           | Minimal (CPU-only, metadata comparison) |
| Duration for 247 tables | ~18 seconds                             |

Structural validation is fast because it compares metadata catalogs, not data. It runs first in the validation pipeline because failures here indicate fundamental migration infrastructure problems that must be fixed before other validation is meaningful.

→ [Statistical Validation](statistical-validation.md)
→ [Multi-Source Validation Overview](multi-source-validation.md)
