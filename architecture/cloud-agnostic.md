# Cloud-Agnostic Design

## Run Anywhere

Sensei is designed to avoid cloud vendor lock-in. Every component has equivalents across major providers, and the platform abstracts cloud-specific services behind portable interfaces.

---

### Abstraction Layers

The platform uses four abstraction layers to maintain cloud portability:

#### 1. Compute Abstraction

Kubernetes is the universal compute layer. Any conformant K8s distribution works:

- **AWS:** EKS
- **GCP:** GKE
- **Azure:** AKS
- **On-premise:** Rancher, OpenShift, or vanilla K8s
- **Edge:** k3s for lightweight deployments

No cloud-specific compute services (AWS Lambda, GCP Cloud Functions) are used in the core platform. Burst capacity uses Ray's built-in auto-scaling rather than serverless functions.

#### 2. Storage Abstraction

Object storage is accessed through an S3-compatible API:

- **AWS:** S3 (native)
- **GCP:** GCS with S3-compatible endpoint
- **Azure:** Blob Storage with S3 adapter
- **On-premise:** MinIO
- **Multi-cloud:** CloudFlare R2 (no egress fees)

Persistent volumes use the Kubernetes CSI (Container Storage Interface), which maps to EBS, Persistent Disk, Azure Disk, or local NVMe as appropriate.

#### 3. LLM Abstraction

The LLM layer is pluggable — the platform defines an interface, and concrete implementations connect to different providers:

| Provider      | Cognitive Tier     | Execution Tier    | Offline Tier |
| ------------- | ------------------ | ----------------- | ------------ |
| OpenAI API    | GPT-4-turbo        | GPT-3.5-turbo     | N/A          |
| Anthropic API | Claude-3 Opus      | Claude-3 Haiku    | N/A          |
| Local (vLLM)  | Llama-3-70B        | Llama-3-8B        | Llama-3-8B   |
| Azure OpenAI  | GPT-4 (Azure)      | GPT-3.5 (Azure)   | N/A          |
| AWS Bedrock   | Claude via Bedrock | Llama via Bedrock | N/A          |

Customers choose their LLM provider based on existing contracts, data sovereignty requirements, and cost constraints. Switching providers requires configuration change, not code change.

#### 4. Database Abstraction

The platform's own databases (not customer source/target databases) use portable technologies:

- **Neo4j:** Runs identically on any infrastructure
- **MongoDB:** Available as managed service or self-hosted
- **Redis:** Available everywhere, including managed options (ElastiCache, Memorystore, Azure Cache)
- **Vector DB:** Pinecone (managed, cloud-neutral) or Weaviate (self-hosted)

---

### Cloud-Specific Optimizations

While the platform is cloud-agnostic at the architecture level, it includes optional optimizations for specific environments:

| Cloud          | Optimization                                                     | Benefit                  |
| -------------- | ---------------------------------------------------------------- | ------------------------ |
| **AWS**        | EBS io2 for Neo4j, S3 Transfer Acceleration for large migrations | Higher I/O throughput    |
| **GCP**        | TPU support for local model inference, BigQuery connector        | Cost-effective inference |
| **Azure**      | Azure OpenAI for reduced latency, Azure AD integration           | Enterprise SSO           |
| **On-premise** | NVMe direct-attach for databases, InfiniBand for Ray cluster     | Maximum performance      |

These optimizations are additive — the platform works without them but performs better with them.

---

### Multi-Cloud and Hybrid Scenarios

Common deployment patterns that span infrastructure boundaries:

**Hybrid SaaS + Customer VPC**
Cognitive and Meta tiers in Sensei's cloud. Execution tier in customer's VPC. Source data never leaves the customer's network. Abstract patterns (not raw data) transit to the Meta tier.

**Multi-Cloud Migration**
Source system on AWS. Target system on GCP. Sensei running on Azure. The platform connects to source and target through encrypted tunnels regardless of where each system resides.

**On-Premise + Cloud Burst**
Base deployment on-premise for data sovereignty. Burst capacity on cloud for large migrations. Ray's auto-scaling seamlessly extends to cloud nodes when on-premise capacity is insufficient.

---

### Portability Testing

The platform's CI/CD pipeline includes portability tests:

- Every release is tested on EKS, GKE, and AKS
- Helm charts are provider-agnostic with cloud-specific value overrides
- Integration tests run against both managed and self-hosted database options
- LLM abstraction layer is tested against all supported providers

→ [Edge & Offline Deployment](edge-deployment.md)
→ [Kubernetes Deployment](kubernetes.md)
→ [Deployment Architecture Overview](deployment.md)
