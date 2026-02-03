# Privacy Budget Accounting

## Tracking Every Bit of Information Disclosure

Privacy budget accounting is the fourth pillar of AMANI's [privacy-preserving learning](../../../product/platform/amani/privacy-preserving-learning.md). It maintains a tamper-proof ledger of all information flows, ensuring that cumulative privacy guarantees are never exceeded.

---

### Why Budgets Matter

[Differential privacy](differential-privacy.md) provides per-operation guarantees (ε per contribution). But privacy **composes** — many small disclosures add up to larger disclosure. Without careful tracking:

- A customer contributing to 100 migrations might exhaust their privacy budget
- Repeated queries about similar patterns could leak information through correlation
- Adversaries could exploit accumulated disclosures across time

Privacy budget accounting prevents this by treating privacy as a finite, depletable resource.

---

### The Ledger

AMANI maintains a **tamper-proof privacy ledger** for each customer:

```yaml
Customer: [customer_id_hash]
Budget:
  initial: 10.0 # Total lifetime epsilon
  consumed: 3.47 # Epsilon spent to date
  remaining: 6.53

Transactions:
  - timestamp: 2026-01-15T08:23:00Z
    operation: pattern_contribution
    epsilon_spent: 0.85
    pattern_dimensions: 128
    noise_magnitude: 0.73
    commitment: sha256:a7f3c2...

  - timestamp: 2026-01-22T14:15:00Z
    operation: pattern_contribution
    epsilon_spent: 0.92
    pattern_dimensions: 128
    noise_magnitude: 0.68
    commitment: sha256:b8e4d1...

  # ... additional transactions
```

Each transaction records:

- When the disclosure occurred
- What type of operation
- How much epsilon was spent
- The noise applied (verifiable against the commitment)
- A cryptographic commitment to the pre-noise pattern (for audit verification)

---

### Automatic Budget Enforcement

When a customer's budget approaches exhaustion:

| Remaining Budget | System Behavior                                                  |
| ---------------- | ---------------------------------------------------------------- |
| >50%             | Normal operation                                                 |
| 25-50%           | Warnings shown in AMANI interface                                |
| 10-25%           | Contribution frequency limited                                   |
| 1-10%            | Only high-value patterns contributed; user confirmation required |
| <1%              | Contributions paused; customer can still receive patterns        |
| 0%               | No contributions; receive-only mode until budget refreshed       |

Budget refresh is available (with customer consent) on an annual basis, resetting the cumulative epsilon while preserving the historical ledger.

---

### Composition Theorems

AMANI uses **advanced composition theorems** to compute cumulative epsilon:

**Basic composition:** k operations at ε each = k × ε total
**Advanced composition:** k operations at ε each ≈ √(2k × ln(1/δ)) × ε + k × ε(e^ε - 1)

For typical parameters (ε = 1.0, δ = 10⁻⁶):

- 10 contributions: ~3.5ε total (not 10ε)
- 100 contributions: ~12ε total (not 100ε)
- 1,000 contributions: ~40ε total (not 1,000ε)

Advanced composition dramatically extends how many operations are possible within a fixed budget.

---

### Customer Transparency

Customers can view their privacy budget status at any time:

```text
┌──────────────────────────────────────────────┐
│          Privacy Budget Dashboard             │
├──────────────────────────────────────────────┤
│                                               │
│  Lifetime Budget:     10.0 ε                 │
│  ████████░░░░░░░░░░░░ 34.7% consumed         │
│                                               │
│  Contributions:       23                      │
│  Last 30 days:        3                       │
│                                               │
│  At current rate:     ~4 years remaining     │
│                                               │
│  [View Transaction History]  [Export Audit]  │
│                                               │
└──────────────────────────────────────────────┘
```

---

### Third-Party Audit

The privacy ledger is designed for external audit:

1. **Immutable history:** Transactions cannot be modified or deleted
2. **Cryptographic verification:** Auditors can verify noise was correctly applied using the commitments
3. **Composition verification:** Auditors can independently compute cumulative epsilon
4. **Export format:** Standard format for privacy audit tools

This enables customers to provide auditable proof to regulators that their participation in cross-migration learning complies with data protection requirements.

---

### Budget for Queries vs. Contributions

Both querying the pattern library and contributing patterns consume privacy budget, but at different rates:

| Operation                         | Epsilon Cost | Rationale                                      |
| --------------------------------- | ------------ | ---------------------------------------------- |
| Pattern contribution              | 0.5 - 1.5 ε  | Reveals information about customer's migration |
| Pattern query (homomorphic)       | 0.0 ε        | Encrypted query reveals nothing                |
| Pattern query (metadata-filtered) | 0.01 - 0.1 ε | Metadata filtering reveals coarse information  |

Homomorphic queries are essentially "free" from a privacy perspective — customers can query as often as needed. Metadata filtering (used for performance) consumes minimal budget.

→ [Differential Privacy](differential-privacy.md)
→ [Privacy-Preserving Learning Overview](../../../product/platform/amani/privacy-preserving-learning.md)
