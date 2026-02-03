# High Availability

## Architectural Guarantees for Continuous Operation

This document describes the high availability architecture of the Sensei platform at the technology stack level. It covers multi-region deployment topology, failover mechanisms for each stateful component, health checking strategy, circuit breaker design, and graceful degradation behavior. For operational HA procedures (failover runbooks, DR drills), see [Deployment: High Availability](../../../operations/deployment/high-availability.md).

---

### Availability Targets

Sensei commits to the following availability targets measured on a rolling 30-day window:

| Tier     | Availability SLA | Monthly Downtime Budget | Measurement Scope            |
| -------- | ---------------- | ----------------------- | ---------------------------- |
| Standard | 99.9%            | 43 minutes              | Single-region, multi-AZ      |
| Enhanced | 99.95%           | 22 minutes              | Multi-region, active-passive |
| Premium  | 99.99%           | 4.3 minutes             | Multi-region, active-active  |

Availability is defined as the percentage of time during which the platform can accept new migration requests and continue processing in-flight migrations. Scheduled maintenance windows are excluded from the calculation only when announced 72 hours in advance.

---

### Multi-Region Deployment Topology

Sensei supports three deployment topologies, each providing progressively stronger availability guarantees:

**Single-region, multi-AZ (Standard).** All services are deployed across a minimum of two availability zones within a single cloud region. Stateless services run at least two replicas with pod anti-affinity rules ensuring cross-AZ distribution. Stateful services (PostgreSQL, Redis, Kafka) replicate across AZs.

**Multi-region, active-passive (Enhanced).** A primary region handles all traffic. A standby region maintains warm replicas of stateless services and asynchronous replicas of stateful services. Failover is initiated manually or by the global health monitor when the primary region is unreachable for a configurable threshold (default: 5 minutes).

**Multi-region, active-active (Premium).** Two or more regions independently serve traffic, with global load balancing routing requests based on latency. Stateful services use synchronous replication for metadata (PostgreSQL) and asynchronous replication for high-volume data (Kafka, object storage). Conflict resolution for concurrent writes follows a last-writer-wins policy with vector clock ordering.

```text
Multi-Region Active-Passive Topology
=====================================

                 ┌──────────────────────┐
                 │   Global DNS / LB    │
                 │  (Route 53 / Cloud   │
                 │   Load Balancing)    │
                 └──────────┬───────────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
     ┌────────▼────────┐       ┌──────────▼──────────┐
     │  PRIMARY REGION  │       │  STANDBY REGION     │
     │  (us-east-1)     │       │  (us-west-2)        │
     │                  │       │                     │
     │  API (active)    │       │  API (warm standby) │
     │  MABA (active)   │◄─────►│  MABA (warm standby)│
     │  KORA (active)   │  WAL  │  KORA (warm standby)│
     │  PG (primary)    │ stream│  PG (replica)       │
     │  Redis (primary) │       │  Redis (replica)    │
     │  Kafka (active)  │       │  Kafka (mirror)     │
     └─────────────────┘       └─────────────────────┘
```

---

### PostgreSQL Streaming Replication

PostgreSQL is the authoritative store for migration metadata, configuration state, and audit records. High availability is achieved through streaming replication managed by Patroni.

**Replication configuration:**

- Synchronous replication with one synchronous standby within the primary region. This ensures zero data loss for committed transactions within the region.
- Asynchronous replication to cross-region read replicas. Replication lag is monitored and alerting triggers at 60 seconds.
- Automatic failover via Patroni consensus. The Patroni cluster uses a distributed consensus store (etcd) to elect a new primary when the current primary becomes unreachable.

**Failover behavior:**

- Patroni detects primary failure within 10 seconds (configurable `ttl` and `loop_wait` parameters).
- A synchronous standby is promoted to primary. Promotion completes in under 30 seconds.
- Application connection strings use the Patroni service endpoint, which automatically resolves to the current primary. No application-level reconfiguration is required.
- In-flight transactions on the failed primary are rolled back. Applications must implement retry logic for transient connection errors.

**Connection management:**

- PgBouncer sits between application services and PostgreSQL, operating in transaction-mode pooling.
- PgBouncer health checks detect backend failures and reroute connections to the new primary.
- Maximum connection pool size is capped at 80% of `max_connections` to reserve headroom for administrative connections and replication slots.

---

### Redis Sentinel

Redis serves as the caching layer, session store, and pub/sub backbone for agent swarm communication. High availability is provided by Redis Sentinel.

**Topology:**

- One primary, two replicas, and three Sentinel instances distributed across availability zones.
- Sentinel quorum is set to 2 (a majority of the three Sentinel instances must agree before failover proceeds).

**Failover behavior:**

- Sentinel detects primary failure within the `down-after-milliseconds` threshold (default: 5,000 ms).
- Sentinel initiates a failover vote. If quorum is reached, a replica is promoted.
- Promotion completes in 10-30 seconds including client notification.
- Application clients use Sentinel-aware connection libraries that subscribe to Sentinel notifications and reconnect to the new primary automatically.

**Degradation under Redis failure:**

- Cache misses increase latency but do not cause data loss. All authoritative state resides in PostgreSQL.
- Swarm learning broadcasts are lost during the failover window. Worker agents continue processing independently; they resume swarm participation when the pub/sub channel recovers.

---

### Kafka Consumer Groups and Partition Rebalancing

Kafka provides the inter-phase communication backbone. High availability is achieved through topic replication and consumer group coordination.

**Broker topology:**

- Minimum three brokers distributed across availability zones.
- All topics configured with replication factor 3 and `min.insync.replicas` of 2.
- Unclean leader election is disabled (`unclean.leader.election.enable=false`) to prevent data loss.

**Consumer group resilience:**

- Each pipeline phase (Ingestion, MABA, KORA, Delivery) operates as a separate consumer group.
- When a consumer instance fails, the consumer group coordinator triggers a partition rebalance. Partitions previously assigned to the failed instance are reassigned to surviving instances within the group.
- Rebalance completes in under 30 seconds for groups with fewer than 100 partitions.
- Consumer offset commits use the `read_committed` isolation level to prevent processing uncommitted messages.

**Producer resilience:**

- Producers use `acks=all` to ensure that messages are replicated to all in-sync replicas before acknowledgement.
- Idempotent producer mode is enabled to prevent duplicate messages on retry.

---

### Health Checks

Sensei implements a three-tier health checking strategy:

**Liveness probes.** Kubernetes liveness probes verify that a process is running and responsive. A failed liveness probe triggers a pod restart. Liveness probes are lightweight: they return HTTP 200 from a dedicated `/healthz` endpoint without performing dependency checks.

**Readiness probes.** Kubernetes readiness probes verify that a service is ready to accept traffic. A failed readiness probe removes the pod from the service endpoint rotation. Readiness probes verify connectivity to critical dependencies (database, Redis, Kafka) with a timeout of 5 seconds.

**Deep health checks.** An internal health monitoring service performs comprehensive health checks every 30 seconds. Deep checks verify end-to-end functionality: writing a test record to PostgreSQL, publishing and consuming a test message on Kafka, and verifying Redis read/write latency. Deep check results are exposed via the `/health/deep` endpoint and feed into the alerting pipeline.

---

### Circuit Breakers

Sensei implements circuit breakers at every inter-service boundary using the standard three-state model:

- **Closed (normal).** Requests flow through. Failures are counted.
- **Open (tripped).** Requests are immediately rejected with a well-defined error. No load is placed on the downstream service. The circuit remains open for a configurable timeout (default: 30 seconds).
- **Half-open (probe).** After the timeout, a single request is allowed through. If it succeeds, the circuit closes. If it fails, the circuit reopens.

Circuit breaker thresholds:

| Dependency       | Failure Threshold | Window      | Timeout    |
| ---------------- | ----------------- | ----------- | ---------- |
| PostgreSQL       | 5 failures        | 60 seconds  | 30 seconds |
| Redis            | 10 failures       | 60 seconds  | 15 seconds |
| Kafka            | 3 failures        | 30 seconds  | 30 seconds |
| External LLM API | 5 failures        | 120 seconds | 60 seconds |
| Source/target DB | 3 failures        | 60 seconds  | 60 seconds |

Circuit breaker state transitions are logged and emitted as metrics for dashboarding and alerting.

---

### Graceful Degradation

When a non-critical subsystem becomes unavailable, Sensei degrades gracefully rather than failing entirely:

| Failed Component    | Degraded Behavior                                   | User Impact                     |
| ------------------- | --------------------------------------------------- | ------------------------------- |
| Redis               | Cache bypass; all reads go to PostgreSQL            | Increased latency (2-5x)        |
| Vector database     | New pattern matching disabled; cached patterns used | Reduced transformation accuracy |
| Neo4j               | Lineage queries unavailable; lineage writes queued  | Lineage reporting delayed       |
| External LLM API    | Fallback to local model (Llama-3) if configured     | Reduced cognitive capability    |
| Observability stack | Metrics and traces dropped; processing continues    | Monitoring blind spot           |
| AMANI channels      | Notifications queued; processing continues          | Delayed user communication      |

Degraded state is reported through the `/health/deep` endpoint, platform dashboards, and AMANI notifications. The platform automatically recovers when the failed component returns to service; no manual intervention is required for transitions out of degraded state.

---

### Availability Monitoring and Alerting

Availability is tracked through a composite Service Level Indicator (SLI) defined as:

```text
SLI = (successful_requests / total_requests) over 5-minute windows
```

Alerting thresholds:

| Condition                              | Action                       |
| -------------------------------------- | ---------------------------- |
| SLI < 99.9% over 5 minutes             | Page on-call engineer        |
| SLI < 99.5% over 15 minutes            | Escalate to engineering lead |
| SLI < 99.0% over 30 minutes            | Initiate incident response   |
| Any region unreachable for > 5 minutes | Evaluate DR failover         |

Error budget burn rate is tracked continuously. When the burn rate exceeds 2x the sustainable rate, non-critical deployments are frozen until the budget recovers.

> [Technology Stack Overview](README.md) > [Scalability](scalability.md) > [Deployment: High Availability](../../../operations/deployment/high-availability.md)
