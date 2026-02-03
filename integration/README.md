# Integration

## Connecting Sensei to Your World

This section covers how to integrate Sensei with your existing infrastructure — connecting to source and target databases, embedding the platform in your workflows, and building on top of the API.

---

### How the Components Work Together

Before integrating externally, understand how Sensei's internal components orchestrate:

```text
┌─────────────────────────────────────────────────────────────┐
│                         AMANI                                │
│              (Human Interface Layer)                         │
│   Web │ WhatsApp │ SMS │ USSD │ Voice                       │
└───────────────────────┬─────────────────────────────────────┘
                        │ User intent + Agent responses
                        ↓
┌─────────────────────────────────────────────────────────────┐
│                    Agent Layer                               │
│  ┌─────────┐ ┌───────────┐ ┌────────┐ ┌───────────┐        │
│  │ Scout   │ │ Architect │ │ Worker │ │ Validator │        │
│  │ Agents  │ │ Agents    │ │ Agents │ │ Agents    │        │
│  └────┬────┘ └─────┬─────┘ └───┬────┘ └─────┬─────┘        │
│       └────────────┼───────────┼────────────┘              │
│                    ↓           ↓                            │
│              ┌─────────┐ ┌──────────┐                       │
│              │  Meta   │ │ Learning │                       │
│              │ Agents  │ │ Agents   │                       │
│              └─────────┘ └──────────┘                       │
└───────────────────────┬─────────────────────────────────────┘
                        │ Orchestration
                        ↓
┌─────────────────────────────────────────────────────────────┐
│                  Processing Layer                            │
│                                                              │
│  ┌──────────────────────┐    ┌──────────────────────┐      │
│  │        MABA          │    │        KORA          │      │
│  │  Universal Transform │ ←→ │  Verification Oracle │      │
│  │       Engine         │    │                      │      │
│  └──────────┬───────────┘    └──────────┬───────────┘      │
└─────────────┼────────────────────────────┼──────────────────┘
              ↓                            ↓
┌─────────────────────────────────────────────────────────────┐
│                   Data Layer                                 │
│  ┌────────────┐  ┌────────────┐  ┌────────────────────┐    │
│  │   Source   │  │   Target   │  │  Knowledge Stores  │    │
│  │ Databases  │  │ Databases  │  │ Vector│Graph│Doc   │    │
│  └────────────┘  └────────────┘  └────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

→ [Component Orchestration](component-orchestration.md) — How MABA, KORA, and AMANI coordinate
→ [Data Flow](data-flow.md) — End-to-end journey of data through the platform
→ [Agent Communication](agent-communication.md) — How agents discover, share, and learn

---

### External Integration Points

| Integration Type                              | Purpose                                    | Documentation                                 |
| --------------------------------------------- | ------------------------------------------ | --------------------------------------------- |
| **[Source Connectors](source-connectors.md)** | Connect to databases you're migrating from | Supported databases, connection configuration |
| **[Target Connectors](target-connectors.md)** | Connect to databases you're migrating to   | Supported targets, write optimization         |
| **[REST API](api-reference.md)**              | Programmatic platform control              | Full API reference                            |
| **[Webhooks](webhooks.md)**                   | Event notifications to your systems        | Event types, payload formats                  |
| **[SDKs](sdks.md)**                           | Client libraries for common languages      | Python, Node.js, Go, Java                     |

---

### Common Integration Patterns

#### Pattern 1: Embedded Migration

Trigger migrations programmatically from your deployment pipeline:

```python
from sensei import Sensei

client = Sensei(api_key="sk_...")
migration = client.migrations.create(
    source={"type": "postgresql", "connection_string": "..."},
    target={"type": "snowflake", "connection_string": "..."},
    strategy="balanced"
)
migration.start()
```

#### Pattern 2: Event-Driven Workflow

Receive webhooks when migration events occur:

```json
{
  "event": "migration.phase.completed",
  "migration_id": "mig_abc123",
  "phase": "schema_analysis",
  "next_phase": "data_transfer",
  "requires_approval": true
}
```

#### Pattern 3: Custom Notification Routing

Route AMANI notifications to your existing channels:

```python
@webhook_handler("notification.stakeholder")
def route_notification(event):
    if event.stakeholder_role == "executive":
        slack.post(channel="#exec-updates", message=event.summary)
    elif event.stakeholder_role == "technical":
        pagerduty.create_incident(event.details)
```

→ [Integration Examples](examples.md) — Complete working examples

---

### Security

All integrations require authentication and respect your security configuration:

- **[Authentication](authentication.md)** — API keys, OAuth, service accounts
- **[Rate Limits](rate-limits.md)** — Request quotas and throttling
- Network security — VPC peering, private endpoints, IP allowlisting

---

### Getting Started

1. **Connect your source** → [Source Connectors](source-connectors.md)
2. **Connect your target** → [Target Connectors](target-connectors.md)
3. **Start your first migration** → [Quickstart Guide](../../product/quick-start.md)
4. **Automate with the API** → [API Reference](api-reference.md)
