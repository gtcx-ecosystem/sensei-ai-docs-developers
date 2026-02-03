---
description: Cryptographic proof of correctness
---

# Behavioral Equivalence Certification

## Cryptographic Migration Proof

When all validation perspectives pass, KORA generates a Behavioral Equivalence Certificate — a cryptographically signed document that constitutes independent, tamper-proof evidence of migration correctness.

---

### What a Certificate Contains

```yaml
BehavioralEquivalenceCertificate:
  certificate_id: BEC-2026-0215-00847
  issued_at: 2026-02-15T14:23:01Z

  migration:
    source: Oracle 19c (HR_PROD schema, 247 tables, 52M rows)
    target: PostgreSQL 16 (hr_production schema)
    started: 2026-02-14T22:00:00Z
    completed: 2026-02-15T16:15:00Z

  validation_results:
    structural:
      score: 1.000
      tables_checked: 247/247
      issues: 0

    statistical:
      score: 0.998
      columns_checked: 3,891/3,891
      variances: 7 (all within tolerance)

    referential:
      score: 1.000
      relationships_checked: 236/236
      orphan_records: 0

    semantic:
      score: 0.994
      rules_evaluated: 1,247
      violations: 8 (all reviewed and classified as acceptable)

    behavioral:
      score: 0.996
      checkpoints_tested: 7
      queries_replayed: 12,000
      exact_matches: 11,943
      semantic_matches: 52
      business_matches: 0
      divergences: 5 (all classified as acceptable variance)

  overall_confidence: 0.997

  coverage:
    table_coverage: 100%
    column_coverage: 100%
    record_coverage: 100%
    logic_path_coverage: 87%
    relationship_coverage: 100%

  hash_chain:
    source_checkpoint_root: 'sha256:a7f3c2...'
    target_reconstruction_root: 'sha256:b8e4d1...'
    validation_evidence_root: 'sha256:c9f5e2...'

  exceptions:
    - description: '5 floating-point divergences in SUM aggregations'
      severity: 'Acceptable variance'
      magnitude: '≤$0.02 per query'
      analysis: 'Accumulation order difference between Oracle and PostgreSQL'

  signature:
    algorithm: Ed25519
    public_key: 'kora-verification-2026-Q1'
    signature: 'sig:x7h9k2...'
```

---

### Cryptographic Properties

**Tamper-proof:** The certificate is cryptographically signed. Any modification — even a single character change — invalidates the signature.

**Hash-linked:** The certificate chains back to source checkpoint hashes and target reconstruction hashes. An auditor can verify the complete evidence chain from source system state through transformation to validation outcome.

**Non-repudiable:** The signing key is KORA's verification key, independent of the migration execution system. KORA attests that it independently verified the migration — not that the migration team claims it's correct.

---

### Regulatory Applicability

| Framework            | Requirement                                                                                  | How the Certificate Satisfies It                                      |
| -------------------- | -------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **GDPR Art. 32**     | "Ability to restore availability and access to personal data in a timely manner"             | Certificate proves data integrity preserved during migration          |
| **HIPAA §164.312**   | "Implement electronic mechanisms to corroborate that ePHI has not been altered or destroyed" | Hash chain provides cryptographic integrity verification              |
| **SOX Section 404**  | "Assessment of internal controls over financial reporting"                                   | Behavioral equivalence proves financial calculations preserved        |
| **Basel III**        | "Banks should have appropriate data architecture and IT infrastructure"                      | Certificate documents data quality controls during system changes     |
| **ISO 27001 A.12.3** | "Information backup — backup copies shall be tested"                                         | Time-Travel Testing provides functional verification of migrated data |

---

### Certificate Lifecycle

**Issuance:** Generated automatically when all validation perspectives pass their thresholds.

**Storage:** Stored in KORA's tamper-proof ledger with full evidence chain. A copy is provided to the customer.

**Audit access:** Third-party auditors can verify the certificate using KORA's public verification key and the evidence chain hashes. No access to customer data is required for verification.

**Retention:** Certificates are retained for the duration specified by the applicable regulatory framework (typically 5-7 years). Evidence data (checkpoint snapshots, replay results) can be archived to cold storage after certificate issuance.

---

### When Certification Fails

If validation results don't meet certification thresholds, KORA does not issue a certificate. Instead, it produces:

- A detailed failure report identifying every validation gap
- Prioritized remediation recommendations
- Estimated effort to reach certification
- Option for conditional certification with documented exceptions (if acceptable to the organization's compliance framework)

The migration team addresses the identified issues and resubmits for validation. KORA tracks remediation and re-validates efficiently, testing only the affected areas.

→ [How KORA Works](how-it-works.md)
→ [Time-Travel Testing](../../../product/platform/kora/time-travel-testing.md)
→ [KORA Overview](../../../product/platform/kora/README.md)
