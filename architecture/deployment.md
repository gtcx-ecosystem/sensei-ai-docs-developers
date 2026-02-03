---
description: Production deployment patterns
---

# Deployment Architecture

## Cloud-Agnostic Deployment

Sensei deploys as a three-tier Kubernetes application designed to run anywhere — managed cloud, private infrastructure, or air-gapped environments.

---

### Three-Tier Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                     COGNITIVE TIER                            │
│                  (GPU-equipped nodes)                         │
│                                                              │
│  Scout Agents · Architect Agents · Frontier LLM inference    │
│  Nodes: 2-8 (GPU) · Models: GPT-4/Claude API or Llama-70B   │
├─────────────────────────────────────────────────────────────┤
│                     EXECUTION TIER                            │
│                  (CPU-optimized nodes)                        │
│                                                              │
│  Worker Agents · Validator Agents · Ray cluster               │
│  Nodes: 4-128 (auto-scaling) · Models: Llama-8B/Mistral     │
├─────────────────────────────────────────────────────────────┤
│                       META TIER                               │
│                  (Persistent services)                        │
│                                                              │
│  Learning Agents · Vector DB · Graph DB · Experiment tracker  │
│  Nodes: 2-4 (persistent) · Storage: Pinecone/Neo4j/MLflow   │
└─────────────────────────────────────────────────────────────┘
          ↕ Redis pub/sub          ↕ Temporal workflows
```

---

### Tier Specifications

#### Cognitive Tier

- **Nodes:** 2–8 GPU-equipped instances
- **GPU:** NVIDIA A100 (for local inference) or API-connected (GPT-4/Claude)
- **Scaling:** Vertical — upgrade node GPU capacity as needed
- **Stateless:** Agents can be restarted without data loss; all state is in vector/graph databases
- **Network:** Requires outbound HTTPS to LLM APIs (unless running local models)

#### Execution Tier

- **Nodes:** Baseline of 4, auto-scales to 128 based on migration workload
- **CPU:** Optimized for throughput (c5/c6 instance families or equivalent)
- **Scaling:** Horizontal via Ray auto-scaling, triggered by backpressure metrics
- **Memory:** 32–64 GB per node for in-memory transformation buffers
- **Network:** High-bandwidth connectivity to source and target databases

#### Meta Tier

- **Nodes:** 2–4 persistent instances
- **Storage:** Attached volumes for Neo4j, MongoDB; managed services for Pinecone
- **Scaling:** Minimal — Meta Agents operate on completed migration data, not live streams
- **Durability:** All data replicated across availability zones
- **Network:** Internal only — no direct external access

---

### Inter-Tier Communication

| Channel                | Purpose                                                       | Latency | Durability                                       |
| ---------------------- | ------------------------------------------------------------- | ------- | ------------------------------------------------ |
| **Redis pub/sub**      | Real-time agent coordination, swarm learning broadcasts       | <1ms    | At-most-once (acceptable for pattern broadcasts) |
| **Temporal workflows** | Migration DAG execution, retry logic, state persistence       | ~10ms   | Exactly-once (critical for migration state)      |
| **gRPC**               | Cognitive ↔ Execution direct calls (escalation, plan updates) | <5ms    | Request-response with retry                      |
| **Object storage**     | Migration snapshots, checkpoint data, large file transfer     | ~100ms  | Durable (S3/R2)                                  |

---

### Deployment Models

#### Fully Managed SaaS

All three tiers hosted and operated by Sensei. Customer provides database connection credentials. Data transits through Sensei infrastructure during migration.

**Best for:** Mid-market customers without infrastructure management teams. Fastest time-to-value.

#### Hybrid Cloud

Execution Tier deploys in the customer's VPC. Source data never leaves the customer's network. Cognitive and Meta Tiers remain in Sensei cloud — the Execution Tier sends abstract queries to Cognitive Agents and abstract patterns to Meta Agents, never raw data.

**Best for:** Organizations with data residency requirements that still want cloud-hosted intelligence.

#### Private Cloud / On-Premise

All three tiers deploy in the customer's infrastructure. LLM inference uses locally-hosted models (Llama-3 via vLLM). Pattern library and knowledge graph are customer-specific (no cross-customer learning unless explicitly opted in).

**Best for:** Regulated industries, government agencies, organizations with strict data sovereignty requirements.

#### Air-Gapped

Identical to private cloud, with no external network connectivity. All models run locally. Pattern library is initialized from a pre-built baseline and grows only from the customer's own migrations.

**Best for:** Defense, classified environments, highly regulated financial institutions.

#### Edge / Offline

AMANI client deploys on local devices (phones, tablets, laptops) with compressed models and cached knowledge. Operates autonomously for up to 30 days. Syncs with server infrastructure via CRDT-based bidirectional reconciliation when connectivity is available.

**Best for:** Frontier market deployments, field operations, intermittent connectivity environments.

---

### Resource Requirements

| Deployment Model       | Minimum Nodes    | GPU Required | Storage    | Monthly Estimate              |
| ---------------------- | ---------------- | ------------ | ---------- | ----------------------------- |
| SaaS (small migration) | Managed          | No (API)     | Managed    | Subscription-based            |
| SaaS (large migration) | Managed          | No (API)     | Managed    | Subscription-based            |
| Hybrid                 | 4 CPU (customer) | No           | 500 GB     | Customer infra + subscription |
| Private cloud          | 4 CPU + 2 GPU    | Yes (A100)   | 2 TB       | Customer infra only           |
| Air-gapped             | 4 CPU + 2 GPU    | Yes (A100)   | 2 TB       | License fee                   |
| Edge                   | 1 device         | No           | 1 GB local | Per-device license            |

---

### High Availability

- **Execution Tier:** Ray handles worker failure transparently. Failed tasks are automatically reassigned. Migration progress checkpoints every N records (configurable) ensure restartability.
- **Meta Tier:** Neo4j and vector database replicated across AZs. MongoDB with replica set. No single point of failure for knowledge assets.
- **Cognitive Tier:** Stateless agents with automatic restart. LLM API failures trigger fallback to secondary model provider or local inference.
- **Temporal workflows:** Durable execution survives node failures. Workflow state persisted to database. Automatic retry with exponential backoff.

→ [Kubernetes Deployment](kubernetes.md)
→ [Cloud-Agnostic Design](cloud-agnostic.md)
→ [Edge & Offline Deployment](edge-deployment.md)
