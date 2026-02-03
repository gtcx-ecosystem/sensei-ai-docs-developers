---
description: Recording source system history
---

# Temporal State Capture

## Capturing Source History

Temporal State Capture is the preparation phase of [Time-Travel Testing](../../../product/platform/kora/time-travel-testing.md). Before migration begins, KORA instruments the source system to record everything needed for post-migration behavioral verification.

---

### What Gets Captured

#### Database Snapshots

Full or incremental snapshots of the source database at defined intervals:

- **Full snapshots:** Complete database state at the first checkpoint
- **Incremental snapshots:** Change-data-capture (CDC) deltas from each subsequent checkpoint, recording only what changed since the previous capture

Snapshot method depends on the source database:

- **PostgreSQL:** Logical replication slots or pg_dump
- **Oracle:** Flashback queries or RMAN incremental
- **SQL Server:** Change Data Capture or database snapshots
- **MySQL:** Binary log replication or mysqldump
- **Others:** Application-level CDC or periodic full export

#### Query Log Sampling

KORA samples representative queries and transactions from the source system's audit/query log:

- **Read queries:** SELECT statements that produce business-critical outputs (reports, dashboards, API responses)
- **Write transactions:** INSERT/UPDATE/DELETE sequences that test data modification behavior
- **Stored procedure calls:** Invocations of business logic procedures with their input parameters
- **Report generation:** Scheduled report queries that produce known outputs

Sample selection prioritizes diversity:

- Different query patterns (joins, aggregations, subqueries, window functions)
- Different table access patterns (single-table, multi-table, full-scan, index-seek)
- Different time ranges (current data, historical data, cross-period queries)
- Different user roles (admin queries, application queries, report queries)

Default sample size: 10,000-15,000 queries. Configurable based on migration complexity and verification requirements.

#### Business Logic Definitions

Complete capture of executable business logic:

| Artifact             | What's Captured                                          |
| -------------------- | -------------------------------------------------------- |
| Stored procedures    | Full definition text, parameter signatures, dependencies |
| Triggers             | Definition text, event types, timing (BEFORE/AFTER)      |
| Views                | Complete view definitions including nested views         |
| Computed columns     | Formula definitions                                      |
| Materialized views   | Query definitions and refresh schedules                  |
| Functions            | UDFs, aggregate functions, table-valued functions        |
| Configuration tables | Full content of tables storing business parameters       |

---

### Checkpoint Frequency

Checkpoint intervals are calibrated to the source system's change rate:

| System Type                     | Change Rate               | Checkpoint Interval | Typical Count  |
| ------------------------------- | ------------------------- | ------------------- | -------------- |
| High-churn transactional (OLTP) | Thousands of changes/hour | Hourly              | 168 (1 week)   |
| Moderate-activity (mixed)       | Hundreds of changes/hour  | Every 4-6 hours     | 28-42 (1 week) |
| Stable reference data           | Tens of changes/day       | Daily               | 7 (1 week)     |
| Near-static archive             | Rare changes              | Weekly              | 4 (1 month)    |

More checkpoints provide finer-grained verification but consume more storage and replay time. KORA recommends the appropriate interval based on source system activity analysis.

---

### Cryptographic Hash Chain

Each checkpoint is hashed and linked to form a tamper-proof chain:

```text
Checkpoint 1 → Hash(data₁ + logic₁ + queries₁)          = H₁
Checkpoint 2 → Hash(data₂ + logic₂ + queries₂ + H₁)     = H₂
Checkpoint 3 → Hash(data₃ + logic₃ + queries₃ + H₂)     = H₃
...
```

This chain guarantees:

- No checkpoint can be modified after creation without breaking the chain
- The sequence of captures is verifiable
- Auditors can confirm that source system state was faithfully recorded

The hash chain becomes part of the [Behavioral Equivalence Certificate](behavioral-equivalence-certification.md).

---

### Storage Requirements

| Migration Size       | Checkpoints | Snapshot Size | Query Samples | Total Storage |
| -------------------- | ----------- | ------------- | ------------- | ------------- |
| Small (1M rows)      | 7           | ~2 GB         | ~50 MB        | ~2.5 GB       |
| Medium (10M rows)    | 14          | ~20 GB        | ~200 MB       | ~22 GB        |
| Large (100M rows)    | 28          | ~200 GB       | ~500 MB       | ~210 GB       |
| Very Large (1B rows) | 42          | ~2 TB         | ~1 GB         | ~2.1 TB       |

Incremental snapshots reduce storage by 60-80% compared to full snapshots at every checkpoint. Storage is temporary — checkpoint data can be archived or deleted after certification is complete.

→ [Historical Replay](historical-replay.md)
→ [Time-Travel Testing Overview](../../../product/platform/kora/time-travel-testing.md)
