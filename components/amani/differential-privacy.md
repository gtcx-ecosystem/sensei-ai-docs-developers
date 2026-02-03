# Differential Privacy

## Mathematical Guarantees, Not Policy Promises

Differential privacy is the second pillar of AMANI's [privacy-preserving learning](../../../product/platform/amani/privacy-preserving-learning.md). It provides mathematical guarantees that no individual customer's participation in cross-migration learning can be detected or exploited by an adversary.

---

### The Guarantee

**Epsilon-differential privacy** ensures that the output of the learning system is statistically indistinguishable whether or not any single customer's data was included.

Formally: For any two datasets D and D' that differ in one customer's data, and for any output O:

```text
P(output = O | D) ≤ e^ε × P(output = O | D')
```

Where ε (epsilon) is the privacy parameter. A smaller epsilon means stronger privacy.

**Practical meaning:** An adversary who knows everything about the learning output cannot determine with high confidence whether a specific customer participated. Their best guess is barely better than random.

---

### How It Works

Each pattern vector transmitted from a customer's environment is perturbed by calibrated Gaussian noise:

```text
transmitted_pattern = true_pattern + N(0, σ²)
```

Where σ (sigma) is determined by:

- The **sensitivity** of the pattern extraction function (how much could one customer's data change the output)
- The **target privacy budget** (ε)

Higher sensitivity or lower ε requires more noise. The noise magnitude is computed using the Gaussian mechanism from differential privacy theory.

---

### Privacy Budget (Epsilon)

Sensei uses a default privacy budget of **ε = 1.0**, which provides strong guarantees:

| Epsilon               | Adversary Advantage | Practical Meaning                                  |
| --------------------- | ------------------- | -------------------------------------------------- |
| ε = 0.1               | Factor of 1.1       | Extremely strong — adversary learns almost nothing |
| ε = 0.5               | Factor of 1.6       | Very strong — participation nearly undetectable    |
| **ε = 1.0** (default) | Factor of 2.7       | Strong — standard for sensitive data               |
| ε = 2.0               | Factor of 7.4       | Moderate — detectable but not exploitable          |
| ε = 5.0               | Factor of 148       | Weak — not recommended for sensitive applications  |

Customers requiring stronger guarantees can configure lower epsilon values (ε = 0.5 or ε = 0.1) at the cost of reduced learning contribution.

---

### Noise Calibration

The noise added depends on the pattern type:

| Pattern Category       | Sensitivity | Noise Level | Impact on Learning                                       |
| ---------------------- | ----------- | ----------- | -------------------------------------------------------- |
| Schema fingerprints    | Low         | Low         | Minimal — structure is coarse-grained                    |
| Performance profiles   | Low         | Low         | Minimal — throughput curves are similar across customers |
| Error signatures       | Medium      | Medium      | Moderate — error distributions vary more                 |
| Strategy effectiveness | Low         | Low         | Minimal — effectiveness scores are normalized            |

The noise is calibrated to preserve utility while guaranteeing privacy. Even with ε = 1.0 noise, pattern similarity search remains 95%+ accurate for common patterns.

---

### Composition

Privacy degrades with repeated queries or contributions. If a customer contributes patterns from 10 migrations, each with ε = 1.0, the total privacy budget consumed is approximately √10 × 1.0 ≈ 3.16 (under advanced composition theorems).

AMANI's [privacy budget accounting](privacy-budget.md) tracks cumulative epsilon expenditure and automatically stops contributions when the customer's lifetime budget is exhausted.

---

### What This Means Practically

**Scenario:** An adversary wants to determine whether Acme Corp used Sensei for a migration.

**Without differential privacy:**
The adversary could potentially:

- Look for patterns that match Acme's known schema characteristics
- Identify timing correlations between Acme's migration and pattern library updates
- Use membership inference attacks on the pattern library

**With differential privacy (ε = 1.0):**

- Patterns are noised so that Acme-specific characteristics are statistically invisible
- The adversary's best guess is at most 2.7× better than random guessing
- Given thousands of customers, correctly identifying Acme is effectively impossible

---

### Verification

Customers can verify that differential privacy is correctly applied:

1. **Noise injection is auditable:** The pattern extraction agent logs noise magnitudes (not the noised values themselves)
2. **Budget tracking is transparent:** Customers can see their cumulative epsilon expenditure
3. **Third-party audits:** External auditors can verify the DP implementation against the claimed privacy parameters

→ [Privacy Budget Accounting](privacy-budget.md)
→ [Privacy-Preserving Learning Overview](../../../product/platform/amani/privacy-preserving-learning.md)
