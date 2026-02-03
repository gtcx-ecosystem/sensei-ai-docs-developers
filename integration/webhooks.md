---
description: Event notifications and callbacks
---

# Webhooks

## Real-Time Event Notifications

Webhooks push events to your systems as they happen — no polling required. Use them to trigger CI/CD pipelines, update dashboards, route alerts, or integrate with your existing notification infrastructure.

---

### Configuring Webhooks

Create a webhook endpoint in the dashboard or via API:

```bash
curl -X POST https://api.sensei.ai/v1/webhooks \
  -H "Authorization: Bearer sk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-server.com/webhooks/sensei",
    "events": ["migration.*", "error.escalated"],
    "secret": "whsec_your_signing_secret"
  }'
```

---

### Event Types

#### Migration Events

| Event                 | Description                     | Trigger                      |
| --------------------- | ------------------------------- | ---------------------------- |
| `migration.created`   | New migration created           | POST /migrations             |
| `migration.started`   | Migration execution began       | POST /migrations/{id}/start  |
| `migration.paused`    | Migration paused                | POST /migrations/{id}/pause  |
| `migration.resumed`   | Migration resumed               | POST /migrations/{id}/resume |
| `migration.completed` | Migration finished successfully | All phases complete          |
| `migration.failed`    | Migration failed                | Unrecoverable error          |
| `migration.cancelled` | Migration cancelled by user     | DELETE /migrations/{id}      |

#### Phase Events

| Event                       | Description                 |
| --------------------------- | --------------------------- |
| `migration.phase.started`   | A migration phase began     |
| `migration.phase.completed` | A migration phase completed |
| `migration.phase.failed`    | A migration phase failed    |

#### Approval Events

| Event               | Description                      |
| ------------------- | -------------------------------- |
| `approval.required` | Human approval needed to proceed |
| `approval.granted`  | Approval received                |
| `approval.denied`   | Approval denied                  |
| `approval.timeout`  | Approval request expired         |

#### Error Events

| Event             | Description                       |
| ----------------- | --------------------------------- |
| `error.occurred`  | An error was encountered          |
| `error.recovered` | An error was auto-recovered       |
| `error.escalated` | An error requires human attention |
| `error.resolved`  | An escalated error was resolved   |

#### Validation Events

| Event                   | Description                              |
| ----------------------- | ---------------------------------------- |
| `validation.started`    | Post-migration validation began          |
| `validation.completed`  | Validation completed                     |
| `validation.failed`     | Validation found issues                  |
| `certificate.generated` | Behavioral Equivalence Certificate ready |

---

### Payload Format

All webhooks share a common envelope:

```json
{
  "id": "evt_abc123def456",
  "type": "migration.phase.completed",
  "created_at": "2026-02-02T14:45:30.123Z",
  "data": {
    "migration_id": "mig_xyz789",
    "migration_name": "Q1 Database Modernization",
    "phase": "data_transfer",
    "phase_number": 3,
    "total_phases": 5,
    "duration_seconds": 3847,
    "records_processed": 5234891,
    "errors_recovered": 47,
    "errors_escalated": 0
  },
  "metadata": {
    "organization_id": "org_abc",
    "environment": "production"
  }
}
```

---

### Signature Verification

All webhooks are signed with your webhook secret. Verify signatures to ensure requests are genuine:

```python
import hmac
import hashlib

def verify_webhook(payload, signature, secret):
    expected = hmac.new(
        secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(f"sha256={expected}", signature)

# In your webhook handler
@app.post("/webhooks/sensei")
def handle_webhook(request):
    signature = request.headers.get("X-Sensei-Signature")
    payload = request.body

    if not verify_webhook(payload, signature, WEBHOOK_SECRET):
        return Response(status=401)

    event = json.loads(payload)
    process_event(event)
    return Response(status=200)
```

The signature is in the `X-Sensei-Signature` header as `sha256=<hex_digest>`.

---

### Retry Policy

Failed webhook deliveries are retried with exponential backoff:

| Attempt | Delay      |
| ------- | ---------- |
| 1       | Immediate  |
| 2       | 1 minute   |
| 3       | 5 minutes  |
| 4       | 30 minutes |
| 5       | 2 hours    |
| 6       | 8 hours    |
| 7       | 24 hours   |

After 7 failed attempts, the event is marked as failed and logged. You can view failed deliveries in the dashboard and manually retry.

**Success criteria:** HTTP 2xx response within 30 seconds.

---

### Event Filtering

Subscribe only to events you care about:

```json
{
  "url": "https://your-server.com/webhooks/sensei",
  "events": ["migration.completed", "migration.failed", "error.escalated"]
}
```

Use wildcards for broader subscriptions:

- `migration.*` — All migration events
- `error.*` — All error events
- `*` — All events (not recommended for production)

---

### Best Practices

1. **Verify signatures** — Always verify webhook signatures in production
2. **Respond quickly** — Return 200 within 5 seconds; do heavy processing asynchronously
3. **Handle duplicates** — Webhooks may be delivered more than once; use event IDs for deduplication
4. **Use HTTPS** — Webhook URLs must use HTTPS in production
5. **Monitor failures** — Set up alerts for webhook delivery failures

---

### Testing Webhooks

Use the webhook testing endpoint to send test events:

```bash
curl -X POST https://api.sensei.ai/v1/webhooks/{id}/test \
  -H "Authorization: Bearer sk_live_abc123" \
  -d '{"event_type": "migration.completed"}'
```

Or use the dashboard's webhook testing tool to send sample payloads to your endpoint.

---

### Webhook Management

| Endpoint                                        | Method | Description            |
| ----------------------------------------------- | ------ | ---------------------- |
| `/webhooks`                                     | GET    | List all webhooks      |
| `/webhooks`                                     | POST   | Create a webhook       |
| `/webhooks/{id}`                                | GET    | Get webhook details    |
| `/webhooks/{id}`                                | PATCH  | Update webhook         |
| `/webhooks/{id}`                                | DELETE | Delete webhook         |
| `/webhooks/{id}/test`                           | POST   | Send test event        |
| `/webhooks/{id}/deliveries`                     | GET    | List delivery attempts |
| `/webhooks/{id}/deliveries/{delivery_id}/retry` | POST   | Retry failed delivery  |

→ [API Reference](api-reference.md)
→ [Integration Examples](examples.md)
→ [Integration Overview](README.md)
