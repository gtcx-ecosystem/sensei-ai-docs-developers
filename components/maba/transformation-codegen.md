# Transformation Code Generation

## From Mapping to Executable Code — Automatically

Once MABA has determined the semantic mapping between source and target schemas, it generates the actual transformation code that converts data. Every transformation is produced in SQL and Python, unit-tested against sample data, and validated before inclusion in the migration plan.

---

### What Gets Generated

#### Type Conversions

Converting between database types with full awareness of edge cases:

- `Oracle NUMBER(38,0)` → `PostgreSQL BIGINT` with overflow detection
- `SQL Server DATETIME2` → `PostgreSQL TIMESTAMPTZ` with timezone inference
- `MySQL ENUM('active','inactive')` → `PostgreSQL VARCHAR` with constraint generation
- `COBOL COMP-3 packed decimal` → standard integer/decimal with sign handling

#### Encoding Normalization

Standardizing character encoding across the migration:

- Latin-1 → UTF-8 with unmappable character substitution
- EBCDIC → UTF-8 with code page detection
- Mixed encoding fields (detected per-value using `chardet`) → UTF-8 with per-value conversion
- BOM handling and normalization

#### Datetime Transformations

Timezone-aware date/time conversions:

- Unix epoch (seconds or milliseconds) → TIMESTAMPTZ
- Excel serial dates → ISO 8601
- Oracle DATE (includes time component) → separate DATE and TIME columns
- CJK date formats (`2024年01月15日`) → ISO 8601
- Ambiguous date formats (MM/DD vs DD/MM) → resolved using statistical analysis of value ranges

#### Value Transformations

Business logic conversions:

- Currency conversion with historical exchange rate lookup
- Unit normalization (imperial ↔ metric)
- Code-to-description lookups (status `99` → `"soft_deleted"`)
- Address standardization and geocoding
- Phone number normalization (E.164 format)

#### Structural Transformations

Schema shape changes:

- Denormalization (joining reference tables into flat records)
- Normalization (splitting wide tables into related entities)
- Pivot / unpivot operations
- Nested JSON → flat relational columns
- Array decomposition (single row with array → multiple rows)
- Hierarchical data restructuring

#### Data Quality Remediation

Fixing issues detected during schema analysis:

- Null handling: default value injection, sentinel values, or nullable column creation
- Deduplication: exact and fuzzy matching with configurable merge rules
- Format standardization: phone numbers, postal codes, dates
- Truncation prevention: type widening when source values exceed target column size

---

### Unit Testing

Every generated transformation is tested before execution:

1. **Sample data extraction:** 100 representative source records (diverse, including edge cases)
2. **Transformation execution:** Run the generated code against sample data
3. **Output validation:** Verify output types match target schema, no data loss, no precision loss, no encoding errors
4. **Roundtrip testing:** For reversible transformations, verify that applying the transformation and its inverse produces the original values

Test results are included in the migration plan. [Architect Agents](../../architecture/architect-agents.md) do not include transformations with failing tests — they regenerate with a different approach.

---

### Dual Output Format

Transformations are generated in both SQL and Python:

**SQL:** Used when the transformation can be expressed as a database-native operation. Executed at the source or target database for maximum performance.

```sql
-- Example: Oracle DATE to PostgreSQL TIMESTAMPTZ
SELECT
  CAST(src.created_date AS TIMESTAMP) AT TIME ZONE 'UTC' AS created_date,
  CASE WHEN src.status = 99 THEN NULL ELSE src.status END AS status,
  COALESCE(src.email, 'unknown@placeholder.com') AS email
FROM source_table src
WHERE src.status != 99  -- Exclude soft-deleted records
```

**Python:** Used for complex logic that can't be expressed in SQL, cross-table operations, and non-SQL sources. Executed within Ray workers.

```python
# Example: COBOL packed decimal with business logic
def transform_amount(packed_bytes: bytes, currency_code: str) -> Decimal:
    value = unpack_comp3(packed_bytes)
    if currency_code != 'USD':
        value = convert_currency(value, currency_code, 'USD',
                                 rate_date=migration_snapshot_date)
    return Decimal(str(value)).quantize(Decimal('0.01'))
```

---

### Code Quality

Generated transformations follow quality standards:

- **Idempotent:** Running the transformation twice produces the same result
- **Null-safe:** Explicit null handling for every input path
- **Deterministic:** Same input always produces same output (no random elements)
- **Documented:** Each transformation includes a comment explaining the semantic mapping it implements
- **Parameterized:** Configurable thresholds and defaults, not hardcoded magic values

---

### Feedback to Pattern Library

When transformations execute successfully during migration and pass KORA validation, the transformation code is stored in the [Pattern Library](../../../product/platform/maba/transformation-engine/pattern-library.md) alongside the semantic mapping that produced it. Future migrations with similar source-target type pairs can reuse proven transformation code rather than generating new code from scratch.

→ [How MABA Works](how-it-works.md)
→ [Performance](../../../product/platform/maba/performance.md)
