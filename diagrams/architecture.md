---
description: System architecture visual overview
---

## System Architecture

High-level view of the Sensei platform, organized into five layers: client interfaces, the API gateway, core gRPC services, supporting microservices, and the data/external layer. Arrows indicate the primary communication paths between components.

```mermaid
flowchart TD

    subgraph Clients["Client Layer"]
        WEB[Web Dashboard<br/><i>Next.js</i>]
        CLI[CLI<br/><i>Rust</i>]
        SDK[SDK<br/><i>TypeScript / Python</i>]
    end

    subgraph Gateway["API Gateway"]
        API[Hono REST API<br/><i>TypeScript :4000</i>]
    end

    subgraph Core["Core Services"]
        MABA[MABA<br/><i>Python gRPC :50051</i><br/>Schema Analysis &<br/>Transformation]
        KORA[KORA<br/><i>Rust gRPC :50052</i><br/>Verification &<br/>Crypto Certificates]
        ORCH[Agent Orchestrator<br/><i>Python gRPC :50053</i><br/>Multi-Agent Runtime]
    end

    subgraph Support["Supporting Services"]
        WEBHOOK[Webhook Service<br/><i>TypeScript</i>]
        SCHEDULER[Scheduler Service<br/><i>Python</i>]
    end

    subgraph Data["Data Layer"]
        PG[(PostgreSQL)]
        REDIS[(Redis)]
        WEAV[(Weaviate<br/>Vector DB)]
    end

    subgraph External["External Providers"]
        LLM[LLM Providers<br/><i>Anthropic / OpenAI</i>]
    end

    WEB --> API
    CLI --> API
    SDK --> API

    API --> ORCH
    API --> MABA
    API --> KORA
    API --> WEBHOOK
    API --> SCHEDULER

    ORCH --> MABA
    ORCH --> KORA
    ORCH --> LLM

    MABA --> PG
    MABA --> WEAV
    KORA --> PG
    API --> PG
    API --> REDIS
    SCHEDULER --> REDIS

    classDef client fill:#dbeafe,stroke:#2563eb,color:#1e3a5f
    classDef gateway fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef core fill:#d1fae5,stroke:#059669,color:#064e3b
    classDef support fill:#ede9fe,stroke:#7c3aed,color:#3b0764
    classDef data fill:#fce7f3,stroke:#db2777,color:#831843
    classDef external fill:#f3f4f6,stroke:#6b7280,color:#1f2937

    class WEB,CLI,SDK client
    class API gateway
    class MABA,KORA,ORCH core
    class WEBHOOK,SCHEDULER support
    class PG,REDIS,WEAV data
    class LLM external
```
