# Agent Communication

## How Agents Discover, Share, and Learn in Real-Time

Sensei's agents communicate through a combination of real-time messaging (Redis pub/sub), durable workflows (Temporal), and shared knowledge stores (vector and graph databases). This page explains the communication infrastructure that enables swarm intelligence.

---

### Communication Channels

| Channel                    | Technology                    | Purpose                             | Latency |
| -------------------------- | ----------------------------- | ----------------------------------- | ------- |
| **Swarm broadcast**        | Redis pub/sub                 | Real-time pattern/discovery sharing | <10ms   |
| **Workflow orchestration** | Temporal                      | Durable, ordered task execution     | <100ms  |
| **Knowledge query**        | Vector DB (Pinecone/Weaviate) | Pattern similarity search           | <50ms   |
| **Relationship traversal** | Graph DB (Neo4j)              | Pattern relationship lookup         | <30ms   |
| **State persistence**      | MongoDB                       | Conversation and session state      | <20ms   |

---

### Swarm Communication (Redis Pub/Sub)

When an agent discovers something useful, it broadcasts immediately to the swarm:

```text
┌─────────────┐     Redis Pub/Sub      ┌─────────────┐
│   Worker    │ ─────────────────────→ │   All       │
│   Agent 1   │  "date_format_found"   │   Workers   │
└─────────────┘                        └─────────────┘
```

**Channel structure:**

```text
sensei:{migration_id}:discoveries    # Pattern discoveries
sensei:{migration_id}:errors         # Error broadcasts
sensei:{migration_id}:solutions      # Successful recoveries
sensei:{migration_id}:metrics        # Performance metrics
sensei:global:patterns               # Cross-migration patterns
```

**Message format:**

```json
{
  "type": "pattern_discovery",
  "agent_id": "worker_47",
  "timestamp": "2026-02-02T14:23:01.234Z",
  "pattern": {
    "category": "date_format",
    "signature": "YYYY-MM-DD HH:MI:SS.FFF",
    "transformation": "TO_TIMESTAMP(col, 'YYYY-MM-DD HH24:MI:SS.FF3')",
    "confidence": 0.98
  },
  "context": {
    "table": "transactions",
    "column": "created_at",
    "sample_count": 1000
  }
}
```

**Subscription patterns:**

```python
# Worker agent subscribes to relevant channels
redis.subscribe(
    f"sensei:{migration_id}:discoveries",
    f"sensei:{migration_id}:solutions",
    f"sensei:global:patterns"
)

# Callback on message
def on_message(channel, message):
    if message.type == "pattern_discovery":
        self.local_cache.add(message.pattern)
    elif message.type == "error_solution":
        self.recovery_strategies.update(message.solution)
```

---

### Workflow Orchestration (Temporal)

Long-running, multi-step operations use Temporal for durability and ordering:

```yaml
Workflow: migration_execution
  Activities:
    - analyze_schema (timeout: 1h, retry: 3)
    - generate_plan (timeout: 30m, retry: 2)
    - await_approval (timeout: 7d, retry: 0)
    - execute_phase_1 (timeout: 24h, retry: 5)
    - execute_phase_2 (timeout: 24h, retry: 5)
    - execute_phase_3 (timeout: 24h, retry: 5)
    - verify_migration (timeout: 6h, retry: 3)
    - generate_certificate (timeout: 1h, retry: 2)
```

Temporal provides:

- **Durability:** Workflows survive process crashes
- **Visibility:** Current state queryable at any time
- **Retry logic:** Configurable retry with backoff
- **Timeouts:** Prevent hung operations
- **Versioning:** Safe workflow updates during execution

---

### Knowledge Queries

Agents query shared knowledge stores for patterns and relationships:

**Vector similarity search (Pinecone/Weaviate):**

```python
# Find similar schema patterns
results = vector_db.query(
    vector=fingerprint.to_vector(),
    top_k=10,
    filter={"migration_type": "oracle_to_postgresql"}
)

# Results include patterns from prior migrations
for result in results:
    if result.score > 0.92:
        # High confidence match - apply directly
        apply_pattern(result.pattern)
    elif result.score > 0.75:
        # Medium confidence - use as prior for LLM
        priors.append(result.pattern)
```

**Graph traversal (Neo4j):**

```cypher
// Find related error-solution pairs
MATCH (e:ErrorPattern {type: 'encoding_mismatch'})
      -[:RESOLVED_BY]->(s:Solution)
      -[:EFFECTIVE_FOR]->(p:MigrationProfile)
WHERE p.source_type = 'oracle' AND p.target_type = 'postgresql'
RETURN s.code, s.success_rate
ORDER BY s.success_rate DESC
LIMIT 5
```

---

### Agent Discovery Protocol

When agents need to find each other (e.g., a Worker Agent needs Validator feedback):

```text
1. Worker queries Redis for active Validators
   SMEMBERS sensei:{migration_id}:agents:validator

2. Redis returns list of agent IDs
   [validator_1, validator_2, validator_3]

3. Worker publishes request to specific Validator
   PUBLISH sensei:{migration_id}:validator_1:requests {...}

4. Validator responds via Worker's response channel
   PUBLISH sensei:{migration_id}:worker_47:responses {...}
```

This enables direct agent-to-agent communication while maintaining observability through Redis.

---

### Swarm Learning Loop

The complete learning loop from discovery to global knowledge:

```text
T+0ms     Worker discovers pattern locally
T+5ms     Worker broadcasts to migration swarm (Redis)
T+10ms    All workers in migration receive pattern
T+15ms    Workers update local caches
          ─────── IMMEDIATE LEARNING COMPLETE ───────

T+100ms   Meta Agent receives pattern
T+200ms   Meta Agent validates pattern quality
T+300ms   Meta Agent embeds pattern as vector
T+400ms   Pattern stored in vector DB (Pinecone)
          ─────── CROSS-MIGRATION LEARNING COMPLETE ───────

T+500ms   Learning Agent analyzes pattern relationships
T+600ms   Graph DB updated with pattern connections
T+700ms   Evolutionary selection evaluates strategy impact
          ─────── STRATEGIC LEARNING COMPLETE ───────
```

---

### Failure Modes and Recovery

| Failure                | Impact                     | Recovery                                         |
| ---------------------- | -------------------------- | ------------------------------------------------ |
| Redis unavailable      | Swarm communication paused | Agents operate locally; sync on recovery         |
| Temporal unavailable   | New workflows can't start  | In-flight workflows continue; queue new requests |
| Vector DB unavailable  | Pattern matching degraded  | Fall back to LLM inference                       |
| Neo4j unavailable      | Relationship lookup fails  | Use flat pattern matching                        |
| Individual agent crash | Partial capacity loss      | Ray auto-scales replacement                      |

The system degrades gracefully — no single communication failure stops the migration.

---

### Observability

All agent communication is observable:

```yaml
Metrics:
  - sensei_redis_messages_total{channel, type}
  - sensei_redis_latency_seconds{channel, operation}
  - sensei_temporal_workflow_duration_seconds{workflow}
  - sensei_vector_query_latency_seconds{operation}
  - sensei_agent_message_rate{agent_type}

Traces:
  - Distributed traces span agent boundaries
  - Correlation IDs link related messages
  - Sampling configurable (default 10%)

Logs:
  - Structured JSON logs
  - Agent ID, migration ID, correlation ID in every log
  - Log levels: debug, info, warn, error
```

→ [Component Orchestration](component-orchestration.md)
→ [Data Flow](data-flow.md)
→ [Integration Overview](README.md)
