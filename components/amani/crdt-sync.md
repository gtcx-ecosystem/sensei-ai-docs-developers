---
description: Conflict-free replicated data types
---

# CRDT Synchronization

## Merging Offline Work Without Conflicts

When AMANI operates [offline](../../../product/platform/amani/offline-first.md), users continue making decisions, approving mappings, and progressing through migration workflows. When connectivity returns, these offline changes must merge with any updates that occurred on the server. AMANI uses **Conflict-Free Replicated Data Types (CRDTs)** to handle this automatically.

---

### The Problem CRDTs Solve

Traditional synchronization requires conflict resolution. If User A changes a setting offline while User B changes it on the server, which wins? Someone must decide, and decisions can lose data.

CRDTs are data structures mathematically designed so that:

- Any two replicas can merge automatically
- Merge order doesn't matter (commutative, associative)
- No conflicts are possible by construction
- No data is ever lost

---

### CRDT Types in AMANI

#### G-Counter (Grow-Only Counter)

For metrics that only increase:

- Records processed
- Errors encountered
- Validation checks passed

```text
Local counter: {node_1: 5000, node_2: 0}
Server counter: {node_1: 3000, node_2: 2000}
Merged: {node_1: 5000, node_2: 2000} → total: 7000
```

Each node tracks only its own increments. Merge takes the maximum per node. Result: accurate total even with concurrent updates.

#### LWW-Register (Last-Writer-Wins Register)

For settings where the most recent value should win:

- Configuration options
- User preferences
- Current phase status

```text
Local: {value: "balanced", timestamp: 1706789400}
Server: {value: "safe", timestamp: 1706789300}
Merged: {value: "balanced", timestamp: 1706789400}
```

The value with the higher timestamp wins. Ties broken by node ID.

#### OR-Set (Observed-Remove Set)

For collections where items can be added and removed:

- Approved mappings
- Acknowledged errors
- Completed tasks

```text
Local adds: {mapping_A, mapping_B}
Local removes: {mapping_C}
Server adds: {mapping_C, mapping_D}
Server removes: {mapping_A}
Merged: {mapping_B, mapping_D}
```

Add wins over concurrent remove. Removes only affect items seen before the remove.

#### LWW-Map (Last-Writer-Wins Map)

For key-value structures:

- Migration state
- Per-table progress
- Per-column mapping decisions

Each key is an independent LWW-Register. Keys can be added but not removed (use tombstone values for logical deletion).

---

### Synchronization Protocol

When AMANI reconnects after offline operation:

**Phase 1: State Exchange**

```text
Client → Server: Here's my state vector (what I've seen)
Server → Client: Here's changes since your last sync
Client → Server: Here's changes I made offline
```

**Phase 2: Merge**
Both client and server independently merge the received changes using CRDT merge operations. Because CRDTs are deterministic, both arrive at identical state.

**Phase 3: Verification**

```text
Client → Server: My merged state hash is X
Server → Client: My merged state hash is X [OK]
```

If hashes match, sync is complete. If not, full state comparison identifies the discrepancy (rare, indicates a bug).

---

### What Syncs

| Data Category        | CRDT Type           | Sync Priority        |
| -------------------- | ------------------- | -------------------- |
| Migration progress   | G-Counter + LWW-Map | High (sync first)    |
| User decisions       | OR-Set              | High                 |
| Configuration        | LWW-Register        | Medium               |
| Conversation history | Append-only log     | Medium               |
| Cached patterns      | LWW-Map             | Low (can re-fetch)   |
| Logs and metrics     | G-Counter           | Low (can regenerate) |

High-priority data syncs immediately on reconnection. Low-priority data syncs in background.

---

### Handling Semantic Conflicts

CRDTs guarantee no _technical_ conflicts, but _semantic_ conflicts can still occur:

**Example:** User A approves mapping X offline. User B rejects mapping X on server. CRDT merges to "approved" (because both add and remove are in the set, and add wins).

AMANI handles this by:

1. Detecting when merge produces a different result than server-side state
2. Flagging these as "merge decisions" requiring review
3. Notifying relevant users that their action was superseded
4. Providing audit trail showing both decisions and the merge outcome

Semantic conflicts are rare (<1% of sync operations) and are surfaced clearly for human review.

---

### Performance

| Metric                          | Value                                        |
| ------------------------------- | -------------------------------------------- |
| Sync initiation                 | <1 second after connectivity detected        |
| Typical sync duration           | 2-10 seconds (depending on offline duration) |
| Maximum offline period          | 30 days (limited by local storage, not CRDT) |
| Bandwidth (typical sync)        | 10-100 KB                                    |
| Bandwidth (1-week offline sync) | 1-5 MB                                       |

Sync is incremental — only changes since last sync are exchanged. Long offline periods don't create proportionally large sync payloads because CRDTs merge redundant updates.

---

### Why Not Traditional Sync?

| Approach                    | Conflict Handling  | Data Loss Risk              | Complexity                         |
| --------------------------- | ------------------ | --------------------------- | ---------------------------------- |
| Last-write-wins (timestamp) | Later change wins  | High (earlier changes lost) | Low                                |
| Server-wins                 | Server always wins | High (client changes lost)  | Low                                |
| Client-wins                 | Client always wins | High (server changes lost)  | Low                                |
| Manual resolution           | User decides       | Low                         | High (requires UI, user attention) |
| **CRDTs**                   | Automatic merge    | **None**                    | Medium (engineering complexity)    |

CRDTs are harder to implement but eliminate an entire category of problems. For offline-first applications, they're the right choice.

→ [Offline-First Architecture](../../../product/platform/amani/offline-first.md)
→ [AMANI Overview](../../../product/platform/amani/README.md)
