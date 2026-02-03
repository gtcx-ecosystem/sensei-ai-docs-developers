---
description: Build with and extend the Sensei platform
---

# Developers

Sensei is a polyglot platform — TypeScript for the API and dashboard, Python for AI agents and transformation, Rust for cryptographic verification. Everything coordinates through gRPC and a shared event bus. This section covers architecture internals, component deep dives, API integration, and configuration reference.

```bash
make setup    # Install all dependencies
make dev      # Start the full stack
make test     # Run all tests
```

---

## Architecture

Sensei's agent system is organized into three layers: cognitive agents that plan migrations, execution agents that transform data, and meta agents that learn across migrations. Understanding this architecture is the foundation for extending the platform.

| Topic | What You'll Learn |
| --- | --- |
| [**Three-Layer Architecture**](architecture/three-layer-architecture/README.md) | Cognitive, execution, and meta agent layers |
| [**Agent Reference**](architecture/scout-agents.md) | Scout, Architect, Worker, Validator, and Learning agents |
| [**Learning Subsystems**](architecture/swarm-learning.md) | Swarm, vector, and evolutionary learning loops |
| [**Infrastructure**](architecture/technology-stack/README.md) | Data flow, security, scalability, and deployment |

---

## Components

Three engines power every migration. MABA handles data transformation, KORA proves correctness, and AMANI coordinates human decisions — each designed to operate independently or as a pipeline.

| Engine | Role | Deep Dive |
| --- | --- | --- |
| **MABA** | Schema analysis, semantic mapping, transformation code generation | [How It Works](components/maba/how-it-works.md) |
| **KORA** | Multi-dimensional validation, temporal state capture, behavioral equivalence | [How It Works](components/kora/how-it-works.md) |
| **AMANI** | CRDT synchronization, differential privacy, offline-first coordination | [How It Works](components/amani/how-it-works.md) |

---

## Integration

Connect your systems to Sensei through REST APIs, client SDKs, CLI, or event-driven webhooks. Database connectors handle the source and target adapters.

| Resource | Description |
| --- | --- |
| [**API Reference**](integration/api-reference/README.md) | REST endpoints for migrations, connections, and validation |
| [**SDKs**](integration/sdks.md) | Python and TypeScript client libraries |
| [**CLI**](integration/cli.md) | Command-line interface for scripting and automation |
| [**Webhooks**](integration/webhooks.md) | Event notifications for migration state changes |
| [**Connectors**](integration/connectors.md) | Source and target database adapters |

---

## Reference

Configuration options, error codes, cross-database type mappings, and architectural decision records.

| Resource | Description |
| --- | --- |
| [**Configuration**](reference/configuration-reference.md) | All config options and environment variables |
| [**Error Codes**](reference/error-codes.md) | Error codes with resolution steps |
| [**Data Type Mappings**](reference/data-types.md) | Cross-database type and SQL translation tables |
| [**Design Records**](adrs/0001-monorepo.md) | ADRs and RFCs documenting key technical decisions |
| [**Diagrams**](diagrams/architecture.md) | Architecture, data flow, and deployment visuals |
