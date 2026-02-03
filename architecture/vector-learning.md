# Cross-Migration Vector Learning

## Starting Every Migration with the Wisdom of All Prior Migrations

Vector learning is the second of Sensei's three learning loops. It operates across migrations, ensuring that patterns discovered during one customer's migration improve planning and execution for all future migrations with similar profiles.

---

### How It Works

After each migration completes, Meta Agents extract abstract patterns from the execution log and encode them as high-dimensional vectors in a persistent knowledge base.

```yaml
Migration #847 completes: Oracle HR → PostgreSQL HR
Meta Agent extracts:
  - Schema pattern: {tables: 47, avg_columns: 12, fk_density: 0.8,
                     dominant_types: [VARCHAR, NUMBER, DATE]}
  - Transform pattern: {date_conversions: 23, encoding_fixes: 7,
                        type_casts: 34, null_remediations: 12}
  - Error pattern: {oracle_date_functions: 15 errors,
                    NUMBER_precision_loss: 3 errors,
                    NLS_encoding: 8 errors}
  - Strategy: {parallel_tables: true, batch_size: 50000,
               checkpoint_interval: 100000, total_time: 4.2hrs}

Encoded as vector → stored in Pinecone with metadata
Graph relationships → stored in Neo4j
```

When Migration #1204 begins (SAP HR → Snowflake), the system queries:

```text
Similarity search: "HR domain schema, high FK density, DATE-heavy"
Returns: Migration #847, #612, #923, #1089 as top matches

Cognitive Agents receive:
  "Similar migrations had 15+ Oracle date function errors.
   Pre-converting to ISO format reduced errors by 78%.
   Recommended batch size: 50,000. Expected duration: 3-5 hours."
```

The new migration starts with a head start that took prior migrations hours of discovery to accumulate.

---

### Vector Encoding

Each migration pattern is encoded as a fixed-dimensional vector that captures its essential characteristics without retaining customer-specific details.

#### What's Encoded

- **Schema structure:** Table count, column count distribution, type distribution, relationship density, constraint complexity
- **Transformation profile:** Categories and frequencies of transformations applied
- **Error profile:** Categories and frequencies of errors encountered, resolution methods used
- **Performance profile:** Throughput curves, bottleneck classifications, optimization effectiveness
- **Strategy effectiveness:** Which strategy parameters produced the best outcomes

#### What's NOT Encoded

- Table names, column names, or any identifiable schema elements
- Data values, sample records, or content
- Customer identity or business context
- Connection strings, credentials, or infrastructure details

The abstraction ensures that the vector captures transferable intelligence (structural similarity, transformation patterns, error likelihood) without leaking customer-specific information.

---

### Similarity Matching

When a new migration begins, the system computes a profile vector from the source and target schema analysis and queries the vector database for the k nearest neighbors (default k=10).

**Similarity metric:** Cosine similarity in the vector space
**Threshold:** Matches above 0.92 cosine similarity are considered high-confidence and can be applied with minimal Cognitive Agent review. Matches between 0.75-0.92 serve as planning priors that bias analysis toward likely-correct approaches. Matches below 0.75 are informational only.

The matching happens at multiple granularities:

- **Migration-level:** Overall source-target pair similarity
- **Table-level:** Individual table structure similarity
- **Column-level:** Specific column pattern similarity (used by MABA for schema mapping)

---

### Privacy Preservation

Cross-migration learning creates an inherent tension: the more specific the stored patterns, the more useful they are for future migrations — but also the more risk of leaking customer information.

Sensei resolves this through AMANI's privacy-preserving pipeline:

1. **Abstraction:** Patterns are extracted at the structural level, stripping all identifiable content before encoding
2. **Differential privacy:** Calibrated Gaussian noise is added to pattern vectors before storage (ε = 1.0 default)
3. **Privacy budget tracking:** Each customer has a cumulative epsilon budget. When exhausted, no further patterns are extracted from that customer's migrations
4. **Homomorphic matching:** Similarity queries can be executed on encrypted vectors, so the vector database never sees the raw query pattern

Customers can configure:

- **Full participation:** Patterns extracted and shared (default, with differential privacy)
- **Receive-only:** Customer benefits from the library but doesn't contribute
- **Isolated:** No cross-customer learning; customer maintains private pattern library

→ [Evolutionary Tournament Selection](tournament-selection.md)
→ [Privacy-Preserving Learning](../../product/platform/amani/privacy-preserving-learning.md)
