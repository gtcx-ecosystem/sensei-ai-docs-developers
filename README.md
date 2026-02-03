---
description: Build with and extend the Sensei platform
---

# Developers

## Getting Started

Sensei is a polyglot system — TypeScript for the API and dashboard, Python for AI agents and transformation, Rust for cryptographic verification. Everything coordinates through gRPC and a shared event bus.

```bash
make setup    # Install all dependencies
make dev      # Start the full stack
make test     # Run all tests
```

---

## Explore the Platform

### Architecture

| Topic | What You'll Learn |
| --- | --- |
| [Three-Layer Architecture](architecture/three-layer-architecture/README.md) | Cognitive, execution, and meta agent layers |
| [Agent Reference](architecture/scout-agents.md) | Scout, Architect, Worker, Validator, and Learning agents |
| [Learning Subsystems](architecture/swarm-learning.md) | Swarm, vector, and evolutionary learning loops |
| [Infrastructure](architecture/technology-stack/README.md) | Data flow, security, scalability, and deployment |

### Components

| Engine | Role | Deep Dive |
| --- | --- | --- |
| **MABA** | Schema analysis and transformation | [How It Works](components/maba/how-it-works.md) |
| **KORA** | Behavioral equivalence verification | [How It Works](components/kora/how-it-works.md) |
| **AMANI** | Human guidance and offline coordination | [How It Works](components/amani/how-it-works.md) |

### Integration

| Resource | Description |
| --- | --- |
| [API Reference](integration/api-reference/README.md) | REST endpoints for migrations, connections, and validation |
| [SDKs](integration/sdks.md) | Python and TypeScript client libraries |
| [CLI](integration/cli.md) | Command-line interface |
| [Webhooks](integration/webhooks.md) | Event notifications |
| [Connectors](integration/connectors.md) | Source and target database adapters |

### Reference

| Resource | Description |
| --- | --- |
| [Configuration](reference/configuration-reference.md) | All config options and environment variables |
| [Error Codes](reference/error-codes.md) | Error codes with resolution steps |
| [Data Type Mappings](reference/data-types.md) | Cross-database type and SQL translation tables |
| [Design Records](adrs/0001-monorepo.md) | ADRs and RFCs for key technical decisions |
| [Diagrams](diagrams/architecture.md) | Architecture, data flow, and deployment visuals |
