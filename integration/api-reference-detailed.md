# API Reference

## Sensei Platform API

The Sensei REST API provides complete programmatic access to platform capabilities. Use it to embed migrations in your CI/CD pipelines, build custom dashboards, or integrate with your existing tooling.

---

### Base URL

```text
Production:  https://api.sensei.ai/v1
Staging:     https://api.staging.sensei.ai/v1
```

All requests require authentication via API key or OAuth token.

---

### Authentication

Include your API key in the `Authorization` header:

```bash
curl -H "Authorization: Bearer sk_live_abc123..." \
     https://api.sensei.ai/v1/migrations
```

Or use OAuth 2.0 for user-context operations:

```bash
curl -H "Authorization: Bearer oauth_token_xyz..." \
     https://api.sensei.ai/v1/migrations
```

→ [Authentication Details](authentication.md)

---

### Core Resources

#### Migrations

| Endpoint                  | Method | Description                    |
| ------------------------- | ------ | ------------------------------ |
| `/migrations`             | GET    | List all migrations            |
| `/migrations`             | POST   | Create a new migration         |
| `/migrations/{id}`        | GET    | Get migration details          |
| `/migrations/{id}`        | PATCH  | Update migration configuration |
| `/migrations/{id}`        | DELETE | Cancel and delete migration    |
| `/migrations/{id}/start`  | POST   | Start migration execution      |
| `/migrations/{id}/pause`  | POST   | Pause migration                |
| `/migrations/{id}/resume` | POST   | Resume paused migration        |
| `/migrations/{id}/status` | GET    | Get current status             |
| `/migrations/{id}/logs`   | GET    | Get migration logs             |

**Create Migration:**

```bash
curl -X POST https://api.sensei.ai/v1/migrations \
  -H "Authorization: Bearer sk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Q1 Database Modernization",
    "source": {
      "type": "oracle",
      "connection_string": "oracle://user:pass@host:1521/ORCL",
      "schema": "HR_PROD"
    },
    "target": {
      "type": "postgresql",
      "connection_string": "postgresql://user:pass@host:5432/hr_prod"
    },
    "strategy": "balanced",
    "options": {
      "auto_approve_high_confidence": true,
      "notify_on_completion": ["team@example.com"]
    }
  }'
```

**Response:**

```json
{
  "id": "mig_abc123def456",
  "name": "Q1 Database Modernization",
  "status": "created",
  "created_at": "2026-02-02T14:30:00Z",
  "source": {
    "type": "oracle",
    "schema": "HR_PROD",
    "tables": null,
    "estimated_rows": null
  },
  "target": {
    "type": "postgresql",
    "schema": "hr_prod"
  },
  "strategy": "balanced",
  "links": {
    "self": "/v1/migrations/mig_abc123def456",
    "start": "/v1/migrations/mig_abc123def456/start",
    "status": "/v1/migrations/mig_abc123def456/status"
  }
}
```

---

#### Connections

| Endpoint                 | Method | Description            |
| ------------------------ | ------ | ---------------------- |
| `/connections`           | GET    | List saved connections |
| `/connections`           | POST   | Create a connection    |
| `/connections/{id}`      | GET    | Get connection details |
| `/connections/{id}`      | DELETE | Delete connection      |
| `/connections/{id}/test` | POST   | Test connection        |

---

#### Schemas

| Endpoint                                 | Method | Description                |
| ---------------------------------------- | ------ | -------------------------- |
| `/migrations/{id}/schema/source`         | GET    | Get analyzed source schema |
| `/migrations/{id}/schema/target`         | GET    | Get target schema          |
| `/migrations/{id}/mappings`              | GET    | Get all mappings           |
| `/migrations/{id}/mappings`              | PATCH  | Update mappings            |
| `/migrations/{id}/mappings/{mapping_id}` | GET    | Get specific mapping       |
| `/migrations/{id}/mappings/{mapping_id}` | PATCH  | Update specific mapping    |

---

#### Validation

| Endpoint                                  | Method | Description                                 |
| ----------------------------------------- | ------ | ------------------------------------------- |
| `/migrations/{id}/validation`             | GET    | Get validation status                       |
| `/migrations/{id}/validation/structural`  | GET    | Structural validation results               |
| `/migrations/{id}/validation/statistical` | GET    | Statistical validation results              |
| `/migrations/{id}/validation/referential` | GET    | Referential validation results              |
| `/migrations/{id}/validation/semantic`    | GET    | Semantic validation results                 |
| `/migrations/{id}/validation/behavioral`  | GET    | Time-Travel Testing results                 |
| `/migrations/{id}/certificate`            | GET    | Download Behavioral Equivalence Certificate |

---

#### Errors

| Endpoint                                     | Method | Description            |
| -------------------------------------------- | ------ | ---------------------- |
| `/migrations/{id}/errors`                    | GET    | List all errors        |
| `/migrations/{id}/errors/{error_id}`         | GET    | Get error details      |
| `/migrations/{id}/errors/{error_id}/resolve` | POST   | Mark error as resolved |
| `/migrations/{id}/errors/{error_id}/retry`   | POST   | Retry failed record    |

---

### Pagination

List endpoints support pagination:

```bash
curl "https://api.sensei.ai/v1/migrations?limit=20&offset=40"
```

Response includes pagination metadata:

```json
{
  "data": [...],
  "pagination": {
    "total": 156,
    "limit": 20,
    "offset": 40,
    "has_more": true
  }
}
```

---

### Filtering and Sorting

```bash
# Filter by status
curl "https://api.sensei.ai/v1/migrations?status=running"

# Filter by date range
curl "https://api.sensei.ai/v1/migrations?created_after=2026-01-01&created_before=2026-02-01"

# Sort by field
curl "https://api.sensei.ai/v1/migrations?sort=-created_at"  # descending
curl "https://api.sensei.ai/v1/migrations?sort=name"         # ascending
```

---

### Error Handling

Errors return appropriate HTTP status codes with structured error bodies:

```json
{
  "error": {
    "code": "validation_error",
    "message": "Source connection failed",
    "details": {
      "field": "source.connection_string",
      "reason": "Unable to connect: timeout after 30s"
    },
    "request_id": "req_xyz789"
  }
}
```

| Status | Meaning                                           |
| ------ | ------------------------------------------------- |
| 400    | Bad Request — Invalid parameters                  |
| 401    | Unauthorized — Invalid or missing API key         |
| 403    | Forbidden — Insufficient permissions              |
| 404    | Not Found — Resource doesn't exist                |
| 409    | Conflict — Operation conflicts with current state |
| 429    | Too Many Requests — Rate limit exceeded           |
| 500    | Internal Error — Something went wrong             |

---

### Rate Limits

| Plan         | Requests/minute | Requests/day |
| ------------ | --------------- | ------------ |
| Starter      | 60              | 10,000       |
| Professional | 300             | 100,000      |
| Enterprise   | 1,000           | Unlimited    |

→ [Rate Limits Details](rate-limits.md)

---

### SDKs

Official client libraries available:

- [Python SDK](sdks.md#python)
- [Node.js SDK](sdks.md#nodejs)
- [Go SDK](sdks.md#go)
- [Java SDK](sdks.md#java)

→ [SDK Documentation](sdks.md)
→ [Integration Examples](examples.md)
→ [Webhooks](webhooks.md)
