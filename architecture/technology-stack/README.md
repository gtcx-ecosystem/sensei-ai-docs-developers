---
description: Production infrastructure components
icon: desktop-arrow-down
---

# Technology Stack

## Production-Ready Composition

Every component in Sensei's stack is commercially available and production-tested. The innovation is not in individual technologies but in their composition — how they're wired together to produce compound migration intelligence.

---

### Stack Overview

```text
┌──────────────────────────────────────────────────────────┐
│                    INFRASTRUCTURE                         │
│  Kubernetes (EKS/GKE) · AWS Lambda · NVIDIA A100 · S3    │
├──────────────────────────────────────────────────────────┤
│                    OBSERVABILITY                          │
│  OpenTelemetry · Grafana · Datadog · MLflow · W&B        │
├──────────────────────────────────────────────────────────┤
│                    EXECUTION                              │
│  Ray 2.0 · Temporal/Prefect · Redis pub/sub · RabbitMQ   │
├──────────────────────────────────────────────────────────┤
│                    MEMORY & LEARNING                      │
│  Pinecone/Weaviate · Neo4j · MongoDB · Feast             │
├──────────────────────────────────────────────────────────┤
│                    AGENT FRAMEWORKS                       │
│  LangChain + LangGraph · AutoGen · CrewAI                │
├──────────────────────────────────────────────────────────┤
│                    LLM LAYER                              │
│  GPT-4-turbo · Claude-3 · Llama-3-70B · BERT (fine-tuned)│
└──────────────────────────────────────────────────────────┘
```

---

### LLM Layer

| Role                      | Technology              | Purpose                                                           |
| ------------------------- | ----------------------- | ----------------------------------------------------------------- |
| **Primary (Cognitive)**   | GPT-4-turbo / Claude-3  | Schema analysis, code generation, strategic reasoning             |
| **Secondary (Execution)** | Llama-3-8B via vLLM     | Fast task execution, error classification, pattern matching       |
| **Validation**            | Fine-tuned BERT         | Semantic validation, anomaly detection, compliance classification |
| **Embeddings**            | OpenAI ada-002 / Cohere | Pattern encoding for vector similarity search                     |
| **Local inference**       | Llama-3-70B via vLLM    | Air-gapped deployments, data sovereignty requirements             |

The LLM layer is **pluggable** — customers can configure which models serve which roles based on their security requirements, cost constraints, and deployment environment. Organizations that cannot send data to external APIs can run entirely on local models with Llama-3-70B handling cognitive tasks and Llama-3-8B handling execution.

---

### Agent Frameworks

| Framework                 | Role                                             | Why This Choice                                                                                                                           |
| ------------------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **LangChain + LangGraph** | Agent lifecycle, tool binding, memory management | Most mature agent framework. LangGraph adds stateful multi-step workflows with cycles and branching. Production-deployed at Fortune 500s. |
| **AutoGen (Microsoft)**   | Multi-agent collaborative conversations          | Purpose-built for agents that need to reason together. Scout + Architect agent conversations use AutoGen's structured dialogue protocol.  |
| **CrewAI**                | Simpler workflow orchestration                   | Used for straightforward sequential agent pipelines where AutoGen's full conversation protocol is overhead.                               |

---

### Memory & Learning

| Technology              | Purpose                                                        | Why                                                                                                             |
| ----------------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Pinecone / Weaviate** | Vector database for pattern storage and similarity search      | Sub-millisecond similarity search at scale. Pinecone for managed, Weaviate for self-hosted.                     |
| **Neo4j**               | Graph database for pattern relationships and knowledge graph   | Enables structural reasoning about migration similarity — not just vector proximity but relationship traversal. |
| **MongoDB**             | Document store for conversation logs, migration execution logs | Flexible schema for heterogeneous log data. Full-text search for log analysis.                                  |
| **Feast**               | Feature store for ML features                                  | Standardized feature serving for prediction models. Time-travel feature retrieval for model training.           |

---

### Execution

| Technology             | Purpose                                                       | Why                                                                                                                                                            |
| ---------------------- | ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Ray 2.0**            | Distributed computing, worker scaling, reinforcement learning | Auto-scales from 4 to 128 workers. Ray RLlib powers evolutionary strategy optimization. Ray Serve handles model inference. Unified framework for compute + ML. |
| **Temporal / Prefect** | Workflow engine for migration DAG execution                   | Durable execution with automatic retry, timeout handling, and state persistence. Migrations survive node failures without data loss.                           |
| **Redis pub/sub**      | Swarm communication, pattern broadcast                        | Sub-millisecond message propagation for real-time agent coordination. The backbone of in-migration swarm learning.                                             |
| **RabbitMQ**           | Task queue for durable job distribution                       | When guaranteed delivery matters more than speed. Used for cross-migration async tasks.                                                                        |

---

### Observability

| Technology            | Purpose                                    | Why                                                                                                                                                                      |
| --------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **OpenTelemetry**     | Distributed tracing, metrics collection    | Vendor-neutral telemetry. Instruments every agent interaction, transformation step, and validation check. Powers the Performance Optimizer Agent's bottleneck diagnosis. |
| **Grafana / Datadog** | Dashboards, alerting, real-time monitoring | Visual migration progress. SLA tracking. Anomaly alerting. Customer-facing dashboards.                                                                                   |
| **MLflow**            | Experiment tracking, model versioning      | Tracks every strategy evolution cycle. Version-controls pattern library updates. Reproducible model evaluation.                                                          |
| **Weights & Biases**  | Training runs, hyperparameter optimization | Used by Meta Agents for reinforcement learning experiment tracking and strategy fitness visualization.                                                                   |

---

### Infrastructure

| Technology               | Purpose                               | Why                                                                                                                                      |
| ------------------------ | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Kubernetes (EKS/GKE)** | Container orchestration, auto-scaling | Three-tier deployment: GPU nodes for Cognitive tier, CPU nodes for Execution tier, persistent services for Meta tier.                    |
| **AWS Lambda**           | Serverless burst capacity             | Spawns ephemeral agents for spike workloads without maintaining idle compute.                                                            |
| **NVIDIA A100**          | GPU inference for local model serving | Required for Llama-3-70B inference in air-gapped or sovereignty-restricted deployments. Batch inference for Meta Agent pattern analysis. |
| **S3 / CloudFlare R2**   | Object storage                        | Migration snapshots, temporal checkpoint data, pattern library backups. R2 for egress-cost-sensitive deployments.                        |

---

### KORA-Specific Stack

KORA's verification engine has its own technology choices driven by performance and security requirements:

| Technology             | Purpose                  | Why                                                                                                                                                 |
| ---------------------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Rust**               | Core verification engine | Memory safety without GC pauses. Zero-cost abstractions for cryptographic operations. Ownership model prevents data races in concurrent validation. |
| **ring crate**         | Cryptographic operations | Constant-time implementations resistant to timing side-channel attacks. Used for Behavioral Equivalence Certificate signing.                        |
| **Great Expectations** | Rule-based validation    | Standard validation framework extended with Sensei-specific custom expectations. Community ecosystem of validation rules.                           |

---

### Deployment Flexibility

The stack is designed for multiple deployment models:

| Model                  | Configuration                                                  | Use Case                                 |
| ---------------------- | -------------------------------------------------------------- | ---------------------------------------- |
| **Fully managed SaaS** | All components hosted by Sensei                                | Default for mid-market customers         |
| **Hybrid**             | Execution tier in customer VPC, Cognitive/Meta in Sensei cloud | Data sovereignty with cloud intelligence |
| **Private cloud**      | All components in customer infrastructure                      | Regulated industries, government         |
| **Air-gapped**         | All components on-premise, local LLMs only                     | Defense, classified environments         |
| **Edge**               | AMANI client on local device, sync when connected              | Frontier market, low-connectivity        |

→ [Deployment Architecture](../deployment.md) → [Agent Communication Protocol](../agent-protocol.md)
