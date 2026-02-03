---
description: Request quotas and throttling
---

# Rate Limits

## API Quotas and Throttling

Rate limits protect the platform and ensure fair usage across all customers. Understanding limits helps you design integrations that work reliably.

---

### Rate Limit Overview

| Plan             | Requests/Minute | Requests/Day | Concurrent Migrations |
| ---------------- | --------------- | ------------ | --------------------- |
| **Starter**      | 60              | 10,000       | 2                     |
| **Professional** | 300             | 100,000      | 10                    |
| **Enterprise**   | 1,000           | Unlimited    | Unlimited             |

Limits apply per organization, not per API key.

---

### Rate Limit Headers

Every API response includes rate limit information:

```text
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 247
X-RateLimit-Reset: 1706889600
X-RateLimit-Retry-After: 0
```

| Header                    | Description                                    |
| ------------------------- | ---------------------------------------------- |
| `X-RateLimit-Limit`       | Maximum requests allowed per minute            |
| `X-RateLimit-Remaining`   | Requests remaining in current window           |
| `X-RateLimit-Reset`       | Unix timestamp when the window resets          |
| `X-RateLimit-Retry-After` | Seconds to wait before retrying (when limited) |

---

### Handling Rate Limits

When you exceed the rate limit, you'll receive a `429 Too Many Requests` response:

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Retry after 23 seconds.",
    "retry_after": 23
  }
}
```

**Best practices:**

```python
import time
import requests

def api_request_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)

        if response.status_code == 429:
            retry_after = int(response.headers.get('X-RateLimit-Retry-After', 60))
            print(f"Rate limited. Waiting {retry_after} seconds...")
            time.sleep(retry_after)
            continue

        return response

    raise Exception("Max retries exceeded")
```

---

### Endpoint-Specific Limits

Some endpoints have additional limits:

| Endpoint                    | Limit     | Reason                              |
| --------------------------- | --------- | ----------------------------------- |
| `POST /migrations`          | 10/minute | Prevents runaway migration creation |
| `POST /connections/test`    | 30/minute | Protects database connections       |
| `GET /migrations/{id}/logs` | 60/minute | Log fetching is resource-intensive  |
| `POST /webhooks/{id}/test`  | 10/minute | Prevents webhook spam               |

---

### Concurrent Migrations

The number of simultaneously running migrations is limited by plan:

| Plan         | Concurrent Migrations |
| ------------ | --------------------- |
| Starter      | 2                     |
| Professional | 10                    |
| Enterprise   | Unlimited             |

Attempting to start a migration when at the limit returns:

```json
{
  "error": {
    "code": "concurrent_limit_exceeded",
    "message": "Maximum concurrent migrations (2) reached. Wait for a migration to complete or upgrade your plan.",
    "current_running": 2,
    "limit": 2
  }
}
```

---

### Bulk Operations

For high-volume operations, use bulk endpoints instead of many individual requests:

**Instead of:**

```python
# Bad: 100 API calls
for mapping_id in mapping_ids:
    client.mappings.update(mapping_id, {"approved": True})
```

**Use:**

```python
# Good: 1 API call
client.mappings.bulk_update(
    migration_id="mig_abc123",
    updates=[{"id": mid, "approved": True} for mid in mapping_ids]
)
```

Bulk endpoints:

- `PATCH /migrations/{id}/mappings/bulk` — Update multiple mappings
- `POST /migrations/{id}/errors/bulk-resolve` — Resolve multiple errors
- `POST /webhooks/bulk` — Create multiple webhooks

---

### Webhook Limits

Webhook deliveries have separate limits:

| Metric                        | Limit      |
| ----------------------------- | ---------- |
| Webhooks per organization     | 100        |
| Events per webhook per minute | 1,000      |
| Payload size                  | 256 KB     |
| Delivery timeout              | 30 seconds |
| Retry attempts                | 7          |

---

### Requesting Limit Increases

Enterprise customers can request limit increases:

1. Contact your account manager
2. Provide justification and expected usage patterns
3. Limits are increased within 24 hours (if approved)

For temporary increases (e.g., one-time large migration):

```bash
curl -X POST https://api.sensei.ai/v1/rate-limits/temporary-increase \
  -H "Authorization: Bearer sk_live_admin_key" \
  -d '{
    "reason": "Q1 migration project - 500 tables",
    "requested_limit": 1000,
    "duration_hours": 48
  }'
```

---

### Monitoring Usage

View your current usage in the dashboard under **Settings → Usage** or via API:

```bash
curl https://api.sensei.ai/v1/usage/current \
  -H "Authorization: Bearer sk_live_abc123"
```

Response:

```json
{
  "period": "2026-02",
  "requests": {
    "total": 45231,
    "limit": 100000,
    "remaining": 54769
  },
  "migrations": {
    "active": 3,
    "limit": 10,
    "total_this_period": 12
  },
  "data_processed_gb": 127.4
}
```

Set up alerts for approaching limits:

```bash
curl -X POST https://api.sensei.ai/v1/alerts \
  -H "Authorization: Bearer sk_live_admin_key" \
  -d '{
    "type": "usage_threshold",
    "metric": "requests",
    "threshold_percent": 80,
    "notify": ["ops@example.com"]
  }'
```

→ [Authentication](authentication.md)
→ [API Reference](api-reference.md)
→ [Integration Overview](README.md)
