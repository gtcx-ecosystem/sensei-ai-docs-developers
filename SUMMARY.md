# Table of contents

## Overview

- [Developers Home](README.md)

## Architecture

- [Overview](architecture/README.md)
- [Design Philosophy](architecture/design-philosophy.md)
- [Three-Layer Architecture](architecture/three-layer-architecture/README.md)
  - [L1: Cognitive Agents](architecture/three-layer-architecture/cognitive-agents.md)
  - [L2: Execution Agents](architecture/three-layer-architecture/execution-agents.md)
  - [L3: Meta Agents](architecture/three-layer-architecture/meta-agents.md)
- [Technology Stack](architecture/technology-stack/README.md)
  - [Data Flow](architecture/technology-stack/data-flow.md)
  - [Security Architecture](architecture/technology-stack/security-architecture.md)
  - [Scalability](architecture/technology-stack/scalability.md)
  - [High Availability](architecture/technology-stack/high-availability.md)
- [Agent Protocol](architecture/agent-protocol.md)
- [Cloud-Agnostic Design](architecture/cloud-agnostic.md)
- [Deployment Architecture](architecture/deployment.md)
- [Edge and Offline Deployment](architecture/edge-deployment.md)
- [Kubernetes Architecture](architecture/kubernetes.md)
- [Agent Reference](architecture/scout-agents.md)
  - [Architect Agents](architecture/architect-agents.md)
  - [Worker Agents](architecture/worker-agents.md)
  - [Validator Agents](architecture/validator-agents.md)
  - [Learning Agents](architecture/learning-agents.md)
- [Learning Subsystems](architecture/swarm-learning.md)
  - [Vector Learning](architecture/vector-learning.md)
  - [Evolutionary Selection](architecture/evolutionary-selection.md)
  - [Tournament Selection](architecture/tournament-selection.md)

## Component Reference

- [MABA Internals](components/maba/how-it-works.md)
  - [Ingestion](components/maba/ingestion.md)
  - [Structural Analysis](components/maba/structural-analysis.md)
  - [Statistical Analysis](components/maba/statistical-analysis.md)
  - [Distributional Fingerprinting](components/maba/distributional-fingerprinting.md)
  - [Semantic Analysis](components/maba/semantic-analysis.md)
  - [LLM Inference](components/maba/llm-inference.md)
  - [Transformation Code Generation](components/maba/transformation-codegen.md)
- [KORA Internals](components/kora/how-it-works.md)
  - [Structural Validation](components/kora/structural-validation.md)
  - [Statistical Validation](components/kora/statistical-validation.md)
  - [Referential Validation](components/kora/referential-validation.md)
  - [Semantic Validation](components/kora/semantic-validation.md)
  - [Multi-Source Validation](components/kora/multi-source-validation.md)
  - [Temporal State Capture](components/kora/temporal-state-capture.md)
  - [Historical Replay](components/kora/historical-replay.md)
  - [Behavioral Equivalence Certification](components/kora/behavioral-equivalence-certification.md)
- [AMANI Internals](components/amani/how-it-works.md)
  - [CRDT Synchronization](components/amani/crdt-sync.md)
  - [Differential Privacy](components/amani/differential-privacy.md)
  - [Privacy Budget Accounting](components/amani/privacy-budget.md)
  - [Federated Learning](components/amani/federated-learning.md)
  - [Homomorphic Matching](components/amani/homomorphic-matching.md)
  - [Change Management](components/amani/change-management.md)
  - [Performance](components/amani/performance.md)
  - [Frontier Markets](components/amani/frontier-markets.md)
  - [Web Interface](components/amani/web-interface.md)

## Integration

- [Overview](integration/README.md)
- [API Reference](integration/api-reference/README.md)
  - [Authentication](integration/api-reference/authentication.md)
  - [Migrations](integration/api-reference/migrations.md)
  - [Connections](integration/api-reference/connections.md)
  - [Validation](integration/api-reference/validation.md)
- [API Reference (Detailed)](integration/api-reference-detailed.md)
- [Authentication Guide](integration/authentication.md)
- [SDKs](integration/sdks.md)
- [CLI](integration/cli.md)
- [Webhooks](integration/webhooks.md)
- [Connectors](integration/connectors.md)
  - [Source Connectors](integration/source-connectors.md)
  - [Target Connectors](integration/target-connectors.md)
- [Examples](integration/examples.md)
- [Rate Limits](integration/rate-limits.md)
- [Data Flow](integration/data-flow.md)
- [Agent Communication](integration/agent-communication.md)
- [Component Orchestration](integration/component-orchestration.md)

## Reference

- [Overview](reference/README.md)
- [Configuration Reference](reference/configuration-reference.md)
- [Environment Variables](reference/environment-variables.md)
- [API Status Codes](reference/api-status-codes.md)
- [Error Codes](reference/error-codes.md)
- [Data Type Mappings](reference/data-types.md)
- [SQL Translations](reference/sql-translations.md)
- [Supported Versions](reference/supported-versions.md)
- [Limits and Quotas](reference/limits-quotas.md)
- [Keyboard Shortcuts](reference/keyboard-shortcuts.md)
- [Regex Patterns](reference/regex-patterns.md)

## Architecture Decision Records

- [ADR 0001: Monorepo Structure](adrs/0001-monorepo.md)
- [ADR 0002: Language Choices](adrs/0002-language-choices.md)
- [ADR 0003: Database Selection](adrs/0003-database-selection.md)

## RFCs

- [RFC 0001: Agent Architecture](rfcs/0001-agent-architecture.md)
- [RFC 0002: Verification Protocol](rfcs/0002-verification-protocol.md)

## Diagrams

- [System Architecture](diagrams/architecture.md)
- [Data Flow](diagrams/data-flow.md)
- [Agent Lifecycle](diagrams/agent-lifecycle.md)
- [Deployment Pipeline](diagrams/deployment-pipeline.md)
- [Entity Relationship](diagrams/entity-relationship.md)
