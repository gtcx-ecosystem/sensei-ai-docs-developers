---
description: Core principles behind Sensei's design
---

# Design Philosophy

## Foundational Design Principles

This document describes the core architectural principles that govern the design of the Sensei platform. These principles are not aspirational; they are enforced through code review gates, automated architecture tests, and CI/CD policy checks. Every subsystem -- MABA, KORA, and AMANI -- is expected to conform to these constraints.

---

### Separation of Concerns: Data Plane vs. Control Plane

Sensei maintains a strict boundary between the **data plane** (where migration data flows) and the **control plane** (where orchestration decisions are made). This separation is modeled after the architecture of modern network infrastructure and container orchestration systems.

**Data plane** components handle ingestion, transformation, verification, and delivery of migration payloads. They are optimized for throughput, operate on stateless worker nodes, and scale horizontally by adding partitions or replicas. Data plane components never make scheduling or routing decisions.

**Control plane** components handle workflow orchestration, agent coordination, resource allocation, and policy enforcement. They are optimized for consistency and correctness, operate on persistent nodes with durable state, and scale vertically or through leader election. Control plane components never touch raw migration data.

This boundary is enforced at the network level. Data plane pods reside in a dedicated subnet and communicate with the control plane exclusively through well-defined gRPC interfaces and Kafka topic contracts. No control plane service has read access to data plane storage volumes.

---

### Agent-Based Architecture Rationale

Sensei decomposes migration work into autonomous agents rather than monolithic pipeline stages. The rationale is threefold:

1. **Fault isolation.** When an agent fails, its failure is contained. The orchestrator reassigns its work unit to another agent of the same type. Monolithic pipeline stages propagate failures across the entire execution path.

2. **Heterogeneous scaling.** Different migration phases have different resource profiles. Scout agents are LLM-bound; Worker agents are I/O-bound; Validator agents are CPU-bound. An agent-based model permits independent scaling of each class, matching resource allocation to actual demand rather than provisioning for peak across all phases.

3. **Composability.** Agents communicate through message contracts, not shared memory. New agent types can be introduced without modifying existing agents. This permits the platform to evolve its capabilities -- adding a new connector type, a new validation strategy, or a new optimization heuristic -- without destabilizing the core execution path.

The agent model imposes overhead: message serialization, coordination latency, and the complexity of distributed state. Sensei accepts this overhead as the cost of a system that can be maintained and extended over a multi-year horizon by a distributed engineering team.

---

### Immutability of Verification Artifacts

All artifacts produced by the KORA verification engine are immutable once signed. This includes Behavioral Equivalence Certificates, temporal checkpoint snapshots, validation evidence bundles, and audit trail records.

Immutability is enforced at multiple levels:

- **Cryptographic signing.** KORA signs all verification artifacts using Ed25519 keys managed by the platform's credential vault. Any modification to a signed artifact invalidates the signature.
- **Append-only storage.** Verification artifacts are written to append-only object storage with versioning enabled. The storage backend does not expose a delete or overwrite API to KORA processes.
- **Content-addressable references.** Internal references to verification artifacts use content hashes (SHA-256), not mutable identifiers. If the content changes, the hash changes, and all downstream references break explicitly rather than silently pointing to modified data.

This design ensures that verification results produced at time T remain trustworthy at time T+N, regardless of subsequent system state changes, software upgrades, or operator actions.

---

### Defense in Depth

Sensei does not rely on any single security boundary. Security controls are layered such that the compromise of one layer does not grant unrestricted access to the system.

The layers, from outermost to innermost:

1. **Network perimeter.** WAF, DDoS protection, and IP allowlisting at the edge.
2. **Transport encryption.** TLS 1.3 for all inter-service communication; mTLS between microservices within the cluster.
3. **Authentication.** API key validation and OIDC token verification at the API gateway.
4. **Authorization.** RBAC policy evaluation at each service boundary, enforced by the control plane's policy engine.
5. **Data encryption.** AES-256-GCM encryption at rest for all persistent storage volumes.
6. **Audit logging.** Tamper-evident audit logs for all state-changing operations, shipped to immutable storage.
7. **Runtime isolation.** Container sandboxing, read-only filesystem mounts, and seccomp profiles for all workload pods.

Each layer operates independently. The failure or misconfiguration of one layer degrades the security posture but does not eliminate it.

---

### Fail-Safe Defaults

When a Sensei component encounters an ambiguous or undefined condition, the default behavior is to deny, halt, or request human intervention -- never to proceed optimistically.

Concrete applications of this principle:

- **Schema mapping ambiguity.** If MABA's confidence score for an auto-generated mapping falls below the configured threshold (default: 0.92), the mapping is flagged for human review. It is never applied silently.
- **Verification failure.** If KORA cannot reach a definitive pass/fail determination for a behavioral equivalence check, the check is recorded as INCONCLUSIVE and the migration does not proceed to the delivery phase.
- **Credential expiry.** If a database credential approaches its TTL without successful rotation, the affected connection pool is drained rather than continuing to operate with potentially stale credentials.
- **Circuit breaker activation.** When a downstream dependency exceeds its error rate threshold, the circuit breaker opens and returns an explicit error to callers. Requests are not queued indefinitely or retried without backoff.

The intent is to make failure visible and recoverable, rather than silent and corrosive.

---

### Principle of Least Privilege

Every component, agent, and service account in the Sensei platform operates with the minimum set of permissions required for its function.

- **Agent permissions.** Scout agents have read-only access to source system metadata. Worker agents have read access to source data and write access to a scoped target staging area. No agent has administrative access to the platform's own infrastructure.
- **Service accounts.** Each microservice authenticates with a dedicated service account whose permissions are scoped to its operational requirements. The API service can read and write to the metadata database; it cannot access the vector database directly.
- **Kubernetes RBAC.** Workload pods run with non-root users, drop all Linux capabilities except those explicitly required, and mount only the specific secrets and config maps they reference.
- **Database roles.** The application connects to PostgreSQL through role-separated credentials: a read-write role for the API tier, a read-only role for the reporting tier, and a migration role for schema changes that is never used by application code.

Permissions are defined declaratively in infrastructure-as-code and audited on every deployment.

---

### Horizontal Scalability by Design

Sensei is architected to scale by adding instances, not by enlarging them. This constraint influences every design decision from data modeling to session management.

**Stateless compute.** All API servers, MABA transformation workers, and KORA verification workers are stateless. They read configuration from environment variables, retrieve state from external stores (PostgreSQL, Redis), and produce outputs to message queues (Kafka) or object storage (S3). Any instance can handle any request; no session affinity is required.

**Partitioned data processing.** Migration workloads are partitioned by table, schema, or data range. Each partition is processed independently by a separate worker. Kafka topic partitioning ensures that related records are processed by the same consumer within a partition, while permitting parallel processing across partitions.

**Shared-nothing workers.** Worker agents do not communicate with each other during normal operation. Swarm learning broadcasts via Redis pub/sub are advisory: they improve performance but are not required for correctness. If the pub/sub channel is unavailable, workers continue processing independently.

**Connection pooling.** Database connections are managed through PgBouncer in transaction-mode pooling, allowing hundreds of application instances to share a bounded set of database connections. This eliminates the connection-count ceiling that typically limits horizontal scaling of database-backed services.

**Infrastructure elasticity.** Kubernetes Horizontal Pod Autoscaler (HPA) and Ray autoscaler independently scale the API tier and compute tier based on CPU utilization, queue depth, and custom metrics. Scale-up latency is bounded by container image pull time (mitigated by image caching on warm nodes). Scale-down respects a configurable cooldown period to avoid thrashing.

---

### Summary of Governing Principles

| Principle                     | Enforcement Mechanism                      | Violation Response                  |
| ----------------------------- | ------------------------------------------ | ----------------------------------- |
| Data/control plane separation | Network policy, gRPC contracts             | CI pipeline rejection               |
| Agent fault isolation         | Message-based communication                | Automatic work reassignment         |
| Verification immutability     | Cryptographic signing, append-only storage | Signature validation failure        |
| Defense in depth              | Layered security controls                  | Degraded posture, not total failure |
| Fail-safe defaults            | Confidence thresholds, circuit breakers    | Halt and escalate                   |
| Least privilege               | RBAC, scoped service accounts              | Permission denied errors            |
| Horizontal scalability        | Stateless design, partitioned processing   | Autoscaler-managed capacity         |

These principles are the architectural invariants of the Sensei platform. They constrain the solution space for all engineering decisions and are not subject to case-by-case exception.

> [Architecture Overview](README.md) > [Technology Stack](technology-stack/) > [Security Architecture](technology-stack/security-architecture.md)
