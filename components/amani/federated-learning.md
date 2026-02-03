# Federated Pattern Extraction

## Your Data Never Leaves Your Network

Federated pattern extraction is the first pillar of AMANI's [privacy-preserving learning](../../../product/platform/amani/privacy-preserving-learning.md). Customer migration data is processed entirely within the customer's environment — only abstract patterns are transmitted to the central knowledge base.

---

### How It Works

A lightweight pattern extraction agent runs inside the customer's network perimeter:

```text
┌─────────────────────────────────────────────────────┐
│                Customer Network                      │
│                                                      │
│  ┌──────────────┐     ┌──────────────────────────┐ │
│  │ Migration     │ ──→ │  Pattern Extraction      │ │
│  │ (MABA + KORA) │     │  Agent                   │ │
│  └──────────────┘     │                          │ │
│                        │  Observes migration      │ │
│                        │  Extracts abstractions   │ │
│                        │  Adds DP noise           │ │
│                        └──────────┬───────────────┘ │
│                                   │                  │
└───────────────────────────────────┼──────────────────┘
                                    │ (abstract patterns only)
                                    ↓
                         ┌─────────────────────┐
                         │ Sensei Central       │
                         │ Knowledge Base       │
                         └─────────────────────┘
```

---

### What Gets Extracted

The agent observes the migration operation and extracts abstract patterns in four categories:

#### 1. Schema Structural Fingerprints

Statistical properties of the source and target schemas, without actual names:

- Data type distribution (percentage numeric, string, temporal, boolean)
- Cardinality ratios (distinct values / total rows)
- Null rate distributions
- Relationship density (FK count / table count)
- Constraint patterns (PK types, index coverage)

**What's transmitted:** `{type_dist: [0.35, 0.45, 0.15, 0.05], card_ratio_mean: 0.73, ...}`
**What's NOT transmitted:** Table names, column names, actual values

#### 2. Performance Profiles

Characteristics of how the migration executed:

- Throughput curves (records/hour over time)
- Bottleneck classifications (network, CPU, I/O, destination)
- Resource utilization patterns
- Optimization effectiveness (which interventions helped)

**What's transmitted:** `{throughput_profile: [0.8, 0.9, 0.7, 0.95], bottleneck: "destination", ...}`
**What's NOT transmitted:** Actual record counts, timing of specific operations

#### 3. Error Signatures

Patterns of errors encountered and resolved:

- Error type frequencies (type mismatch, encoding, constraint violation)
- Recovery success rates by error type
- Escalation patterns

**What's transmitted:** `{error_dist: {type_mismatch: 0.4, encoding: 0.3, constraint: 0.2, other: 0.1}, recovery_rate: 0.94}`
**What's NOT transmitted:** Actual error messages, record identifiers, data values

#### 4. Strategy Effectiveness

Which migration strategies worked for this migration profile:

- Strategy parameters used
- Outcome metrics (speed, accuracy, efficiency)
- Relative effectiveness compared to alternatives tried

**What's transmitted:** `{strategy_id: 47, effectiveness: 0.92, profile_match: 0.88}`
**What's NOT transmitted:** Customer identifier, migration details

---

### What Never Leaves

The following data categories are never transmitted, under any circumstances:

- Raw data values from any table
- Table names or column names
- Schema identifiers or database names
- Customer identifiers or metadata
- Query text or stored procedure content
- User information or credentials
- Any data that could identify the source system

The pattern extraction is designed so that even if the transmitted patterns were intercepted, they would reveal nothing about the specific customer or their data — just abstract structural and performance characteristics.

---

### Local Processing

The pattern extraction agent runs entirely within the customer's control:

- **Deployed in customer VPC:** The agent is a container running in the customer's infrastructure
- **No external data access:** The agent reads from the migration pipeline; it cannot access other customer systems
- **Customer-auditable:** Customers can inspect the agent's code and network traffic
- **Customer-terminable:** Customers can disable or remove the agent at any time

This architecture means customers don't have to trust Sensei's promises about data handling — they can verify that raw data never leaves their network.

---

### Differential Privacy Addition

Before transmission, the extracted patterns are further protected by [differential privacy](differential-privacy.md) noise. This ensures that even the abstract patterns can't be used to infer customer-specific information.

→ [Differential Privacy](differential-privacy.md)
→ [Privacy-Preserving Learning Overview](../../../product/platform/amani/privacy-preserving-learning.md)
