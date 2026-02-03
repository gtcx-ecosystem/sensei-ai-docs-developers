---
description: Agent state transitions
---

## Multi-Agent Cognitive Architecture

Sensei uses a three-layer agent architecture. The Cognitive layer reasons about schemas and plans migrations. The Execution layer carries out transformations and performance tuning. The Meta layer validates results and feeds learned patterns back into the Cognitive layer, creating a self-improving loop.

```mermaid
flowchart LR

    subgraph Cognitive["Cognitive Layer"]
        SCOUT[Scout<br/><i>Analyze schemas</i>]
        ARCHITECT[Architect<br/><i>Plan migrations</i>]
        LEARNER[Learner<br/><i>Extract patterns</i>]
    end

    subgraph Execution["Execution Layer"]
        WORKER[Worker<br/><i>Transform data</i>]
        OPTIMIZER[Optimizer<br/><i>Tune performance</i>]
    end

    subgraph Meta["Meta Layer"]
        VALIDATOR[Validator<br/><i>Verify integrity</i>]
    end

    PATTERNS[(Pattern Library)]

    SCOUT -- "Schema Analysis" --> ARCHITECT
    ARCHITECT -- "Migration Plan" --> WORKER
    WORKER -- "Transformed Data" --> OPTIMIZER
    OPTIMIZER -- "Optimized Output" --> VALIDATOR
    VALIDATOR -- "Verification Report" --> LEARNER
    LEARNER -- "New Patterns" --> PATTERNS
    PATTERNS -. "Historical Patterns" .-> SCOUT

    LEARNER -. "Feedback Loop" .-> SCOUT

    classDef cognitive fill:#dbeafe,stroke:#2563eb,color:#1e3a5f
    classDef execution fill:#d1fae5,stroke:#059669,color:#064e3b
    classDef meta fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef store fill:#fce7f3,stroke:#db2777,color:#831843

    class SCOUT,ARCHITECT,LEARNER cognitive
    class WORKER,OPTIMIZER execution
    class VALIDATOR meta
    class PATTERNS store
```
