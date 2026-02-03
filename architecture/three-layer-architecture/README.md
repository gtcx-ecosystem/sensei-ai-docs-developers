---
description: Cognitive, execution, and meta layers
icon: sitemap
---

# Three-Layer Architecture

Sensei organizes its AI agents into three layers, each operating at a different timescale and optimized for a different class of work.

---

### Layer Summary

| Layer                                | Role                         | Timescale       | Model Class                     | Examples                     |
| ------------------------------------ | ---------------------------- | --------------- | ------------------------------- | ---------------------------- |
| [L1: Cognitive](cognitive-agents.md) | Reasoning and planning       | Minutes         | Frontier (GPT-4, Claude-3)      | Scout, Architect, Resolver   |
| [L2: Execution](execution-agents.md) | Data movement and validation | Milliseconds    | Efficient (Llama-3-8B, Mistral) | Worker, Recovery, Validator  |
| [L3: Meta](meta-agents.md)           | Learning and optimization    | Weeks to months | Mixed (GPT-4 + RL models)       | Learning, Conductor, Monitor |

---

### Design Rationale

Separating agents into layers serves three purposes:

**Independent scaling.** Cognitive agents require GPU-accelerated frontier model inference but process relatively few requests. Execution agents handle millions of records per migration and scale horizontally across Ray worker pools. Meta agents run asynchronously on batch schedules. Each layer scales according to its own resource profile.

**Failure isolation.** A failure in the execution layer (e.g., a worker encountering a malformed record) does not propagate to the cognitive layer. Each layer maintains its own error handling, retry policies, and circuit breakers. Cross-layer communication occurs through well-defined message contracts on Kafka topics.

**Cost optimization.** Frontier model inference is expensive. By restricting frontier models to cognitive tasks (schema understanding, strategy design, code generation) and using efficient models for execution tasks (known transformation application, error classification), Sensei minimizes inference cost without sacrificing quality where reasoning depth is required.

---

### Communication Model

Layers communicate through two channels:

- **Kafka command topics** carry task assignments, completion events, and error reports between layers. Messages are durable, ordered, and replayable.
- **Redis shared state** provides low-latency access to migration context, agent status, and coordination signals within a layer.

Cognitive agents produce migration plans. Execution agents consume and execute those plans, reporting progress and errors. Meta agents observe completed migrations to extract patterns and refine strategies for future cognitive planning.

---

### Further Reading

- [L1: Cognitive Agents](cognitive-agents.md) — schema analysis, strategy design, code generation
- [L2: Execution Agents](execution-agents.md) — data movement, error recovery, real-time validation
- [L3: Meta Agents](meta-agents.md) — pattern extraction, strategy optimization, compound intelligence
