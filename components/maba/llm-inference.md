---
description: Language model reasoning for schemas
---

# LLM Semantic Inference

## LLM-Powered Schema Reasoning

LLM Semantic Inference is the second pillar of MABA's [Semantic Pattern Recognition](../../../product/platform/maba/ingestion-pipeline/semantic-pattern-recognition.md). For columns that can't be resolved by [distributional fingerprinting](distributional-fingerprinting.md) alone or [pattern library](../../../product/platform/maba/transformation-engine/pattern-library.md) matches, MABA uses frontier language models to reason about what data means.

---

### When LLM Inference Is Triggered

Not every column needs LLM analysis. The decision tree:

1. **Fingerprint resolves it?** (e.g., 100% datetime values → datetime column) → Skip LLM
2. **Pattern library has high-confidence match?** (cosine similarity > 0.92) → Skip LLM
3. **Neither resolves it?** → Invoke LLM inference

In practice, LLM inference is needed for 25-40% of columns in a typical migration — the ambiguous cases where statistics and historical patterns aren't sufficient.

---

### The Structured Prompt

For each unresolved column (or group of correlated columns), MABA constructs a prompt containing:

```yaml
Table: customer_data_v2_final
Column: ref_code
Declared type: NUMBER(4)
Statistical profile:
  - 45 distinct values
  - Range: 1001-3099
  - No nulls
  - Distribution: clustered in bands (1000s, 2000s, 3000s)
  - High mutual information with column "department"
Sample values (selected for diversity):
  1001, 1023, 2001, 2045, 3001
Column comment: (none)

Target schema:
  Table: transactions
  Fields: id (BIGINT), department_ref (INTEGER),
          category_code (VARCHAR), status (SMALLINT), ...

Tasks:
1. Identify the semantic category of this data
2. Determine the most likely target field mapping with confidence
3. Detect any implicit business logic in the data patterns
4. Identify PII or sensitive data requiring special handling
5. Recommend the transformation function
```

#### Sample Value Selection

The five sample values are not random. They're selected using a diversity maximization algorithm that ensures:

- At least one value from each distinct cluster (if clusters exist)
- At least one edge case (min, max, or boundary value)
- At least one "typical" value (near the mode)
- Representation of format variations (if the column has mixed formats)

This selection gives the LLM maximum information from minimum examples.

---

### The Five Analysis Tasks

#### 1. Semantic Category Classification

The LLM classifies the column into one of eight semantic categories:

- **Person:** Names, identifiers, contact information
- **Organization:** Company names, department codes, business identifiers
- **Location:** Addresses, coordinates, region codes
- **Temporal:** Dates, times, durations, epochs
- **Financial:** Currency amounts, account numbers, transaction identifiers
- **Identifier:** Primary keys, reference numbers, UUIDs
- **Status:** State codes, flags, enumerations
- **Descriptive:** Free-text, notes, comments

#### 2. Target Field Mapping

The LLM identifies the most likely target field and provides a confidence score (0.0-1.0). When multiple target fields are plausible, it ranks them with separate confidence scores.

#### 3. Business Logic Detection

The LLM identifies patterns that encode business logic:

- "First digit indicates department (1=Sales, 2=Engineering, 3=Support)"
- "Values above 9000 appear to be test/staging records"
- "Null values only appear for records with status=99 (soft deleted)"

These detections are surfaced to stakeholders via AMANI and stored in the migration knowledge base.

#### 4. PII Detection

The LLM flags columns that may contain personally identifiable information:

- Direct PII: names, SSNs, email addresses, phone numbers
- Indirect PII: combinations of fields that could identify individuals
- Encrypted PII: columns whose names or patterns suggest encrypted sensitive data

PII detection triggers additional handling requirements in the migration plan.

#### 5. Transformation Recommendation

The LLM recommends the specific transformation function:

- `CAST(ref_code AS INTEGER)` for simple type conversion
- `TO_TIMESTAMP(dt_mod, 'YYYY-MM-DD HH24:MI:SS') AT TIME ZONE 'UTC'` for datetime handling
- Custom Python function for complex business logic transformations

---

### Cost and Performance

| Model               | Latency per Column | Cost per Column   | Accuracy |
| ------------------- | ------------------ | ----------------- | -------- |
| GPT-4-turbo         | ~3 seconds         | ~$0.03            | 92-95%   |
| Claude-3 Opus       | ~4 seconds         | ~$0.04            | 91-94%   |
| Llama-3-70B (local) | ~2 seconds         | ~$0.005 (compute) | 85-90%   |

For a migration with 3,891 columns where 35% require LLM inference (~1,362 columns):

- **GPT-4-turbo:** ~68 minutes, ~$41
- **Claude-3 Opus:** ~91 minutes, ~$55
- **Llama-3-70B local:** ~45 minutes, ~$7

Parallelization across multiple LLM sessions reduces wall-clock time to 10-20 minutes regardless of provider.

---

### Feedback Loop

When human reviewers confirm or correct LLM-generated mappings via AMANI, the feedback flows back into the system:

1. Confirmed mappings are encoded as vectors and added to the pattern library
2. Corrections are used as few-shot examples for future LLM prompts
3. Systematic errors (the LLM consistently misclassifies a pattern) trigger prompt refinement

This feedback loop is why accuracy rises from 85-95% on first pass to 97%+ after incorporating human review — and why each successive migration requires less human review than the last.

→ [Distributional Fingerprinting](distributional-fingerprinting.md)
→ [Pattern Library](../../../product/platform/maba/transformation-engine/pattern-library.md)
→ [Semantic Pattern Recognition Overview](../../../product/platform/maba/ingestion-pipeline/semantic-pattern-recognition.md)
