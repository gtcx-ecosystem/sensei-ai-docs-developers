# Kubernetes Deployment

## Production-Grade Container Orchestration

Sensei deploys as a Kubernetes-native application. This page covers the specific K8s configuration, resource allocation, and operational considerations.

---

### Namespace Architecture

```text
sensei-system/
├── cognitive/          # GPU-equipped pods for frontier model inference
│   ├── scout-agents
│   └── architect-agents
├── execution/          # CPU-optimized pods with Ray auto-scaling
│   ├── ray-head
│   ├── ray-workers     # 4-128 pods, auto-scaled
│   └── validator-agents
├── meta/               # Persistent services
│   ├── learning-agents
│   ├── neo4j
│   ├── mongodb
│   └── mlflow
├── messaging/          # Communication infrastructure
│   ├── redis
│   └── temporal
└── gateway/            # External access
    ├── amani-web
    ├── amani-api
    └── monitoring (Grafana)
```

---

### Node Pools

| Pool          | Instance Type               | GPU          | Count | Purpose                               |
| ------------- | --------------------------- | ------------ | ----- | ------------------------------------- |
| **cognitive** | p3.2xlarge / a2-highgpu-1g  | 1× V100/A100 | 2-8   | Frontier LLM inference (local models) |
| **execution** | c5.4xlarge / c2-standard-16 | None         | 4-128 | Ray workers, transformation execution |
| **meta**      | r5.2xlarge / n2-highmem-8   | None         | 2-4   | Databases, persistent services        |
| **system**    | m5.xlarge / e2-standard-4   | None         | 2-3   | Redis, Temporal, monitoring, ingress  |

**Note:** The cognitive pool is only required for on-premise/air-gapped deployments that run local LLMs. SaaS and hybrid deployments use API-based inference and can skip GPU nodes entirely.

---

### Auto-Scaling Configuration

#### Ray Worker Auto-Scaling

Ray manages its own worker scaling within the execution node pool:

```yaml
# Simplified Ray cluster config
ray_cluster:
  head:
    resources:
      cpu: 4
      memory: 8Gi
  workers:
    min_replicas: 4
    max_replicas: 128
    resources_per_worker:
      cpu: 8
      memory: 32Gi
    scaling:
      trigger: queue_depth > 100 pending tasks
      scale_up_speed: 4 workers per minute
      scale_down_delay: 300 seconds (drain before termination)
```

#### Kubernetes Horizontal Pod Autoscaler

For non-Ray components:

```yaml
# AMANI web interface
hpa:
  min_replicas: 2
  max_replicas: 20
  target_cpu_utilization: 70%
  target_memory_utilization: 80%

# Validator agents
hpa:
  min_replicas: 2
  max_replicas: 16
  target_cpu_utilization: 75%
```

---

### Storage

| Component             | Storage Type                | Size          | Access Mode   |
| --------------------- | --------------------------- | ------------- | ------------- |
| Neo4j                 | Persistent Volume (SSD)     | 100-500 GB    | ReadWriteOnce |
| MongoDB               | Persistent Volume (SSD)     | 200 GB - 1 TB | ReadWriteOnce |
| Redis                 | In-memory + AOF persistence | 16-64 GB      | ReadWriteOnce |
| Temporal              | Persistent Volume (SSD)     | 50-200 GB     | ReadWriteOnce |
| MLflow artifacts      | S3/GCS/R2                   | Unlimited     | ReadWriteMany |
| Migration checkpoints | S3/GCS/R2                   | Per-migration | ReadWriteMany |

---

### Networking

**Internal:** All inter-service communication uses Kubernetes service mesh (Istio or Linkerd) with mTLS encryption. No unencrypted traffic between pods.

**External ingress:**

- AMANI web interface: HTTPS (port 443) via ingress controller
- AMANI API: HTTPS (port 443) via ingress controller
- Grafana monitoring: HTTPS (port 443), restricted by IP allowlist
- Database connections (source/target): Encrypted tunnels via dedicated egress pods

**Network policies:** Strict pod-to-pod policies ensure:

- Execution pods cannot reach external networks directly
- Only gateway pods accept external connections
- Meta tier pods are not externally accessible
- Database pods only accept connections from their designated clients

---

### Resource Estimates by Migration Size

| Migration Scale     | Node Count | Total vCPU | Total RAM | Monthly Infra Cost (AWS) |
| ------------------- | ---------- | ---------- | --------- | ------------------------ |
| Small (<1M records) | 8          | 64         | 128 GB    | ~$2,000                  |
| Medium (1-10M)      | 16         | 192        | 512 GB    | ~$6,000                  |
| Large (10-100M)     | 40         | 640        | 1.5 TB    | ~$18,000                 |
| Very Large (>100M)  | 80+        | 1,280+     | 3+ TB     | ~$40,000+                |

These estimates assume SaaS deployment with API-based LLM inference. On-premise deployments with local GPU inference add ~$5,000-15,000/month for GPU node costs.

---

### High Availability

- **Multi-AZ deployment:** Pods distributed across availability zones
- **Pod disruption budgets:** Critical services maintain minimum replica counts during rolling updates
- **Liveness/readiness probes:** All pods have health checks; unhealthy pods are automatically replaced
- **Persistent volume replication:** Database volumes replicated across AZs
- **Temporal durability:** Workflow state persists through any infrastructure failure

→ [Cloud-Agnostic Design](cloud-agnostic.md)
→ [Deployment Architecture Overview](deployment.md)
