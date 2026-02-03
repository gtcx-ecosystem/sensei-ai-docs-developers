# How MABA Works

## End-to-End: From Raw Data to Transformed Records

This page walks through MABA's complete pipeline — from connecting to a source system to producing validated, transformed records ready for the target.

> For platform overview and business context, see [MABA Overview](../../../product/platform/maba/README.md).

---

### The Pipeline

```text
Source System
    ↓
┌─────────────────────────────────┐
│ 1. INGESTION                     │
│    Format detection → Parsing    │
│    → Normalized IR               │
├─────────────────────────────────┤
│ 2. SCHEMA ANALYSIS               │
│    Structural → Statistical      │
│    → Semantic (LLM)              │
├─────────────────────────────────┤
│ 3. MAPPING                       │
│    Pattern library lookup        │
│    → LLM inference → Confidence  │
│    scoring → Human review        │
├─────────────────────────────────┤
│ 4. CODE GENERATION               │
│    SQL + Python transforms       │
│    → Unit testing → Validation   │
├─────────────────────────────────┤
│ 5. EXECUTION                     │
│    Ray distributed processing    │
│    → Streaming transform         │
│    → Output to target format     │
└─────────────────────────────────┘
    ↓
KORA Validation → Target System
```

---

### Step 1: Ingestion

MABA connects to the source and reads data. For databases, this means connecting via native drivers and extracting both data and metadata. For files, this means detecting the format (even when extensions are wrong) and parsing into a normalized intermediate representation.

**What happens:**

- Format detection using magic bytes, extension analysis, and statistical content analysis
- Parsing through format-specific adapters (50+ supported)
- Normalization into an intermediate representation (IR) that captures data values, inferred types, null rates, cardinality, value distributions, and encoding information

**Output:** Normalized IR with rich metadata for every column in every table.

→ [Ingestion Details](ingestion.md)

---

### Step 2: Schema Analysis

MABA analyzes the source schema in three phases, each building on the previous:

**Structural Phase:** Extracts table definitions, column types, constraints, indexes, foreign keys, stored procedures, triggers, and views from the database catalog. This is what every ETL tool does.

**Statistical Phase:** Profiles every column — null rates, distinct value counts, min/max, mean/median/mode, length distributions, pattern frequencies, and cross-column correlations. This creates the distributional fingerprint that powers semantic analysis.

**Semantic Phase:** Uses frontier LLMs to infer the meaning of each column from the combined evidence of structure, statistics, sample values, and any available documentation. This is MABA's patent-pending innovation — the step that turns "cust_ref VARCHAR(15)" into "customer reference number, sequential, links to orders table."

→ [Schema Analysis Details](../../../product/platform/maba/ingestion-pipeline/schema-analysis.md)

---

### Step 3: Mapping

With semantic understanding of both source and target schemas, MABA generates mappings:

**Pattern library first:** Before invoking expensive LLM inference, MABA queries the pattern library for similar prior mappings. High-confidence matches (cosine similarity > 0.92) are applied directly. This dramatically reduces LLM costs for common patterns.

**LLM inference for novel cases:** Columns without high-confidence library matches get individual LLM analysis. The model receives the structural metadata, statistical profile, sample values, and target schema definition, then produces a mapping with confidence score.

**Confidence thresholds:**

- **> 0.95:** Auto-apply without human review
- **0.80 - 0.95:** Apply with human notification (flagged for review)
- **< 0.80:** Require human confirmation before proceeding

**Human-in-the-loop:** AMANI presents low-confidence mappings to the appropriate stakeholder for confirmation. The confirmed mappings feed back into the pattern library, improving future accuracy.

→ [Semantic Pattern Recognition](../../../product/platform/maba/ingestion-pipeline/semantic-pattern-recognition.md)
→ [Pattern Library](../../../product/platform/maba/transformation-engine/pattern-library.md)

---

### Step 4: Code Generation

For every confirmed mapping, MABA generates executable transformation code:

- **SQL transformations** for database-to-database migrations (executed at the source or target)
- **Python transformations** for complex logic, cross-table operations, and non-SQL sources
- **Unit tests** that verify each transformation against sample data

Generated code handles type conversions, encoding normalization, value transformations, structural transformations, and data quality remediation. Every transformation is tested before inclusion in the migration plan.

→ [Transformation Code Generation](transformation-codegen.md)

---

### Step 5: Execution

MABA executes the transformation pipeline using Ray for distributed processing:

- **Ingestion actors** read from the source in parallel
- **Transformation actors** apply mapping and transformation logic
- **Emission actors** write transformed records to the target format

Each stage auto-scales independently based on backpressure. A bottleneck in transformation spawns more transformation actors without affecting ingestion or emission.

Records flow through the pipeline continuously — MABA doesn't read everything into memory before transforming. This streaming architecture enables processing of datasets far larger than available RAM.

→ [Performance Architecture](../../../product/platform/maba/performance.md)

---

### Example: Real Migration Walkthrough

**Source:** Oracle 19c, 247 tables, ~52M rows
**Target:** PostgreSQL 16

1. **Ingestion** (2 minutes): MABA connects to Oracle, extracts metadata for 247 tables and 3,891 columns, samples 100 rows per table
2. **Structural analysis** (30 seconds): Foreign keys, constraints, indexes catalogued
3. **Statistical profiling** (15 minutes): Distributional fingerprints computed for all 3,891 columns
4. **Pattern library lookup** (5 seconds): 2,847 columns match known patterns (73% coverage)
5. **LLM semantic inference** (20 minutes): Remaining 1,044 columns analyzed individually
6. **Mapping review**: 3,712 mappings auto-approved (>0.95 confidence), 147 flagged for review, 32 require human confirmation
7. **Code generation** (10 minutes): 187 transformation functions generated, 187 unit tests pass
8. **Execution** (18-24 hours): 52M records transformed and loaded via 16 Ray workers scaling to 64

**Total schema analysis time:** ~40 minutes
**Traditional manual equivalent:** 3-6 weeks of senior analyst time
