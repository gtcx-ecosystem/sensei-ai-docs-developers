# Referential Validation

## Are All the Relationships Intact?

Referential validation verifies that every foreign key relationship in the target database resolves correctly — no orphan records, no broken references, no cascade failures.

---

### What Gets Checked

| Check                      | Method                                                              | Failure Indicates                                 |
| -------------------------- | ------------------------------------------------------------------- | ------------------------------------------------- |
| **FK resolution**          | For every child record, verify parent exists                        | Orphan records — child migrated but parent didn't |
| **Reverse FK**             | For every parent record, verify expected children exist             | Missing child records                             |
| **Cascade behavior**       | Simulate UPDATE/DELETE cascades, verify propagation                 | Missing or incorrect cascade rules                |
| **Implicit relationships** | Verify value overlap for Scout-detected implicit FKs                | Undocumented relationships broken                 |
| **Self-referential FKs**   | Verify hierarchical data integrity (parent-child within same table) | Hierarchy corruption                              |
| **Circular dependencies**  | Verify multi-table reference chains complete                        | Partial migration of circular reference groups    |

---

### Why This Matters

Referential integrity failures are among the most common and most damaging migration defects:

- An order referencing a non-existent customer → application errors when loading order details
- An employee assigned to a deleted department → reporting queries return wrong results
- A payment linked to a missing invoice → financial reconciliation failures

These failures often pass row-count validation (all records exist) and even statistical validation (distributions look correct). Only referential validation catches them by verifying the actual connections between records.

---

### Implicit Relationship Verification

Many legacy databases have relationships by convention rather than constraint. [Scout Agents](../../architecture/scout-agents.md) detect these implicit relationships during discovery. KORA validates them alongside formal FKs:

- `orders.cust_ref` → `customers.reference_number` (no formal FK, but Scout detected value overlap)
- `transactions.dept_code` → `departments.code` (naming convention relationship)
- `invoices.order_num` → `orders.order_number` (different column names, same values)

Implicit relationship validation has a configurable tolerance — 100% match is required for formal FKs, but 95%+ match may be acceptable for implicit relationships where the source system itself had some orphan records.

---

### Performance

| Metric                                                    | Value                                                              |
| --------------------------------------------------------- | ------------------------------------------------------------------ |
| Throughput                                                | 80,000 relationships/hour                                          |
| Accuracy                                                  | 100% (deterministic)                                               |
| Resource cost                                             | Medium (I/O bound — requires reading both parent and child tables) |
| Duration for 236 relationships (189 formal + 47 implicit) | ~11 seconds                                                        |

→ [Semantic Validation](semantic-validation.md)
→ [Multi-Source Validation Overview](multi-source-validation.md)
