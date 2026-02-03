# Distributional Fingerprinting

## A Statistical Portrait of Every Column

Distributional fingerprinting is the first pillar of MABA's [Semantic Pattern Recognition](../../../product/platform/maba/ingestion-pipeline/semantic-pattern-recognition.md). Every source column is encoded as a fixed-dimensional vector capturing its statistical properties — a compact "portrait" that enables rapid similarity matching and pre-filters before expensive LLM inference.

---

### What a Fingerprint Contains

Each fingerprint is a vector with the following dimensions:

#### Type Distribution

What percentage of values in the column parse as each basic type:

- Integer, float, date/datetime, boolean, string
- Example: A column where 98% of values parse as datetime and 2% are null → strong datetime signal

#### Value Range and Central Tendency

- Minimum, maximum, mean, median, mode
- Standard deviation and interquartile range
- Skewness and kurtosis (shape of the distribution)

#### Null Rate and Uniqueness

- Percentage of null/empty values
- Distinct value count and ratio (distinct/total)
- A column with 100% unique values is likely an identifier; a column with 5 distinct values is likely a status code

#### Character-Level Patterns

Percentage of values matching common formats:

- Email addresses (`*@*.*`)
- Phone numbers (various regional formats)
- URLs (`http*://`)
- Dates (ISO, US, European, Asian formats)
- Currency values (`$`, `€`, `£` prefixed numbers)
- Postal/ZIP codes (regional formats)
- IP addresses (v4, v6)
- UUIDs

#### Cross-Column Relationships

- Mutual information with every other column in the same table
- Identifies likely foreign key relationships even when no formal FK constraint exists
- Detects composite keys (columns that are unique only in combination)

---

### How Fingerprints Are Used

#### 1. Pre-Filtering for LLM Inference

LLM calls are expensive ($0.01-0.10 per column analysis). Fingerprints enable MABA to skip LLM calls for obvious cases:

- Column with 100% datetime values, name containing "date" or "dt" → datetime field, no LLM needed
- Column with exactly 2 distinct values (0 and 1) → boolean, no LLM needed
- Column with values matching email regex in 97% of rows → email address, no LLM needed

For a typical migration with 3,891 columns, fingerprint-based pre-filtering resolves 40-60% of columns without any LLM calls.

#### 2. Pattern Library Matching

Fingerprints are the primary key for pattern library similarity search. When MABA encounters a new column, its fingerprint is compared against all stored patterns using cosine similarity. High-similarity matches provide validated mappings from prior migrations.

#### 3. Clustering Related Columns

Fingerprint similarity within a table identifies groups of related columns that should be analyzed together. A set of columns with highly correlated fingerprints (e.g., `addr_line1`, `addr_line2`, `city`, `state`, `zip`) are grouped and presented to the LLM as a unit, improving semantic inference accuracy.

---

### Computation

Fingerprints are computed during the Statistical Phase of schema analysis:

- **Sample size:** 10,000 rows per column (or full table if smaller)
- **Computation time:** ~50ms per column on a single CPU core
- **Total for a typical migration (3,891 columns):** ~3 minutes
- **Parallelized across Ray workers:** Under 30 seconds

The computation is lightweight by design — fingerprints need to be fast because they're computed for every column, while expensive LLM inference is reserved for ambiguous cases identified by fingerprint analysis.

---

### Fingerprint Vector Format

The fingerprint is encoded as a fixed-dimensional vector (128 dimensions by default) suitable for efficient similarity search in Pinecone/Weaviate:

```text
Dimensions 0-15:   Type distribution (integer, float, date, boolean, string, mixed)
Dimensions 16-31:  Value range normalized to [0,1] for each detected type
Dimensions 32-47:  Central tendency metrics
Dimensions 48-63:  Null and uniqueness characteristics
Dimensions 64-95:  Character pattern match rates (16 common patterns)
Dimensions 96-127: Cross-column relationship strengths (top 32 correlations)
```

The fixed dimensionality enables hardware-accelerated similarity search across millions of stored patterns.

→ [LLM Semantic Inference](llm-inference.md)
→ [Pattern Library](../../../product/platform/maba/transformation-engine/pattern-library.md)
→ [Semantic Pattern Recognition Overview](../../../product/platform/maba/ingestion-pipeline/semantic-pattern-recognition.md)
