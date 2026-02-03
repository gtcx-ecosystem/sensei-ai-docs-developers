## Migration Data Flow

End-to-end sequence for a single migration request, from the initial API call through agent orchestration, schema analysis, data transformation, cryptographic verification, and final delivery of the migration certificate back to the client.

```mermaid
sequenceDiagram
    autonumber

    participant Client
    participant API as API Gateway
    participant Orch as Agent Orchestrator
    participant Scout as Scout Agent
    participant Arch as Architect Agent
    participant Worker as Worker Agent
    participant Val as Validator Agent
    participant MABA as MABA Service
    participant KORA as KORA Service
    participant SrcDB as Source DB
    participant Weav as Weaviate

    Client->>API: POST /migrations
    API->>Orch: CreateMigration (gRPC)

    Note over Orch: Phase 1 - Analysis
    Orch->>Scout: AnalyzeSource
    Scout->>MABA: AnalyzeSchema (gRPC)
    MABA->>SrcDB: Connect & introspect
    SrcDB-->>MABA: Schema metadata
    MABA->>Weav: Store embeddings
    MABA-->>Scout: Schema analysis result
    Scout-->>Orch: Analysis complete

    Note over Orch: Phase 2 - Planning
    Orch->>Arch: PlanMigration
    Arch->>MABA: GenerateMapping (gRPC)
    MABA-->>Arch: Column mappings & transforms
    Arch-->>Orch: Migration plan ready

    Note over Orch: Phase 3 - Execution
    Orch->>Worker: ExecuteMigration
    Worker->>MABA: TransformData (gRPC)
    MABA-->>Worker: Transformed rows
    Worker-->>Orch: Execution complete

    Note over Orch: Phase 4 - Verification
    Orch->>Val: VerifyMigration
    Val->>KORA: VerifyIntegrity (gRPC)
    KORA-->>Val: Verification scores
    Val-->>Orch: Verification passed

    KORA-->>API: Signed certificate
    API-->>Client: Migration complete + certificate
```
