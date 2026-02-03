---
description: Data distribution validation
---

# Statistical Validation

## Do the Numbers Still Add Up?

Statistical validation compares column-level statistical profiles between source and target. If the source column has a mean of 47.3 and the target's mean is 47.3 ± epsilon, the transformation preserved the distribution. If the mean shifted to 23.1, something went systematically wrong.

---

### What Gets Compared

For every column pair (source → target), KORA compares:

| Metric                            | Tolerance                                                   | Failure Indicates                |
| --------------------------------- | ----------------------------------------------------------- | -------------------------------- |
| **Row count**                     | Exact match (or planned difference for filtered migrations) | Missing or duplicated records    |
| **Null count**                    | Exact match ± planned changes                               | Null introduction or elimination |
| **Distinct count**                | Exact match ± planned changes (dedup, merge)                | Value collapse or split          |
| **Min / Max**                     | Type-appropriate epsilon                                    | Range truncation or overflow     |
| **Mean**                          | ≤ 0.01% relative difference for numerics                    | Systematic transformation error  |
| **Median**                        | ≤ 0.1% relative difference                                  | Distribution shift               |
| **Standard deviation**            | ≤ 1% relative difference                                    | Variance change (precision loss) |
| **Distribution shape**            | Kolmogorov-Smirnov test, p > 0.05                           | Fundamental distribution change  |
| **Length distribution** (strings) | Mean length ± 5%                                            | Truncation or padding            |
| **Pattern frequencies**           | Top-10 patterns match within 1%                             | Format transformation errors     |

---

### What It Catches

Statistical validation excels at detecting **systematic** errors — problems that affect many records in a consistent way:

- **Silent truncation:** A `VARCHAR(50)` column truncated to `VARCHAR(30)` — row counts match, but string length distribution shifts
- **Precision loss:** `DECIMAL(10,4)` rounded to `DECIMAL(10,2)` — means shift slightly, standard deviation decreases
- **Encoding corruption:** UTF-8 characters mangled during Latin-1 conversion — string length distribution changes due to multi-byte character corruption
- **Off-by-one errors:** Date transformations that consistently add or subtract a day — min/max shift by exactly 1
- **Null injection:** A default value that converts empty strings to NULL — null count increases

---

### What It Can't Catch

Statistical validation misses errors that don't change aggregate statistics:

- **Swapped records:** Two rows with exchanged values — all column statistics remain identical
- **Individual errors:** A single wrong value in 10M correct values — statistically invisible
- **Semantic errors:** Correct distributions but wrong business meaning (values assigned to wrong entities)

These are caught by referential, semantic, and behavioral validation.

---

### Configurable Tolerances

Different migrations require different tolerance levels:

| Scenario             | Tolerance Level                                       | Use Case                                                  |
| -------------------- | ----------------------------------------------------- | --------------------------------------------------------- |
| **Strict**           | Exact match for all metrics                           | Financial data, regulatory compliance                     |
| **Normal** (default) | Small epsilon for floating-point, exact for integers  | General enterprise migration                              |
| **Relaxed**          | Larger tolerances for means, allow distribution shift | Migrations with intentional aggregation or transformation |
| **Custom**           | Per-column tolerance overrides                        | When specific columns are known to change                 |

---

### Performance

| Metric                     | Value                                          |
| -------------------------- | ---------------------------------------------- |
| Throughput                 | 100,000 columns/hour                           |
| Accuracy                   | 99.9%                                          |
| Resource cost              | Low (CPU + memory for statistical computation) |
| Duration for 3,891 columns | ~2.3 minutes                                   |

→ [Referential Validation](referential-validation.md)
→ [Multi-Source Validation Overview](multi-source-validation.md)
