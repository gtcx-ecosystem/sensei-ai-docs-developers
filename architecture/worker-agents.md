---
description: Transformation execution agents
---

# Worker Agents

## The Hands That Move the Data

Worker Agents are the execution backbone of every migration. They pick up tasks from the migration DAG, execute transformations, recover from errors, and broadcast discoveries to the swarm — all autonomously.

---

### Role in the Hierarchy

Worker Agents are one of two Execution Agent types (alongside [Validator Agents](validator-agents.md)). They use smaller, faster models (Llama-3-8B or Mistral) optimized for throughput rather than deep reasoning.

```text
Scout Agents → Architect Agents
  ↓ Executable Migration Plan
Worker Agents (YOU ARE HERE) → Validator Agents
  ↓ Execution Logs
Meta Agents (Learning)
```

The tradeoff is deliberate: Worker Agents handle thousands of records per second. They need fast pattern matching and error classification, not the multi-step reasoning that Cognitive Agents provide. When a problem exceeds their capability, they escalate.

---

### Core Capabilities

#### 1. Task Execution

Workers pull tasks from the migration DAG via Ray's distributed task queue. Each task typically involves:

1. Read a batch of records from the source system
2. Apply the transformation chain specified by the Architect Agent
3. Write transformed records to the target system
4. Report completion and any issues to the swarm

Workers are **stateless** — they don't maintain migration context between batches. All state lives in the workflow engine (Temporal) and the shared pattern cache. This means workers can be spawned, killed, and replaced without data loss.

#### 2. Autonomous Error Recovery

When a transformation fails, the Worker doesn't just log the error and stop. It initiates a multi-step recovery process:

**Step 1: Classify the error**
The Worker's local model classifies the error into known categories: type mismatch, encoding error, constraint violation, null violation, format error, connection failure, timeout.

**Step 2: Search for known solutions**
The Worker checks three sources in order:

- Local swarm cache (patterns discovered during this migration)
- Cross-migration pattern library (solutions from prior migrations)
- Error-solution pairs from the Meta Agent knowledge graph

**Step 3: Generate a fix**
If no known solution exists, the Worker generates a candidate fix using its local model. For common error types, this works well — Llama-3-8B can write a type conversion or encoding fix reliably.

**Step 4: Test the fix**
The candidate fix is tested against the failing records in a sandboxed environment. If the fix produces correct output (validated against expected constraints), it's applied.

**Step 5: Broadcast or escalate**

- If the fix works → broadcast the error-solution pair to the swarm
- If the fix fails → escalate to a Cognitive Agent for deeper analysis

**Recovery success rates improve over time:**

| Maturity | Auto-Recovery Rate | Avg Recovery Time |
| -------- | ------------------ | ----------------- |
| Month 1  | 60%                | ~30 seconds       |
| Month 6  | 85%                | ~10 seconds       |
| Year 1   | 95%                | ~3 seconds        |
| Year 3   | 99%+               | ~1 second         |

The improvement comes from the growing pattern library — most "new" errors have been seen before in prior migrations.

#### 3. Dynamic Resource Optimization

Workers monitor their own resource consumption and adapt:

- **CPU-bound tasks:** Worker signals the Performance Optimizer, which spawns additional workers
- **I/O-bound tasks:** Worker reduces batch size to decrease memory pressure and increases parallelism on reads
- **Network-bound tasks:** Worker increases compression and pipelines requests
- **Memory-bound tasks:** Worker switches from in-memory transformation to streaming mode

These adaptations happen autonomously, guided by the Performance Optimizer Agent's real-time telemetry analysis.

#### 4. Self-Organizing Work Distribution

Workers don't receive explicit task assignments. They pull from a shared task queue managed by Ray, naturally gravitating toward tasks they can process efficiently.

Over time, workers develop **organic specialization**: a worker that has processed several address-format tables accumulates a warm pattern cache for address variations. Ray's work-stealing scheduler routes similar tasks to workers with warm caches, creating implicit specialization without explicit configuration.

---

### Scaling Behavior

Workers auto-scale via Ray based on backpressure metrics:

| Migration Size      | Baseline Workers | Peak Workers | Scale Trigger             |
| ------------------- | ---------------- | ------------ | ------------------------- |
| Small (<1M records) | 4                | 8            | Queue depth > 100 tasks   |
| Medium (1-10M)      | 8                | 32           | Queue depth > 500 tasks   |
| Large (10-100M)     | 16               | 64           | Queue depth > 1,000 tasks |
| Very Large (>100M)  | 32               | 128          | Queue depth > 5,000 tasks |

Scale-up takes ~30 seconds (new Ray worker initialization). Scale-down is gradual — workers are drained (finish current task) before termination.

---

### Interaction with Validator Agents

Workers and [Validators](validator-agents.md) operate in a continuous feedback loop:

```text
Source → Worker (transform) → Validator (check) → Target
                                    ↓ (failure)
                              Error Queue → Worker (retry with fix)
```

Validators don't just catch errors after the fact — they run continuously alongside workers, checking records as they flow through the pipeline. This means errors are caught within seconds of occurrence, not hours later in a post-migration validation pass.

→ [Validator Agents](validator-agents.md)
→ [Swarm Learning](swarm-learning.md)
→ [Execution Agents Overview](three-layer-architecture/execution-agents.md)
