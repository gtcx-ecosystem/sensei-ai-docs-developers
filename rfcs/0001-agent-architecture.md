# RFC 0001: Agent Architecture

## Summary

This RFC proposes a three-layer agent architecture for the Sensei platform: a Cognitive Layer for reasoning and planning, an Execution Layer for task performance, and a Meta Layer for system-wide coordination and learning.

## Status

Accepted

## Authors

Sensei Architecture Team

## Date

2025-07-15

---

## Problem Statement

Enterprise data migration requires coordinated execution of heterogeneous tasks: schema analysis, transformation generation, data movement, error recovery, validation, and stakeholder communication. These tasks vary in computational profile, latency requirements, and autonomy level. A monolithic orchestration engine cannot efficiently serve this diversity. A flexible agent-based architecture is required to decompose migration work into independently schedulable, recoverable, and observable units of execution.

---

## Proposed Design

### Layer 1: Cognitive Layer

Agents responsible for reasoning, planning, and decision-making.

| Agent               | Responsibility                                                                                                                                              |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Scout Agent**     | Analyzes source schemas, infers semantic meaning from naming patterns and data distributions, produces annotated schema representations.                    |
| **Architect Agent** | Generates migration plans as executable DAGs including table ordering, transformation specs, parallelism configuration, and rollback strategies.            |
| **Resolver Agent**  | Handles ambiguous mappings below confidence threshold. Generates alternatives, ranks them, and either selects the best option or escalates to human review. |

Cognitive agents are stateless between invocations. State is persisted in the migration plan object in PostgreSQL, permitting horizontal scaling and restart recovery.

### Layer 2: Execution Layer

Agents responsible for data movement, transformation, and verification.

| Agent               | Responsibility                                                                                                                                                   |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Worker Agent**    | Executes transformation tasks: extract, transform, load. Operates on batches with checkpoint support.                                                            |
| **Recovery Agent**  | Diagnoses worker failures, consults the pattern library, generates fixes, tests against sample data, and applies corrections. Unresolvable errors are escalated. |
| **Validator Agent** | Executes KORA verification: captures source snapshots, replays queries against target, compares behavioral outputs.                                              |

Execution agents are long-running and stateful. State is persisted to Redis for fast recovery and PostgreSQL for durability.

### Layer 3: Meta Layer

Agents operating across individual migrations for coordination and learning.

| Agent               | Responsibility                                                                                                                          |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Conductor Agent** | Orchestrates migration lifecycle: dispatches Cognitive agents, schedules Execution agents, manages resources, enforces SLA constraints. |
| **Learning Agent**  | Extracts generalizable patterns from completed migrations and updates the global pattern library with differential privacy guarantees.  |
| **Monitor Agent**   | Observes system metrics and triggers adaptive responses: scaling workers, rebalancing partitions, alerting operators.                   |

---

### Agent Lifecycle

```text
INITIALIZING -> READY -> RUNNING -> COMPLETING -> COMPLETED
                  |         |
                  |         +-> FAILING -> FAILED
                  |         |
                  |         +-> PAUSING -> PAUSED -> RUNNING
                  |
                  +-> CANCELLED
```

State transitions are recorded as events on per-migration Kafka topics for audit. Terminal transitions are also persisted to PostgreSQL.

---

### Communication Patterns

**Command channels (Kafka).** The Conductor publishes task assignments to per-migration topics. Execution agents consume, process, and publish completion or failure events. At-least-once delivery with idempotent processing.

**Shared state (Redis).** Real-time coordination -- progress counters, batch identifiers, resource locks -- uses atomic Redis operations (MULTI/EXEC, Lua scripts) to prevent race conditions.

Cross-layer communication follows strict hierarchy: Meta directs Cognitive, Cognitive produces artifacts for Execution, Execution reports upward. No circular dependencies.

---

### Failure Semantics

| Severity        | Behavior                                                       |
| --------------- | -------------------------------------------------------------- |
| **Transient**   | Automatic retry with exponential backoff (max 3 attempts).     |
| **Recoverable** | Recovery Agent diagnoses, generates fix, retests, and applies. |
| **Escalatable** | Agent pauses, records diagnostics, notifies operators.         |
| **Fatal**       | Migration halted, rollback initiated if configured.            |

---

## Implementation Notes

- TypeScript orchestration agents: `packages/api/src/agents/`
- Rust verification agents: `packages/kora-core/src/agents/`
- Protocol buffers: `packages/shared/proto/agents.proto`
- Differential privacy: `packages/ml-services/privacy/`

## References

- ADR-0001: Monorepo Structure
- ADR-0002: Language Choices
- ADR-0003: Database Selection
- RFC-0002: Verification Protocol
