# Statistical Analysis

## Phase 2: Profiling What the Data Looks Like

Statistical analysis is the second phase of MABA's [Schema Analysis Engine](../../../product/platform/maba/ingestion-pipeline/schema-analysis.md). It computes a comprehensive statistical profile for every column in the source system, producing the [distributional fingerprints](distributional-fingerprinting.md) that power pattern library matching and guide semantic inference.

---

### What Gets Computed

For every column in every table, MABA computes:

#### Completeness Metrics

- **Null rate:** Percentage of null/empty values
- **Fill rate:** Percentage of non-null values
- **Empty string rate:** For string columns, percentage that are technically non-null but contain no useful data

#### Uniqueness Metrics

- **Distinct count:** Number of unique values
- **Uniqueness ratio:** Distinct count / total rows
- **Duplicate distribution:** How many values appear 1×, 2×, 5×, 10×, 100×+

#### Value Distribution

- **Minimum and maximum:** For ordered types
- **Mean, median, mode:** For numeric types
- **Standard deviation, variance, IQR:** Dispersion measures
- **Skewness and kurtosis:** Distribution shape
- **Histogram:** Binned value distribution (20 bins)
- **Percentiles:** 1st, 5th, 25th, 50th, 75th, 95th, 99th

#### String-Specific Metrics

- **Length distribution:** Min, max, mean, std of character counts
- **Character class distribution:** Uppercase, lowercase, numeric, special character percentages
- **Pattern frequencies:** Top 10 most common value patterns (using regex generalization)
- **Encoding detection:** Detected character encoding (UTF-8, Latin-1, EBCDIC, etc.)

#### Format Detection

Percentage of values matching common formats:

- Email addresses, phone numbers, URLs
- Date/time formats (ISO, US, European, Asian, Unix epoch)
- Currency amounts, postal codes, IP addresses
- UUIDs, SSNs, national ID formats
- Custom patterns detected through regex inference

#### Cross-Column Analysis

- **Mutual information:** Pairwise statistical dependency between all columns in the same table
- **Correlation coefficients:** Pearson for numeric pairs, Cramér's V for categorical pairs
- **Functional dependency detection:** Columns where one deterministically determines another
- **Composite key identification:** Column combinations that are jointly unique

---

### Sampling Strategy

Computing statistics on 52 million rows per column would be prohibitively slow. MABA uses stratified sampling:

- **Sample size:** 10,000 rows per table (configurable)
- **Strategy:** Stratified by the first column's value range to avoid bias from ordered inserts
- **Confidence:** For most metrics, 10,000 rows provides 99% confidence within ±2% margin
- **Exact computation:** For critical metrics (distinct count, null rate on small tables), exact counts are used when the table has fewer than 100,000 rows

---

### Distributional Fingerprint Generation

The statistical profile is compressed into a [128-dimensional fingerprint vector](distributional-fingerprinting.md):

```text
Raw statistics (200+ values per column)
    ↓ dimensionality reduction
Fingerprint vector (128 dimensions)
    ↓ stored in
Pattern Library (Pinecone/Weaviate) for similarity search
```

The fingerprint preserves the statistical properties most relevant to semantic classification while discarding noise. Two columns with the same data type, similar distributions, and matching format patterns will have nearly identical fingerprints — regardless of their names or the database they came from.

---

### Performance

| Metric                            | Value                         |
| --------------------------------- | ----------------------------- |
| Columns profiled per hour         | 100,000+                      |
| Sample size per column            | 10,000 rows                   |
| Computation per column            | ~50ms (single core)           |
| Parallelized (4-node Ray cluster) | ~15 minutes for 3,891 columns |
| Memory per column                 | ~2 MB (sample + statistics)   |

Statistical analysis is embarrassingly parallel — each column can be profiled independently. Ray distributes columns across workers for linear speedup.

→ [Semantic Analysis](semantic-analysis.md)
→ [Distributional Fingerprinting](distributional-fingerprinting.md)
→ [Schema Analysis Overview](../../../product/platform/maba/ingestion-pipeline/schema-analysis.md)
