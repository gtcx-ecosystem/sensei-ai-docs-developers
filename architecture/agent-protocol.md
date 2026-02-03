# Agent Communication Protocol

## Agent Coordination Protocol

Sensei's agents aren't isolated workers executing independent tasks. They're a coordinated swarm that communicates, shares discoveries, escalates problems, and collectively improves in real-time.

---

### Communication Channels

Agents communicate through three distinct channels, each optimized for different interaction patterns:

#### 1. Broadcast Channel (Redis pub/sub)

**Pattern:** One-to-many, fire-and-forget
**Use:** Swarm learning — when an agent discovers a pattern or solution, it broadcasts to all peers.
**Latency:** Sub-millisecond within a cluster.
**Durability:** At-most-once delivery. Acceptable because pattern broadcasts are idempotent — receiving the same pattern twice is harmless, and missing one broadcast means the agent discovers the pattern independently (slightly slower but not incorrect).

**Topics:**

- `migration.{id}.patterns` — New pattern discoveries
- `migration.{id}.errors` — Error-solution pairs
- `migration.{id}.performance` — Throughput observations
- `migration.{id}.alerts` — Anomaly notifications

#### 2. Workflow Channel (Temporal)

**Pattern:** Orchestrated sequences with state persistence
**Use:** Migration DAG execution — tasks that must execute in order with guaranteed delivery and retry logic.
**Latency:** ~10ms per step.
**Durability:** Exactly-once execution. Workflow state survives node failures. Critical for migration correctness — a record must not be migrated twice or skipped.

**Workflows:**

- `MigrationPlanWorkflow` — Top-level DAG execution
- `TableMigrationWorkflow` — Per-table migration with checkpointing
- `ValidationWorkflow` — Multi-perspective validation sequence
- `RollbackWorkflow` — Coordinated rollback on failure

#### 3. Conversation Channel (AutoGen)

**Pattern:** Multi-turn structured dialogue between agents
**Use:** Complex reasoning that requires multiple agents to collaborate — Scout presenting findings to Architect, Architect negotiating resource allocation with Performance Optimizer, Validator escalating an ambiguous anomaly to Cognitive tier.
**Latency:** Seconds (involves LLM inference).
**Durability:** Conversation logs persisted to MongoDB for Meta Agent analysis.

---

### Message Types

All inter-agent messages follow a standard envelope:

```yaml
AgentMessage:
  id: uuid
  timestamp: iso8601
  source_agent: agent_id
  target: agent_id | broadcast_topic
  migration_context: migration_id
  type: discovery | error | solution | escalation | directive | status
  priority: low | normal | high | critical
  payload:
    # Type-specific content
  confidence: 0.0-1.0
  ttl: seconds (for broadcast messages)
```

#### Key Message Types

| Type           | Direction                             | Purpose                                                |
| -------------- | ------------------------------------- | ------------------------------------------------------ |
| **Discovery**  | Worker → Broadcast                    | "I found a new date format pattern"                    |
| **Error**      | Worker → Broadcast + Escalation queue | "I hit an encoding mismatch I can't resolve"           |
| **Solution**   | Worker → Broadcast                    | "Here's how I fixed that encoding mismatch"            |
| **Escalation** | Execution → Cognitive                 | "This requires reasoning beyond my model's capability" |
| **Directive**  | Cognitive → Execution                 | "Use this strategy for the next batch"                 |
| **Status**     | Any → Monitoring                      | "Migration progress: 47% complete, 3 errors recovered" |

---

### Escalation Protocol

Not every problem can be solved by Execution Agents. The escalation protocol defines when and how problems are elevated to Cognitive Agents:

**Level 1 — Worker Self-Resolution**
Worker Agent attempts to resolve using its local model and the pattern library. Success rate: ~85% (at system maturity).

**Level 2 — Peer Consultation**
Worker broadcasts the problem. If another Worker has seen a similar issue, it responds with a solution. Adds ~100ms latency.

**Level 3 — Cognitive Escalation**
Problem is sent to a Scout or Architect Agent for analysis using a frontier model. The Cognitive Agent examines the full context — schema structure, data sample, error details, attempted solutions — and generates a resolution strategy. Adds seconds of latency but brings reasoning power.

**Level 4 — Human Escalation**
If Cognitive Agents cannot resolve with sufficient confidence (configurable threshold, default 80%), the issue is escalated to a human operator via AMANI. The human receives a structured summary of the issue, attempted resolutions, and recommended actions — not a raw error log.

**Level 5 — Migration Pause**
For critical issues (potential data corruption, compliance violation, integrity breach), the migration is paused automatically. All agents halt. AMANI notifies relevant stakeholders. Migration resumes only after human authorization.

---

### Agent Lifecycle

```text
┌─────────┐     ┌──────────┐     ┌─────────┐     ┌──────────┐
│  SPAWN  │ ──→ │  READY   │ ──→ │ ACTIVE  │ ──→ │ COMPLETE │
└─────────┘     └──────────┘     └─────────┘     └──────────┘
                     ↑                │
                     │           ┌────┴────┐
                     └───────────│  ERROR  │
                                 └─────────┘
```

- **Spawn:** Agent instantiated with model, tools, and migration context
- **Ready:** Agent registered with swarm, subscribed to relevant broadcast topics
- **Active:** Agent pulling and processing tasks
- **Error:** Agent encountered a fatal error; attempts self-recovery, escalates if needed, respawns if unrecoverable
- **Complete:** Agent's work is done; execution log submitted to Meta Agents for pattern extraction

Agents are ephemeral — they're spawned for a migration and terminated on completion. Persistent state lives in the vector database, graph database, and workflow engine, not in agent memory.

---

### Concurrency and Conflict Resolution

When multiple agents make conflicting decisions (e.g., two Worker Agents propose different transformations for the same column), the protocol uses a confidence-weighted resolution:

1. Both agents submit their proposed transformation with confidence scores
2. If one agent has significantly higher confidence (>0.15 delta), its proposal is accepted
3. If confidence is similar, the proposals are escalated to a Cognitive Agent for arbitration
4. The resolution is broadcast as a pattern for future reference

This prevents both gridlock (no decision) and inconsistency (conflicting decisions applied to different records).

→ [Swarm Learning](swarm-learning.md)
→ [Vector Learning](vector-learning.md)
