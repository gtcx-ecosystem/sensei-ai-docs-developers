## Database Entity Relationship Diagram

The Sensei platform persists eight Prisma models in PostgreSQL. An Organization owns Users, Migrations, and AuditLogs. Each Migration tracks its Schemas, Verifications (with cryptographic certificates), and AgentExecutions. The Pattern model is standalone and stores learned migration patterns with Weaviate embedding references.

```mermaid
erDiagram

    Organization {
        string id PK
        string name
        string slug UK
        enum plan
        json settings
        datetime createdAt
        datetime updatedAt
    }

    User {
        string id PK
        string orgId FK
        string email UK
        string name
        enum role
        string avatarUrl
        datetime createdAt
        datetime updatedAt
    }

    Migration {
        string id PK
        string orgId FK
        string name
        string description
        enum status
        json sourceConfig
        json targetConfig
        float progress
        datetime startedAt
        datetime completedAt
    }

    Schema {
        string id PK
        string migrationId FK
        enum type
        json definition
        string fingerprint
        json metadata
    }

    Verification {
        string id PK
        string migrationId FK
        enum type
        enum status
        float score
        string certificateId UK
        json result
    }

    AgentExecution {
        string id PK
        string migrationId FK
        enum agentType
        enum status
        int tokensUsed
        int durationMs
        json input
        json output
    }

    AuditLog {
        string id PK
        string orgId FK
        string userId FK
        string action
        string resourceType
        string resourceId
        string ipAddress
        datetime createdAt
    }

    Pattern {
        string id PK
        string type
        string fingerprint UK
        string embeddingId
        int successCount
        int failureCount
        json metadata
    }

    Organization ||--o{ User : "has members"
    Organization ||--o{ Migration : "owns"
    Organization ||--o{ AuditLog : "tracks"
    Migration ||--o{ Schema : "defines"
    Migration ||--o{ Verification : "validates"
    Migration ||--o{ AgentExecution : "runs"
    User ||--o{ AuditLog : "generates"
```
