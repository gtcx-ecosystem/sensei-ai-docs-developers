# Homomorphic Pattern Matching

## Searching Without Revealing What You're Looking For

Homomorphic pattern matching is the third pillar of AMANI's [privacy-preserving learning](../../../product/platform/amani/privacy-preserving-learning.md). It enables the central knowledge base to find relevant patterns for a customer's migration without ever seeing what that migration looks like.

---

### The Problem

When a new migration begins, the system needs to retrieve relevant patterns from prior migrations. But the query itself could leak information:

- "Find patterns similar to [my schema fingerprint]" reveals the structure of my schema
- Even encrypted queries could be compared against known patterns by the server
- The results returned could reveal what the server learned about the query

Homomorphic encryption solves this by enabling computation on encrypted data.

---

### How It Works

**Step 1: Query Encryption**
The customer's pattern extraction agent encrypts the query vector using the customer's public key before sending it to the central knowledge base.

```text
query_encrypted = Encrypt(query_vector, customer_public_key)
```

**Step 2: Encrypted Computation**
The central knowledge base performs similarity matching on the encrypted query. Using partially homomorphic encryption, the server computes:

```text
similarity_encrypted = dot_product(query_encrypted, pattern_vectors)
```

This works because certain encryption schemes allow arithmetic operations on encrypted values. The server never decrypts the query — it operates entirely on ciphertext.

**Step 3: Encrypted Results**
The results (similarity scores) are returned encrypted:

```text
results_encrypted = {pattern_1: score_enc_1, pattern_2: score_enc_2, ...}
```

**Step 4: Client Decryption**
Only the customer, holding the private key, can decrypt the results:

```text
results = Decrypt(results_encrypted, customer_private_key)
```

The customer sees which patterns are most relevant to their migration. The server never sees the query or the results.

---

### What's Protected

| Information      | Without Homomorphic Encryption          | With Homomorphic Encryption                     |
| ---------------- | --------------------------------------- | ----------------------------------------------- |
| Query content    | Server sees the query                   | Server sees encrypted blob                      |
| Query structure  | Server can infer schema characteristics | Server learns nothing about structure           |
| Result relevance | Server knows which patterns matched     | Server can't interpret match scores             |
| Query frequency  | Server can track query patterns         | Server can limit queries but not interpret them |

---

### Performance

Homomorphic encryption is computationally expensive. AMANI uses **partially homomorphic encryption** (PHE) rather than fully homomorphic encryption (FHE) to maintain practical performance:

| Operation                       | Unencrypted | PHE Encrypted | Overhead |
| ------------------------------- | ----------- | ------------- | -------- |
| Single similarity computation   | ~1μs        | ~10ms         | 10,000×  |
| Top-10 pattern retrieval        | ~1ms        | ~100ms        | 100×     |
| Full library scan (1M patterns) | ~1s         | Not practical | N/A      |

To make encrypted search practical, AMANI uses a two-stage approach:

1. **Coarse filtering:** Unencrypted metadata (migration type, approximate complexity range) narrows the search space
2. **Fine matching:** Homomorphic similarity search on the narrowed candidate set (~1,000 patterns)

This hybrid approach achieves <500ms query latency while maintaining full privacy for the pattern content.

---

### Cryptographic Foundation

AMANI uses the **Paillier cryptosystem** for homomorphic pattern matching:

- **Additively homomorphic:** Supports addition and scalar multiplication on encrypted values
- **Semantic security:** Ciphertexts reveal nothing about plaintexts
- **Efficient:** Much faster than fully homomorphic schemes like BGV or CKKS
- **Well-studied:** 25+ years of cryptographic analysis with no known practical attacks

The dot product computation (similarity = Σ query[i] × pattern[i]) is implemented using Paillier's additive homomorphism:

```text
Encrypt(a) × Encrypt(b) = Encrypt(a + b)
Encrypt(a)^k = Encrypt(a × k)
```

---

### Trust Model

With homomorphic matching, customers don't need to trust that Sensei's servers are secure. Even if an adversary:

- Compromises the central knowledge base
- Intercepts all queries
- Has unlimited computational resources

They still cannot determine:

- What any customer searched for
- Which results were relevant to which customer
- The content of any customer's schema or migration

The security is mathematical, not operational.

→ [Privacy Budget Accounting](privacy-budget.md)
→ [Privacy-Preserving Learning Overview](../../../product/platform/amani/privacy-preserving-learning.md)
