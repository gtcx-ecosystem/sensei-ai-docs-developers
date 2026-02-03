# Semantic Analysis

## Phase 3: Understanding What the Data Means

Semantic analysis is the third and most innovative phase of MABA's [Schema Analysis Engine](../../../product/platform/maba/ingestion-pipeline/schema-analysis.md). It's where statistical fingerprints and structural metadata become **meaning** — enabling MABA to map source columns to target schemas based on what data represents, not just how it's stored.

---

### The Semantic Gap

Consider two columns:

- `EMP_DOB` in an Oracle HR database: `DATE` type, values range from 1945 to 2003, 100% non-null
- `GBDAT` in an SAP HCM system: `DATS` type (8-digit integer), values range from 19450101 to 20031231, 100% non-null

Structurally, these are completely different — different types, different formats, different names. Statistically, their distributions are nearly identical: similar ranges, similar null rates, similar uniqueness patterns.

**Semantically, they're the same thing:** employee date of birth.

Bridging this gap — recognizing that structurally different columns carry the same meaning — is what semantic analysis does. No rule-based system can do this. It requires the kind of reasoning that large language models provide.

---

### Three-Step Process

#### Step 1: Pattern Library Lookup

Each column's [distributional fingerprint](distributional-fingerprinting.md) is searched against the [Pattern Library](../../../product/platform/maba/transformation-engine/pattern-library.md) for similar prior mappings.

- **High-confidence match (>0.92):** Mapping applied directly. No LLM call needed. This resolves 55-85% of columns at platform maturity.
- **Medium-confidence match (0.75-0.92):** Used as a prior that biases LLM inference toward the likely-correct mapping.
- **No useful match (<0.75):** Full LLM inference required.

#### Step 2: LLM Inference

For unresolved columns, MABA constructs a [structured prompt](llm-inference.md) and sends it to a frontier language model. The LLM performs five analysis tasks:

1. Semantic category classification (person, organization, location, temporal, financial, identifier, status, descriptive)
2. Target field mapping with confidence score
3. Implicit business logic detection
4. PII/sensitive data identification
5. Transformation function recommendation

#### Step 3: Confidence Scoring and Routing

Every mapping — whether from the pattern library or LLM inference — receives a confidence score that determines its handling:

| Confidence      | Action               | Human Involvement                                 |
| --------------- | -------------------- | ------------------------------------------------- |
| **> 0.95**      | Auto-apply           | None (notified post-migration)                    |
| **0.80 - 0.95** | Apply with flag      | Review optional, recommended for critical columns |
| **< 0.80**      | Require confirmation | Must be approved before migration proceeds        |

---

### Business Logic Discovery

Semantic analysis doesn't just map columns — it discovers business rules encoded in data patterns:

**Status codes:** "Column `status` uses values 1-10 for active states and 99 for soft deletion. This convention is confirmed by stored procedure `proc_cleanup_customers` which filters on `status != 99`."

**Composite codes:** "Column `ref_code` encodes department hierarchy: first digit = division (1=Sales, 2=Engineering, 3=Support), remaining digits = sequential within division."

**Temporal patterns:** "Column `last_login` has a bimodal distribution — 70% of values cluster in the last 90 days and 30% are exactly `1970-01-01`, which likely represents 'never logged in' rather than a literal 1970 date."

These discoveries are:

1. Surfaced to stakeholders via AMANI for confirmation
2. Incorporated into transformation logic (e.g., converting Unix epoch `0` to `NULL` instead of `1970-01-01`)
3. Stored in the knowledge base for future migrations with similar patterns

---

### PII Detection

Semantic analysis identifies personally identifiable information that requires special handling during migration:

| PII Type                                                  | Detection Method                             | Migration Implication                         |
| --------------------------------------------------------- | -------------------------------------------- | --------------------------------------------- |
| **Direct PII** (names, SSNs, emails)                      | Pattern matching + LLM classification        | Encryption, masking, or tokenization required |
| **Indirect PII** (combinations that identify individuals) | Cross-column correlation analysis            | Column groups flagged for review              |
| **Encrypted PII** (columns containing encrypted values)   | Entropy analysis + naming patterns           | Encryption key migration required             |
| **Derived PII** (calculated from PII sources)             | Lineage tracing through views and procedures | Same handling as source PII                   |

PII detection triggers compliance workflows in the migration plan — KORA validates that PII handling meets GDPR, HIPAA, or other applicable regulatory requirements.

---

### Accuracy Progression

| Migration Count   | Library Hit Rate | LLM Required | Overall Accuracy |
| ----------------- | ---------------- | ------------ | ---------------- |
| 1st migration     | 0%               | 100%         | 85-90%           |
| 10th migration    | 25%              | 75%          | 90-93%           |
| 100th migration   | 55%              | 45%          | 93-96%           |
| 1,000th migration | 73%              | 27%          | 96-98%           |

The improvement comes from two sources: the growing pattern library reduces reliance on LLM inference (which is less accurate than validated library matches), and the LLM itself benefits from better priors derived from library matches in the medium-confidence range.

→ [Semantic Pattern Recognition](../../../product/platform/maba/ingestion-pipeline/semantic-pattern-recognition.md)
→ [LLM Inference](llm-inference.md)
→ [Schema Analysis Overview](../../../product/platform/maba/ingestion-pipeline/schema-analysis.md)
