# SDKs

## Official Client Libraries

Sensei provides official SDKs for popular programming languages. All SDKs are open-source, type-safe, and designed for both synchronous and asynchronous usage.

---

## Python SDK

The most feature-complete SDK, ideal for data engineering workflows.

### Installation

```bash
pip install sensei-sdk
```

### Quick Start

```python
from sensei import Sensei

# Initialize client
client = Sensei(api_key="sk_live_abc123")

# Create a migration
migration = client.migrations.create(
    name="Oracle to PostgreSQL",
    source={"connection_id": "conn_oracle"},
    target={"connection_id": "conn_postgres"},
    options={"parallelism": 8}
)

# Wait for planning to complete
migration.wait_for_plan()

# Review and approve low-confidence mappings
for mapping in migration.plan.mappings:
    if mapping.confidence < 0.9:
        print(f"Review: {mapping.source} -> {mapping.target}")
        print(f"  Confidence: {mapping.confidence}")
        print(f"  Reasoning: {mapping.reasoning}")
        mapping.approve()

# Start migration
migration.start()

# Monitor progress
for event in migration.stream_events():
    print(f"[{event.phase}] {event.message}")
    if event.type == "progress":
        print(f"  Progress: {event.percent}%")
```

### Async Support

```python
import asyncio
from sensei import AsyncSensei

async def run_migration():
    client = AsyncSensei(api_key="sk_live_abc123")

    migration = await client.migrations.create(...)
    await migration.wait_for_plan()
    await migration.start()

    async for event in migration.stream_events():
        print(event)

asyncio.run(run_migration())
```

### Features

- Full API coverage
- Type hints for IDE support
- Async/await support
- Streaming for logs and events
- Automatic retry with backoff
- Connection pooling

**Documentation:** [Python SDK Reference](https://docs.sensei.ai/sdks/python)

**Source:** [github.com/sensei-ai/sensei-sdk](https://github.com/sensei-ai/sensei-sdk)

---

## TypeScript/JavaScript SDK

For Node.js backends and full-stack applications.

### Installation

```bash
npm install @sensei-ai/sdk
# or
yarn add @sensei-ai/sdk
```

### Quick Start

```typescript
import { Sensei } from '@sensei-ai/sdk';

const client = new Sensei({ apiKey: 'sk_live_abc123' });

// Create migration
const migration = await client.migrations.create({
  name: 'Oracle to PostgreSQL',
  source: { connectionId: 'conn_oracle' },
  target: { connectionId: 'conn_postgres' },
});

// Start and monitor
await migration.start();

for await (const event of migration.streamEvents()) {
  console.log(`[${event.phase}] ${event.message}`);
}
```

### Features

- Full TypeScript types
- ESM and CommonJS support
- Node.js 18+ and Bun support
- Streaming with async iterators
- Automatic retry logic

**Documentation:** [TypeScript SDK Reference](https://docs.sensei.ai/sdks/typescript)

---

## Go SDK

For high-performance backend services.

### Installation

```bash
go get github.com/sensei-ai/sensei-go
```

### Quick Start

```go
package main

import (
    "context"
    "fmt"
    "github.com/sensei-ai/sensei-go"
)

func main() {
    client := sensei.NewClient("sk_live_abc123")
    ctx := context.Background()

    migration, err := client.Migrations.Create(ctx, &sensei.MigrationParams{
        Name:   "Oracle to PostgreSQL",
        Source: &sensei.ConnectionRef{ID: "conn_oracle"},
        Target: &sensei.ConnectionRef{ID: "conn_postgres"},
    })
    if err != nil {
        panic(err)
    }

    // Start migration
    err = migration.Start(ctx)
    if err != nil {
        panic(err)
    }

    // Stream events
    events := migration.StreamEvents(ctx)
    for event := range events {
        fmt.Printf("[%s] %s\n", event.Phase, event.Message)
    }
}
```

**Documentation:** [Go SDK Reference](https://docs.sensei.ai/sdks/go)

---

## Java SDK

For enterprise Java applications.

### Installation

**Maven:**

```xml
<dependency>
  <groupId>ai.sensei</groupId>
  <artifactId>sensei-sdk</artifactId>
  <version>1.0.0</version>
</dependency>
```

**Gradle:**

```groovy
implementation 'ai.sensei:sensei-sdk:1.0.0'
```

### Quick Start

```java
import ai.sensei.Sensei;
import ai.sensei.models.*;

public class Example {
    public static void main(String[] args) {
        Sensei client = Sensei.builder()
            .apiKey("sk_live_abc123")
            .build();

        Migration migration = client.migrations().create(
            MigrationCreateParams.builder()
                .name("Oracle to PostgreSQL")
                .source(ConnectionRef.builder().id("conn_oracle").build())
                .target(ConnectionRef.builder().id("conn_postgres").build())
                .build()
        );

        migration.start();

        migration.streamEvents().forEach(event -> {
            System.out.printf("[%s] %s%n", event.getPhase(), event.getMessage());
        });
    }
}
```

**Documentation:** [Java SDK Reference](https://docs.sensei.ai/sdks/java)

---

## SDK Comparison

| Feature            | Python | TypeScript | Go  | Java |
| ------------------ | ------ | ---------- | --- | ---- |
| Full API Coverage  | Yes    | Yes        | Yes | Yes  |
| Async Support      | Yes    | Yes        | Yes | Yes  |
| Streaming          | Yes    | Yes        | Yes | Yes  |
| Type Safety        | Yes    | Yes        | Yes | Yes  |
| Auto Retry         | Yes    | Yes        | Yes | Yes  |
| Connection Pooling | Yes    | Yes        | Yes | Yes  |

---

## Common Patterns

### Error Handling

```python
from sensei import Sensei, SenseiError, RateLimitError

client = Sensei(api_key="sk_live_abc123")

try:
    migration = client.migrations.get("mig_invalid")
except RateLimitError as e:
    print(f"Rate limited. Retry after {e.retry_after} seconds")
except SenseiError as e:
    print(f"API error: {e.code} - {e.message}")
```

### Pagination

```python
# Iterate through all migrations
for migration in client.migrations.list(status="completed"):
    print(migration.name)

# Or fetch pages manually
page = client.migrations.list(limit=20)
while page.has_more:
    for migration in page.data:
        print(migration.name)
    page = page.next_page()
```

### Webhook Verification

```python
from sensei.webhooks import verify_signature

@app.post("/webhooks/sensei")
def handle_webhook(request):
    payload = request.body
    signature = request.headers["X-Sensei-Signature"]

    if not verify_signature(payload, signature, webhook_secret):
        return Response(status=401)

    event = json.loads(payload)
    # Process event...
```

---

## REST API

Prefer direct HTTP? All SDK functionality is available via REST:

```bash
curl -X POST https://api.sensei.ai/v1/migrations \
  -H "Authorization: Bearer sk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{"name": "My Migration", "source": {...}, "target": {...}}'
```

→ [API Reference](api-reference/README.md)
→ [CLI](cli.md)
→ [Webhooks](webhooks.md)
