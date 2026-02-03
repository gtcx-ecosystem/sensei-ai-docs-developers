# Scalability

## Scaling Strategy and Performance

This document describes the scalability architecture of the Sensei platform: how each tier scales, the mechanisms that enable horizontal growth, and the performance characteristics operators should expect at various scale points.

---

### Scaling Philosophy

Sensei scales horizontally at every tier. Vertical scaling (larger instances) is used only for stateful components where horizontal partitioning introduces unacceptable coordination overhead (e.g., PostgreSQL primary). All stateless components -- API servers, MABA transformation workers, KORA verification workers, and AMANI interface nodes -- are designed to scale by adding instances with no configuration changes and no downtime.

The scaling strategy is summarized as: **stateless compute, partitioned data, pooled connections, elastic infrastructure**.

---

### Stateless API Tier

The API tier is a fleet of identical, stateless Node.js/TypeScript processes behind a Layer 7 load balancer. Each instance:

- Retrieves configuration from environment variables and Kubernetes ConfigMaps.
- Authenticates requests by validating tokens against the control plane's auth service.
- Reads and writes persistent state through PostgreSQL (via PgBouncer) and Redis.
- Publishes events to Kafka.
- Holds no in-memory state between requests.

**Scaling mechanism:** Kubernetes Horizontal Pod Autoscaler (HPA) scales the API deployment based on CPU utilization (target: 70%) and request rate (custom metric via Prometheus adapter). Scale-up latency is typically 15-30 seconds, dominated by container image pull time on cold nodes. Warm nodes with pre-cached images scale in under 5 seconds.

**Scaling limits:** The API tier itself has no inherent scaling ceiling. The practical limit is determined by the capacity of downstream dependencies: PostgreSQL connection pool size, Kafka partition count, and Redis throughput.

```yaml
# API tier autoscaling configuration
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: sensei-api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sensei-api
  minReplicas: 3
  maxReplicas: 50
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: '500'
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Pods
          value: 4
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Pods
          value: 2
          periodSeconds: 120
```

---

### Kafka Partitioning for Parallelism

Kafka is the primary mechanism for distributing work across transformation and verification workers. Throughput scales linearly with partition count up to the number of available consumers.

**Partitioning strategy:**

- Migration-level topics (e.g., `ingestion.complete`, `transformation.complete`) are partitioned by migration ID. This ensures that all events for a single migration are processed in order by a single consumer, while permitting parallel processing across migrations.
- Data-level topics (e.g., `partition.data`) are partitioned by table and partition key. This enables fine-grained parallelism within a single migration: multiple workers process different tables or data ranges concurrently.

**Partition sizing guidelines:**

| Migration Scale                        | Partitions per Data Topic | Expected Consumer Count |
| -------------------------------------- | ------------------------- | ----------------------- |
| Small (< 50 tables, < 10M rows)        | 12                        | 4-8                     |
| Medium (50-500 tables, 10M-1B rows)    | 48                        | 16-32                   |
| Large (500+ tables, 1B+ rows)          | 128                       | 32-64                   |
| Enterprise (multi-database, 10B+ rows) | 256                       | 64-128                  |

Partition count is configured at topic creation and cannot be reduced. Operators should provision for expected peak workload.

**Throughput characteristics:** Each Kafka partition sustains approximately 10 MB/s write throughput with `acks=all` and replication factor 3. A 128-partition topic supports approximately 1.28 GB/s aggregate write throughput, which exceeds the practical ingestion rate for most migration workloads.

---

### Worker Pool Auto-Scaling

MABA transformation workers and KORA verification workers run as Ray tasks on a dynamically sized Ray cluster. Ray's autoscaler manages the worker pool independently of the Kubernetes API tier autoscaler.

**Scaling mechanism:**

- Ray monitors the pending task queue depth and resource utilization across the cluster.
- When the pending task queue exceeds the cluster's processing capacity, Ray requests additional worker nodes from Kubernetes.
- Kubernetes provisions new pods (and, if using cluster autoscaler, new nodes) to satisfy the resource request.
- When the task queue drains and utilization falls below the scale-down threshold (default: 20%), Ray releases idle worker nodes after a cooldown period (default: 5 minutes).

**Worker node profiles:**

| Worker Type   | CPU      | Memory | GPU     | Use Case                                     |
| ------------- | -------- | ------ | ------- | -------------------------------------------- |
| MABA Standard | 8 cores  | 32 Gi  | 0       | SQL/Python transformation execution          |
| MABA Heavy    | 16 cores | 64 Gi  | 0       | Large table transformations, complex joins   |
| KORA Standard | 8 cores  | 16 Gi  | 0       | Behavioral equivalence verification          |
| LLM Inference | 8 cores  | 32 Gi  | 1x A100 | Local model serving (air-gapped deployments) |

**Scaling bounds:**

- Minimum worker count: 2 (ensures processing continuity during scale events).
- Maximum worker count: 128 (configurable; constrained by cluster resource quotas).
- Scale-up increment: up to 8 workers per scaling event.
- Scale-down increment: up to 4 workers per scaling event (conservative to avoid premature reclamation).

---

### Database Connection Pooling

PostgreSQL connection management is the most common bottleneck in horizontally scaled database-backed services. Sensei addresses this through PgBouncer operating in transaction-mode pooling.

**Architecture:**

- PgBouncer runs as a sidecar container alongside each API pod and as a standalone deployment for the worker tier.
- Each PgBouncer instance maintains a pool of persistent connections to the PostgreSQL primary and read replicas.
- Application code acquires a connection for the duration of a single transaction, then returns it to the pool. Connections are not held across request boundaries.

**Pool sizing:**

| Component    | Max Pool Size per Instance | Expected Instances | Effective Connection Demand |
| ------------ | -------------------------- | ------------------ | --------------------------- |
| API Server   | 10                         | 3-50               | 30-500                      |
| MABA Worker  | 5                          | 2-64               | 10-320                      |
| KORA Worker  | 3                          | 2-32               | 6-96                        |
| Orchestrator | 5                          | 2                  | 10                          |

PgBouncer aggregates these into a shared pool of at most 200 backend connections to PostgreSQL (configurable). This allows hundreds of application instances to coexist without exceeding PostgreSQL's `max_connections` limit.

---

### Read Replicas

Read-heavy workloads are distributed across PostgreSQL read replicas to reduce load on the primary:

- **Reporting queries** (migration status dashboards, audit log retrieval) are routed exclusively to read replicas.
- **AMANI conversation history** reads are served from read replicas with a staleness tolerance of 5 seconds.
- **Schema discovery** metadata reads during the ingestion phase use read replicas when the metadata is not expected to change during the read window.

Write operations and strongly consistent reads are always routed to the primary. The application's data access layer uses a connection router that directs queries based on annotated read/write intent.

**Replica scaling:** Read replicas are added or removed based on read query latency. When p95 read latency exceeds 100ms, a new replica is provisioned. When p95 read latency falls below 20ms across all replicas, a replica is decommissioned. Minimum replica count is 2 for high availability.

---

### CDN for Static Assets

The AMANI web interface and platform dashboard serve static assets (JavaScript bundles, CSS, images, fonts) through a CDN (CloudFront or Cloudflare).

**Caching policy:**

- Immutable assets (hashed filenames) are cached with `Cache-Control: public, max-age=31536000, immutable`.
- HTML entry points are cached with `Cache-Control: public, max-age=300` and revalidated on deployment.
- API responses are never cached at the CDN layer.

**Impact on scalability:** CDN offloading eliminates static asset traffic from the API tier, reducing its request rate by approximately 60-80% for dashboard-heavy usage patterns. This permits the API tier to allocate its full capacity to migration API calls.

---

### Performance Benchmarks

The following benchmarks were measured on a standard deployment (3 API instances, 16 MABA workers, 8 KORA workers, 48 Kafka partitions, 1 PostgreSQL primary + 2 read replicas) in the us-east-1 region:

| Metric                                      | Value           | Conditions                             |
| ------------------------------------------- | --------------- | -------------------------------------- |
| API request throughput                      | 2,400 req/s     | Mixed read/write, p50 payload 2 KB     |
| API response latency (p50)                  | 12 ms           | Cached metadata reads                  |
| API response latency (p99)                  | 85 ms           | Uncached reads with PG round-trip      |
| Ingestion throughput                        | 150 MB/s        | Bulk extract from PostgreSQL source    |
| Transformation throughput                   | 500,000 rows/s  | Simple type-mapped transformations     |
| Transformation throughput                   | 50,000 rows/s   | Complex transformations with LLM calls |
| Verification throughput                     | 1,000 queries/s | Behavioral equivalence query replay    |
| Delivery throughput                         | 200,000 rows/s  | Batched inserts to PostgreSQL target   |
| End-to-end migration (100M rows, 50 tables) | 45 minutes      | Standard configuration                 |
| End-to-end migration (1B rows, 200 tables)  | 6 hours         | Enhanced configuration with 64 workers |

These figures represent sustained throughput, not burst capacity. Burst capacity is approximately 2x sustained throughput for up to 5 minutes, limited by autoscaler response time.

---

### Scaling Limits and Bottlenecks

| Resource                         | Practical Limit        | Mitigation                                 |
| -------------------------------- | ---------------------- | ------------------------------------------ |
| PostgreSQL write throughput      | ~10,000 TPS            | Batch writes, reduce write amplification   |
| PostgreSQL connection count      | 500 connections        | PgBouncer pooling                          |
| Kafka partition count per topic  | 256                    | Sufficient for enterprise-scale migrations |
| Ray cluster size                 | 128 workers            | Cluster autoscaler node provisioning       |
| LLM API rate limits              | Provider-dependent     | Request queuing, fallback to local models  |
| Object storage (S3) request rate | 5,500 PUT/s per prefix | Prefix partitioning by migration ID        |

Operators encountering these limits should consult the capacity planning guide for mitigation strategies specific to their deployment.

> [Technology Stack Overview](README.md) > [High Availability](high-availability.md) > [Data Flow](data-flow.md)
