# Security Architecture

## Protection, Access, and Compliance

This document describes the security architecture of the Sensei platform. It covers encryption, credential management, access control, audit logging, network segmentation, and compliance posture. Security is not a feature of Sensei; it is an architectural constraint that governs every subsystem.

---

### Threat Model Summary

Sensei operates on customer data that is frequently sensitive: production database contents, schema definitions that encode business logic, and credentials for source and target systems. The threat model accounts for:

- **External adversaries** attempting unauthorized access via network-facing APIs.
- **Compromised credentials** through phishing, credential stuffing, or supply chain attacks.
- **Insider threats** from operators or engineers with legitimate but scoped access.
- **Data exfiltration** through compromised agents, logging pipelines, or third-party integrations.
- **Supply chain compromise** through tampered dependencies or container images.

Every control described in this document maps to one or more of these threat categories.

---

### Encryption at Rest

All persistent data is encrypted at rest using AES-256-GCM. This applies to:

- **PostgreSQL storage volumes.** Encrypted via the cloud provider's managed encryption service (AWS EBS encryption, GCP CMEK, or Azure Disk Encryption). The encryption key is a customer-managed key (CMK) stored in the provider's key management service.
- **Object storage (S3/GCS/R2).** Server-side encryption with customer-managed keys. Bucket policies enforce that objects cannot be written without encryption.
- **Redis persistence files.** When AOF or RDB persistence is enabled, the underlying volume is encrypted. Redis itself does not encrypt data in memory; this is mitigated by network segmentation (Redis is not exposed outside the cluster network).
- **Kafka log segments.** Encrypted at the filesystem level via encrypted volumes. In-flight encryption between brokers is covered under transport encryption.
- **Verification artifacts.** KORA's Behavioral Equivalence Certificates and evidence bundles are encrypted at rest in object storage and additionally carry their own Ed25519 digital signatures for integrity verification.

**Key management:**

- Encryption keys are managed through the configured secrets provider: AWS KMS, HashiCorp Vault, or GCP Cloud KMS.
- Key rotation is automated on a 90-day cycle. Re-encryption of existing data occurs during scheduled maintenance windows.
- Key access is restricted to the platform's service accounts. No human operator has direct access to encryption keys.
- Key deletion is a two-phase operation requiring confirmation from two authorized principals and a 7-day waiting period.

---

### Encryption in Transit

All network communication within the Sensei platform uses TLS 1.3 as the minimum protocol version. TLS 1.2 is accepted only for connections to legacy source or target systems that do not support TLS 1.3, and only when explicitly enabled in the migration configuration.

**Inter-service communication (mTLS):**

- All communication between microservices within the Kubernetes cluster uses mutual TLS.
- Certificates are provisioned and rotated automatically by a service mesh (Istio or Linkerd) or by cert-manager with a private CA.
- Certificate lifetime is 24 hours with automatic renewal at 50% of lifetime.
- Services that cannot present a valid client certificate are rejected at the transport layer.

**External API endpoints:**

- The public API terminates TLS at the load balancer with certificates issued by a public CA (Let's Encrypt or AWS Certificate Manager).
- Cipher suites are restricted to AEAD ciphers: `TLS_AES_256_GCM_SHA384`, `TLS_AES_128_GCM_SHA256`, `TLS_CHACHA20_POLY1305_SHA256`.
- HSTS headers are set with `max-age=31536000; includeSubDomains`.
- OCSP stapling is enabled to reduce certificate validation latency.

**Database connections:**

- PostgreSQL connections require SSL with certificate verification (`sslmode=verify-full`).
- Redis connections use TLS when operating in sentinel mode across availability zones.
- Kafka inter-broker and client-broker communication uses TLS with SASL/SCRAM authentication.

---

### Credential Vault Integration

Sensei does not store credentials in configuration files, environment variables, or container images. All credentials are retrieved at runtime from a credential vault.

**Supported vault providers:**

| Provider            | Use Case                           |
| ------------------- | ---------------------------------- |
| HashiCorp Vault     | Self-hosted and hybrid deployments |
| AWS Secrets Manager | AWS-native deployments             |
| GCP Secret Manager  | GCP-native deployments             |
| Azure Key Vault     | Azure-native deployments           |

**Credential lifecycle:**

1. **Provisioning.** Credentials are created in the vault by an authorized administrator or by an automated provisioning pipeline. The platform never generates credentials for external systems.
2. **Retrieval.** Services retrieve credentials at startup and cache them in memory with a TTL equal to the credential's remaining validity period. Cached credentials are never written to disk.
3. **Rotation.** The vault rotates credentials on a configurable schedule (default: 90 days for long-lived credentials, 24 hours for database passwords). The platform detects rotated credentials by monitoring the vault's version metadata and refreshes its in-memory cache.
4. **Revocation.** When a credential is revoked, the platform's cache TTL ensures that the revoked credential is evicted within one TTL cycle. For immediate revocation, the control plane broadcasts a cache-invalidation event via Redis pub/sub.

**Source and target system credentials:**

- Credentials for customer source and target databases are stored in the vault under a customer-scoped namespace.
- No Sensei engineer or support operator can retrieve customer credentials. Access is restricted to the platform's service accounts that execute migrations.
- Credential access is logged in the audit trail with the requesting service identity, timestamp, and operation context.

---

### Role-Based Access Control (RBAC)

Sensei implements RBAC at three levels: platform, organization, and migration.

**Platform roles:**

| Role             | Permissions                                                            |
| ---------------- | ---------------------------------------------------------------------- |
| Platform Admin   | Full access to all platform operations, user management, configuration |
| Support Engineer | Read access to migration metadata, logs; no access to customer data    |
| System Operator  | Infrastructure management, deployment operations, no data access       |

**Organization roles:**

| Role             | Permissions                                                        |
| ---------------- | ------------------------------------------------------------------ |
| Org Admin        | Manage users, configure organization settings, view all migrations |
| Migration Owner  | Create, configure, execute, and monitor migrations within the org  |
| Migration Viewer | Read-only access to migration status, reports, and certificates    |
| Auditor          | Read access to audit logs, certificates, and compliance reports    |

**Migration-level roles:**

| Role            | Permissions                                                          |
| --------------- | -------------------------------------------------------------------- |
| Migration Admin | Full control over a specific migration (configure, execute, approve) |
| Reviewer        | Approve or reject schema mappings, review KORA verification results  |
| Observer        | Read-only access to a specific migration's progress and artifacts    |

Roles are composable: a user may hold different roles on different migrations within the same organization. Role assignments are stored in PostgreSQL, cached in Redis, and evaluated on every API request by the authorization middleware.

**Policy enforcement:** Authorization decisions are evaluated at the API gateway for coarse-grained access control and at each service boundary for fine-grained resource-level permissions. Denied requests return HTTP 403 with a structured error body indicating the missing permission. Denied requests are logged to the audit trail.

---

### Audit Logging

Every state-changing operation in the Sensei platform is recorded in a tamper-evident audit log.

**Audit log schema:**

| Field       | Description                                                                |
| ----------- | -------------------------------------------------------------------------- |
| `event_id`  | Unique identifier (UUIDv7, time-ordered)                                   |
| `timestamp` | Event time in UTC (nanosecond precision)                                   |
| `actor`     | Authenticated principal (user ID or service account)                       |
| `action`    | Operation performed (e.g., `migration.create`, `mapping.approve`)          |
| `resource`  | Target resource identifier                                                 |
| `outcome`   | `success`, `denied`, or `error`                                            |
| `context`   | Request metadata (IP address, user agent, trace ID)                        |
| `diff`      | Before/after state for mutation operations (redacted for sensitive fields) |

**Storage and retention:**

- Audit logs are written to a dedicated PostgreSQL table with append-only permissions (no UPDATE or DELETE grants on the table).
- Logs are replicated to immutable object storage (S3 with Object Lock in compliance mode) on a 15-minute cycle.
- Retention period is 7 years for compliance-grade deployments, 1 year for standard deployments.
- Audit logs are indexed by actor, action, resource, and timestamp for efficient querying.

**Tamper detection:**

- Each audit log entry includes a SHA-256 hash of the previous entry, forming a hash chain.
- An independent verification process periodically validates the hash chain integrity and alerts on any discontinuity.

---

### Network Segmentation

The Sensei Kubernetes cluster is segmented into network zones using Kubernetes NetworkPolicy resources and, where available, cloud-provider VPC security groups.

**Network zones:**

| Zone       | Components                               | Ingress                       | Egress                                      |
| ---------- | ---------------------------------------- | ----------------------------- | ------------------------------------------- |
| Public     | Load balancer, CDN                       | Internet                      | API zone                                    |
| API        | API servers, AMANI web                   | Load balancer only            | Internal zone, PostgreSQL zone              |
| Internal   | MABA workers, KORA workers, Orchestrator | API zone, Kafka zone          | Kafka zone, PostgreSQL zone, Object storage |
| Kafka      | Kafka brokers                            | Internal zone, API zone       | Internal zone (inter-broker)                |
| Data       | PostgreSQL, Redis                        | API zone, Internal zone       | None (no outbound)                          |
| Management | Monitoring, logging, vault               | All zones (read-only metrics) | External alerting endpoints                 |

**Enforcement:**

- Default deny-all NetworkPolicy is applied to every namespace. Services must explicitly declare their ingress and egress requirements.
- Inter-zone traffic is encrypted via mTLS regardless of whether the underlying network is trusted.
- The Data zone has no egress. PostgreSQL and Redis cannot initiate outbound connections, eliminating a class of data exfiltration vectors.

---

### Secrets Management

Beyond the credential vault, Sensei manages several categories of secrets:

- **API keys** for platform authentication are generated as 256-bit random tokens, stored hashed (bcrypt) in PostgreSQL, and transmitted only over TLS.
- **JWT signing keys** for session tokens are RSA-4096 keys stored in the vault and rotated every 30 days. Previous keys are retained for the duration of the longest valid token lifetime to support graceful rotation.
- **KORA signing keys** for Behavioral Equivalence Certificates are Ed25519 keys stored in the vault with access restricted to the KORA service account exclusively.
- **Kubernetes secrets** are encrypted at rest in etcd using the Kubernetes encryption provider (AWS KMS, GCP KMS, or the built-in AES-CBC provider). Access to Kubernetes secrets is restricted by RBAC and audited.

No secret appears in application logs, error messages, or API responses. A log-scrubbing filter runs in the logging pipeline to detect and redact patterns matching known secret formats.

---

### Vulnerability Management

**Container image scanning:**

- All container images are scanned for known vulnerabilities (CVEs) before deployment using Trivy or Snyk Container.
- Images with critical or high-severity vulnerabilities are blocked from deployment by the CI/CD pipeline's admission policy.
- Base images are rebuilt weekly to incorporate upstream security patches.

**Dependency scanning:**

- Application dependencies (npm, pip, Cargo) are scanned on every pull request using Dependabot and `cargo audit`.
- Dependencies with known vulnerabilities are flagged and must be resolved before merge.
- A Software Bill of Materials (SBOM) is generated for each release and archived for compliance.

**Runtime scanning:**

- Falco monitors runtime behavior in the Kubernetes cluster, alerting on unexpected system calls, file access patterns, and network connections.
- Container images run with read-only root filesystems and restricted seccomp profiles.

**Penetration testing:** External penetration testing is conducted annually by an independent security firm. Findings are triaged within 48 hours, with critical findings remediated within 7 days.

---

### SOC 2 Type II Controls

Sensei's security architecture is designed to satisfy the Trust Service Criteria defined by SOC 2 Type II:

| Criteria                      | Control                                 | Implementation                                                              |
| ----------------------------- | --------------------------------------- | --------------------------------------------------------------------------- |
| CC6.1 - Logical access        | RBAC, MFA, least privilege              | Role-based authorization at three levels; MFA enforced for all human access |
| CC6.2 - Credential management | Vault integration, rotation             | Automated credential rotation, no static credentials in code                |
| CC6.3 - Encryption            | AES-256-GCM at rest, TLS 1.3 in transit | All data encrypted; key management via KMS                                  |
| CC6.6 - Network controls      | Network segmentation, WAF               | Zone-based isolation; default-deny policies                                 |
| CC7.1 - Monitoring            | Audit logging, SIEM integration         | Tamper-evident logs; real-time anomaly detection                            |
| CC7.2 - Incident response     | Runbooks, escalation paths              | Documented procedures; annual tabletop exercises                            |
| CC8.1 - Change management     | CI/CD gates, code review                | All changes reviewed, tested, and audited before deployment                 |
| A1.2 - Availability           | Multi-AZ, failover, monitoring          | 99.9% SLA with automated failover                                           |

The SOC 2 audit cycle runs annually. The most recent audit report is available to customers under NDA upon request.

---

### Compliance and Regulatory Considerations

Sensei's security controls support compliance with:

- **GDPR.** Data residency enforcement, PII detection and handling, right-to-erasure support via data lifecycle policies.
- **HIPAA.** Encryption at rest and in transit, audit logging, access controls, BAA availability for healthcare customers.
- **PCI DSS.** Network segmentation, encryption, access logging, vulnerability management.
- **SOX.** Audit trail immutability, segregation of duties, change management controls.

Specific compliance configurations are documented in the deployment guides for each regulatory framework.

> [Technology Stack Overview](README.md) > [High Availability](high-availability.md) > [Design Philosophy](../design-philosophy.md)
