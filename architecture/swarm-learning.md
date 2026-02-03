---
description: Distributed learning across migrations
---

# In-Migration Swarm Learning

## Collective Intelligence in Milliseconds

Swarm learning is the fastest of Sensei's three learning loops. It operates within a single migration, enabling agents to share discoveries in real-time so the swarm gets collectively smarter as the migration progresses.

---

### How It Works

When a Worker Agent encounters and resolves a new situation — an unfamiliar date format, an encoding mismatch, a constraint violation — it serializes the discovery as a pattern-solution pair and broadcasts it to all peer agents via Redis pub/sub.

```text
Agent #47 encounters: "YYYY年MM月DD日" (Japanese date format in VARCHAR)
Agent #47 resolves:   TO_DATE(col, 'YYYY"年"MM"月"DD"日"')
Agent #47 broadcasts:  {pattern: "CJK_date_format", regex: "\\d{4}年\\d{2}月\\d{2}日",
                        solution: "TO_DATE with literal separators",
                        confidence: 0.95}

Time elapsed: ~200ms from discovery to swarm-wide availability

Agent #12 encounters the same format 3 seconds later
Agent #12 matches against swarm knowledge → applies known solution immediately
Agent #12 skips: error detection, diagnosis, solution generation, testing
Time saved: ~5 seconds per occurrence × thousands of records = hours saved
```

---

### Pattern-Solution Pairs

Every swarm broadcast follows a standard structure:

```yaml
SwarmPattern:
  pattern_id: uuid
  discovered_by: agent_id
  timestamp: iso8601

  # What was encountered
  pattern:
    category: date_format | encoding | constraint | type_mismatch | null_handling | custom
    signature: string # Compact representation for matching
    context:
      table: string
      column: string
      data_type: string
      sample_values: list[string]

  # How it was resolved
  solution:
    transformation: string # Code or SQL expression
    pre_conditions: list[string] # When this solution applies
    post_validation: string # How to verify the fix worked

  confidence: float # 0.0-1.0
  test_result: pass | fail # Was solution verified against sample data?
```

---

### Broadcast Mechanics

**Channel:** `migration.{id}.patterns`
**Protocol:** Redis pub/sub
**Delivery:** At-most-once (idempotent — duplicates are harmless)
**Propagation latency:** <1ms within cluster

Each agent maintains a local pattern cache that grows through the migration:

1. Agent starts with patterns loaded from the cross-migration vector library (Loop 2)
2. As the migration progresses, swarm broadcasts add new patterns to the local cache
3. When encountering a new situation, agent checks local cache before attempting independent resolution
4. Cache hit → apply known solution (~1ms). Cache miss → diagnose and solve (~5-30 seconds), then broadcast.

---

### Swarm Intelligence Dynamics

The swarm exhibits emergent collective intelligence that exceeds any individual agent's capability:

**Specialization.** Agents that process tables with similar data types accumulate domain-specific pattern caches. An agent that has been processing customer address fields develops expertise in address format variations. Ray's work-stealing scheduler naturally routes similar work to agents with warm caches, creating organic specialization.

**Error front-running.** When an early agent encounters an error in Table A, the solution broadcasts before other agents reach the equivalent situation in Tables B, C, D. The error is experienced once and solved everywhere — the swarm "front-runs" errors that would otherwise be encountered independently by dozens of agents.

**Confidence calibration.** When multiple agents independently discover the same pattern, the confidence score increases. When agents discover conflicting solutions for the same pattern, the conflict triggers Cognitive Agent arbitration and the resolution is broadcast with higher confidence than either original solution.

---

### Performance Impact

For a typical migration of 50 million records across 200 tables:

| Metric                              | Without Swarm Learning | With Swarm Learning          |
| ----------------------------------- | ---------------------- | ---------------------------- |
| Unique errors encountered           | ~500                   | ~500 (same)                  |
| Total error occurrences             | ~15,000                | ~500 (first occurrence only) |
| Error resolution time (total)       | ~20 hours              | ~40 minutes                  |
| Agent idle time (waiting for fixes) | ~15%                   | ~1%                          |

The swarm doesn't reduce the number of _unique_ problems — it eliminates the redundant re-discovery of solutions across agents. The first encounter costs full resolution time. Every subsequent encounter costs a cache lookup.

→ [Cross-Migration Vector Learning](vector-learning.md)
→ [Agent Communication Protocol](agent-protocol.md)
