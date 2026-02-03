# Semantic Validation

## Does the Data Make Business Sense?

Semantic validation uses AI to verify that migrated data isn't just structurally correct but **business-correct** — values that are syntactically valid but semantically implausible are flagged for review.

---

### The Gap Between Syntax and Semantics

A date of birth of `2025-01-15` is:

- **Structurally valid:** It's a proper DATE type
- **Statistically plausible:** It falls within the column's observed range
- **Referentially correct:** The parent customer record exists
- **Semantically wrong:** A customer record created in 2020 can't have a 2025 date of birth

No rule-based validation catches this unless someone explicitly wrote a check for it. Semantic validation uses fine-tuned BERT models and LLM analysis to catch business logic violations that aren't encoded in schema constraints.

---

### What Gets Checked

#### Business Rule Inference

KORA infers business rules from source system behavior and validates them in the target:

- **Temporal plausibility:** Hire date must precede termination date. Birth date must precede hire date. Created-at must precede modified-at.
- **Value plausibility:** Salary of $0.01 passes type checking but fails business sense. Quantity of -5 may be valid for returns but impossible for initial orders.
- **Cross-field consistency:** If country = "United States" then ZIP code should match US format. If currency = "JPY" then amount should not have decimal places.
- **Status transitions:** If current status is "shipped" then previous status should have been "processing" or "confirmed", not "cancelled."

#### Configuration Table Validation

Values referencing lookup tables (status codes, category codes, region codes) are validated against the migrated lookup data:

- Every status value in `orders.status` must exist in `status_codes.code`
- Every region in `customers.region` must exist in `regions.region_id`

#### Implicit Constraint Validation

Constraints that exist in stored procedures or application code but not in schema:

- A trigger that enforces `email LIKE '%@%.%'` → validate all email values match
- A stored procedure that rejects orders where `quantity > 10000` → validate no orders exceed limit
- An application check that prevents `effective_date < SYSDATE - 365` → validate date range

---

### How It Works

Semantic validation operates in two layers:

**Layer 1: Rule-based (fast, deterministic)**
Known business rules extracted during Scout Agent discovery are encoded as validation rules in Great Expectations format. These run at 100,000+ rules/hour with 100% accuracy for the rules they cover.

**Layer 2: AI-driven (slower, catches novel issues)**
A fine-tuned BERT model scores records for semantic plausibility. Records with low plausibility scores are flagged and optionally escalated to a frontier LLM for detailed analysis.

The AI layer catches issues that no one thought to write rules for — because the business rules were never documented, or because the violation is a novel combination that wasn't anticipated.

---

### Accuracy and False Positives

| Metric                                   | Value               |
| ---------------------------------------- | ------------------- |
| True positive rate (catches real issues) | 94-98%              |
| False positive rate                      | 2-6%                |
| Throughput (rule-based)                  | 100,000+ rules/hour |
| Throughput (AI-driven)                   | 10,000 rules/hour   |

The false positive rate means KORA occasionally flags values that are unusual but correct. This is intentional — it's better to review a few false alarms than to miss genuine semantic corruption. False positives are resolved during human review and fed back into the validation model to reduce future noise.

---

### Feedback Loop

Every semantic validation outcome improves future validations:

- **Confirmed violations** become new validation rules added to the library
- **False positives** refine the AI model's plausibility thresholds
- **Novel rules discovered** are generalized and shared across migrations (privacy-preserving)

→ [Behavioral Validation (Time-Travel Testing)](../../../product/platform/kora/time-travel-testing.md)
→ [Multi-Source Validation Overview](multi-source-validation.md)
